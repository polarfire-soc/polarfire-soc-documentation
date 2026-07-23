# PolarFire SoC sNVM Boot Guide

## Overview

This guide describes the HSS sNVM boot feature, which enables booting an HSS payload
from the PolarFire SoC Secure NVM (sNVM) into L2-LIM without requiring DDR memory or
an external flash device.

The PolarFire SoC contains 221 pages of Secure NVM (sNVM), each holding 252 bytes in
non-authenticated plaintext mode, for a total capacity of approximately **55 KB**
(55,692 bytes). The sNVM is accessed through the System Controller via the
`MSS_SYS_secure_nvm_read()` and `MSS_SYS_secure_nvm_write()` services.

The sNVM boot feature adds a storage backend to the Hart Software Services (HSS) that
reads an HSS-format payload from sNVM pages at boot time, assembles it into an L2-LIM
staging area, and passes it to the standard HSS boot service. This enables a boot chain
that uses only on-chip memory (eNVM + sNVM + L2-LIM) with no DDR or external flash
dependency.

This implementation uses non-authenticated plaintext sNVM pages. HSS still performs
its normal boot-image validation, including the HSS header CRC, but the sNVM storage
format does not authenticate the payload contents. Enable HSS payload signing when the
sNVM payload itself must be authenticated.

Prior to using sNVM, you must ensure that the pages are correctly allocated in Libero.
They must be readable from the MSS at least and writable if you intend on using YMODEM.
Ensure that enough pages are allocated and that the Kconfig matches exactly the range
of pages allocated in Libero to avoid access violations.

### Boot Chain

```text
eNVM (128 KB)          sNVM (55 KB)          L2-LIM (1.5 MB)
+-------------+        +------------+        +------------------+
| HSS  (E51)  |  --->  |  Payload   |  --->  | Payload runs on  |
| Boot Mode 1 |        |  (binary)  |        | U54 harts        |
+-------------+        +------------+        +------------------+
```

1. HSS boots from eNVM on the E51 monitor hart (Boot Mode 1)
2. HSS reads the payload from sNVM pages and assembles it in the L2-LIM staging area
3. HSS chunk-copies the payload to its target execution address in L2-LIM
4. Payload runs on U54 hart(s) in S-mode (with OpenSBI services provided by HSS)

### Memory Layout (No DDR)

Most PolarFire SoC reference designs configure L2-LIM as a 0.5 MB region and
place larger payloads in DDR. This no-DDR flow assumes a 1.5 MB L2-LIM
configuration, with the payload, sNVM staging buffer, and stack all located in
L2-LIM.

For a 0.5 MB L2-LIM design, update `CONFIG_SERVICE_BOOT_SNVM_STAGING_ADDR` and
the payload link address so that the staging buffer does not overlap HSS, the
payload, or the stack. The layout below follows the 1.5 MB L2-LIM map used by
the wolfBoot example:

```text
L2-LIM (1.5 MB: 0x08000000 - 0x0817FFFF):
  0x08000000 - 0x0803FFFF  HSS resident runtime (~256 KB)
  0x08040000 - 0x0805FFFF  Payload code + data (128 KB target area)
  0x08060000 - 0x080FFFFF  Application area (640 KB)
  0x08100000 - 0x0810DFFF  sNVM staging buffer (~56 KB, temporary during boot)
  0x08120000 - 0x0817FFFF  Stack (384 KB)

sNVM (55 KB, 221 pages x 252 bytes):
  Pages 0-220: HSS payload (hss-payload-generator output)

eNVM (128 KB):
  HSS firmware (Boot Mode 1)
```

## HSS Configuration

### Kconfig Options

The sNVM boot feature is controlled by the following Kconfig options under the
**Boot Service** menu:

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `CONFIG_SERVICE_BOOT_SNVM` | bool | n | Enable sNVM boot storage backend |
| `CONFIG_SERVICE_BOOT_SNVM_START_PAGE` | int | 0 | First sNVM page index (0-220) |
| `CONFIG_SERVICE_BOOT_SNVM_PAGE_COUNT` | int | 221 | Number of pages to read |
| `CONFIG_SERVICE_BOOT_SNVM_STAGING_ADDR` | hex | 0x08100000 | L2-LIM address for page assembly |
| `CONFIG_SERVICE_BOOT_SNVM_MAX_SIZE` | hex | 0xE000 | Maximum payload size (also used as the YMODEM receive buffer when DDR is disabled) |

Additional recommended settings for no-DDR operation:

| Option | Value | Description |
|--------|-------|-------------|
| `CONFIG_SKIP_DDR` | y | Skip DDR initialisation at boot |
| `CONFIG_SERVICE_MMC` | n | Disable eMMC/SD (not needed) |
| `CONFIG_SERVICE_YMODEM` | y | Enable YMODEM for sNVM programming |
| `CONFIG_SERVICE_TINYCLI` | y | Enable the TinyCLI console |

### Example def_config

A reference configuration for the MPFS Icicle Kit with sNVM boot is provided at:

```text
boards/mpfs-icicle-kit/def_config_examples/def_config_snvm
```

Key settings in this configuration:

- `CONFIG_SERVICE_BOOT_SNVM=y` with all sNVM sub-options
- `CONFIG_SKIP_DDR=y` — no DDR initialisation
- `CONFIG_SERVICE_YMODEM=y` — YMODEM console programming
- `CONFIG_SERVICE_TINYCLI_TIMEOUT=5` — longer CLI timeout for development
- `CONFIG_DEBUG_CHUNK_DOWNLOADS=y` — visible boot progress

## Building HSS with sNVM Support

```bash
cd hart-software-services
cp boards/mpfs-icicle-kit/def_config_examples/def_config_snvm .config
make clean && make BOARD=mpfs-icicle-kit -j$(nproc)
```

Program the HSS binary to eNVM using SoftConsole, FPExpress, or the boot mode
programmer:

```bash
make BOARD=mpfs-icicle-kit program
```

## Generating an HSS Payload

The payload must be in HSS payload format produced by `hss-payload-generator`.

### Payload YAML Template

```yaml
set-name: 'PolarFire-SoC-HSS::MyPayload-L2LIM'
hart-entry-points: {
  u54_1: '0x08040000',
  u54_2: '0x08040000',
  u54_3: '0x08040000',
  u54_4: '0x08040000'
}
payloads:
  my_payload.elf: {
    owner-hart: u54_1,
    priv-mode: prv_s
  }
```

- **hart-entry-points** must match the payload's link address in L2-LIM
- **owner-hart** specifies which U54 core executes the payload
- **priv-mode** is typically `prv_s` (S-mode) when running under HSS/OpenSBI

### Generate the Payload Binary

```bash
hss-payload-generator -vvv -c payload.yaml payload.bin
ls -la payload.bin   # Must be <= 55,692 bytes (221 pages x 252 bytes)
```

If the payload exceeds sNVM capacity, reduce ELF size by:

- Using `-Os` compiler optimisation
- Stripping debug symbols: `riscv64-unknown-elf-strip --strip-debug payload.elf`
- Disabling unused features
- Using link-time optimisation (LTO)

## Programming sNVM via YMODEM

HSS provides runtime sNVM programming through the YMODEM utility. No external
programmer is required.

### Step-by-Step Programming

1. Connect to the HSS UART0 console (115200 baud, 8N1).

2. At the HSS TinyCLI prompt, enter the YMODEM utility:

   ```text
   >> YMODEM
   ```

3. Select option **3** to start YMODEM receive:

   ```text
    3. YMODEM Receive -- receive application file from host
   ```

4. Send the payload binary from the host:

   **Using `sz` (lrzsz package):**

   ```bash
   sz --ymodem payload.bin < /dev/ttyUSB0 > /dev/ttyUSB0
   ```

   **Using minicom:** press `Ctrl+A` then `S`, select `ymodem`, navigate to `payload.bin`.

   **Using picocom:** launch with `--send-cmd "sz --ymodem -vv"` press `Ctrl+A` then `Ctrl+S`, type the path to `payload.bin`.

5. After the transfer completes, select option **7** to write to sNVM:

   ```text
    7. sNVM Write -- write received file to sNVM pages
   ```

   Progress is displayed as each page is written.

6. Optionally verify with option **8**:

   ```text
    8. sNVM Verify -- verify sNVM contents against received file
   ```

7. Select option **6** to exit, then issue the `SNVM` command to boot immediately
   without a power cycle:

   ```text
   >> SNVM
   ```

### Full Programming Session Example

With `def_config_snvm` (MMC and QSPI disabled), the YMODEM utility menu appears as:

```text
/sNVM Utility

 3. YMODEM Receive -- receive application file from host
 4. YMODEM Transmit -- send application file to host
 6. Quit -- quit Utility
 7. sNVM Write -- write received file to sNVM pages
 8. sNVM Verify -- verify sNVM contents against received file

 Select a number:
3
Waiting for YMODEM transfer...
[transfer payload.bin from host]
Received 48576 bytes

 Select a number:
7
Writing 48576 bytes to sNVM ...
Writing 193 bytes to sNVM pages 0-192 ...
  page 1/193
  page 11/193
  ...
  page 191/193
sNVM write complete: 193 pages written

 Select a number:
8
Verifying 193 sNVM pages ...
sNVM verify OK: 193 pages match

 Select a number:
6
>> SNVM
```

## Verification

After programming and rebooting, the HSS console should show:

1. HSS banner with DDR training skipped (when `CONFIG_SKIP_DDR=y`)
2. `sNVM: reading page X of Y...` — sNVM page assembly in progress
3. `Boot Image registered...` — payload header validated
4. Chunk download messages — payload copied to target L2-LIM address
5. Payload output on its configured UART

## Troubleshooting

| Symptom | Likely Cause | Resolution |
|---------|-------------|------------|
| `sNVM: bad magic` | sNVM contains invalid or uninitialised data | Re-program via YMODEM |
| Payload too large | ELF exceeds 55,692 bytes | Strip symbols, apply `-Os`, use LTO |
| No output after boot | Wrong entry point or link address | Verify YAML entry points match linker script |
| `sNVM write page N failed` | System Controller error | Check page range (0-220), retry |
| YMODEM timeout | Serial connection issue | Verify baud rate (115200), cable, and port |

## sNVM Technical Reference

| Property | Value |
|----------|-------|
| Page size (non-authenticated) | 252 bytes |
| Page count | 221 (module indices 0-220) |
| Total capacity | 55,692 bytes (~54.4 KB) |
| Access method | System Controller services |
| Read service | `MSS_SYS_secure_nvm_read()` |
| Write service | `MSS_SYS_secure_nvm_write()` |
| Write format | `MSS_SYS_SNVM_NON_AUTHEN_TEXT_REQUEST_CMD` |
| Payload authentication | HSS signing only, when `CONFIG_CRYPTO_SIGNING=y` |
| Persistence | Non-volatile; survives power cycles |

---

## wolfBoot Integration

[wolfBoot](https://github.com/wolfSSL/wolfBoot) is a secure bootloader that provides
firmware authentication and update. When combined with the HSS sNVM boot feature,
wolfBoot runs on a U54 hart in S-mode from L2-LIM and verifies/loads a signed
application from external QSPI flash.

In this flow, wolfBoot authenticates the application it loads after HSS starts it.
The HSS-to-wolfBoot handoff from sNVM is still non-authenticated plaintext unless HSS
payload signing is enabled.

### wolfBoot sNVM Boot Chain

```text
eNVM            sNVM              L2-LIM              QSPI Flash
+------+       +----------+      +-----------+       +-------------+
| HSS  | --->  | wolfBoot | ---> | wolfBoot  | --->  | Signed App  |
| (E51)|       | payload  |      | verifies  |       | Image       |
+------+       +----------+      | & loads   |       +-------------+
                                 | app       |
                                 +-----------+
```

1. HSS (E51, eNVM) reads the wolfBoot HSS payload from sNVM
2. wolfBoot (U54, L2-LIM) verifies the signed application in QSPI flash
3. Application runs from L2-LIM after signature verification

### wolfBoot Configuration

The wolfBoot L2-LIM configuration and HSS payload YAML are maintained in the wolfBoot
repository:

- **Config**: `config/examples/polarfire_mpfs250_hss_l2lim.config`
- **Payload YAML**: `hal/mpfs-l2lim.yaml`
- **Linker script**: `hal/mpfs250-hss.ld`

Key settings in `polarfire_mpfs250_hss_l2lim.config`:

| Setting | Value | Description |
|---------|-------|-------------|
| `WOLFBOOT_ORIGIN` | 0x08040000 | wolfBoot load address in L2-LIM |
| `WOLFBOOT_LOAD_ADDRESS` | 0x08060000 | Application load area in L2-LIM |
| `WOLFBOOT_STACK_TOP` | 0x08180000 | Top of L2-LIM stack |
| `WOLFBOOT_PARTITION_BOOT_ADDRESS` | 0x20020000 | QSPI flash boot partition |
| `WOLFBOOT_PARTITION_UPDATE_ADDRESS` | 0x22000000 | QSPI flash update partition |
| `WOLFBOOT_PARTITION_SWAP_ADDRESS` | 0x24000000 | QSPI flash swap partition |

### Building wolfBoot for sNVM

```bash
cd wolfBoot
cp config/examples/polarfire_mpfs250_hss_l2lim.config .config
make clean && make wolfboot.elf
```

### Generating the wolfBoot HSS Payload

```bash
hss-payload-generator -vvv -c hal/mpfs-l2lim.yaml wolfboot.bin
ls -la wolfboot.bin   # Verify <= 55,692 bytes
```

### Size Considerations

wolfBoot must fit within sNVM capacity (~55 KB) including the HSS payload header
overhead. Options to reduce binary size:

- Crypto algorithm selection (ECC256 vs ECC384)
- Math library selection (`SPMATHALL`, `NO_ASM`)
- Compiler flags (`-Os`, LTO)
- Disable unused features (delta updates, encrypted images)

### Programming wolfBoot to sNVM

Follow the YMODEM programming steps above, substituting `wolfboot.bin` as the payload.

### Signing and Loading the Application

After wolfBoot is running from sNVM/L2-LIM, sign and program the application to QSPI:

```bash
# Sign the application image
./tools/keytools/sign --ecc384 --sha384 test-app/image.elf \
    wolfboot_signing_private_key.der 1

# Program to QSPI flash via UART
python3 tools/scripts/mpfs_qspi_prog.py /dev/ttyUSB1 \
    test-app/image_v1_signed.bin 0x20000
```

### Expected Boot Output

On UART1 (U54):

```text
wolfBoot HAL Init
wolfBoot version: X.Y.Z
Verifying image at 0x20020000...
Signature OK (ECC384)
Booting application at 0x08060000
```

## References

- [PolarFire SoC documentation](https://github.com/polarfire-soc/polarfire-soc-documentation)
- [wolfBoot](https://github.com/wolfSSL/wolfBoot)
- PolarFire SoC MSS User's Guide — System Controller and sNVM services
