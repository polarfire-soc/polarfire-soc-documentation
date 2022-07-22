# Hart Software Services Payloads

- [HSS payloads](#payload-info)
- [Role of the HSS Payload Generator](#info)
- [Downloading the HSS Payload Generator](#download)
  - [Pre-built executables](#executables)
- [Generating payloads](#payloads)
  - [Generating a payload with a Linux image](#linux-payload)
  - [Configuring a bare metal application to be a payload](#config-bare-metal-payload)
  - [Generating a payload with a bare metal application](#bare-metal-payload)
    - [More detail on the use of OpenSBI](#opensbi)

This document describes:

- What a HSS payload is.
- The function of the HSS Payload Generator in the software flow for PolarFire SoC.
- How to build the HSS Payload Generator.
- How to generate a payload for PolarFire SoC using the HSS Payload Generator.

<a name="payload-info"></a>

## HSS payloads

A HSS payload is an image containing a header and one or more binary files that have been merged together to form the payload. An image contains:

- A header (containing a file identifier, data sizes and offsets, image information, hart specific configuration).
- A table of initialized boot chunks (code and data).
- A table of BSS and zero init chunks.
- The data itself broken into chunks.

Payloads are stored in non-volatile off chip memory (e.g., eMMC) and loaded on boot. This allows applications larger than on-chip storage (eNVM) to be executed by the PolarFire SoC MSS. Payloads are typically used to load a large executable from eMMC or SD card to DDR memory. When loading a payload, the HSS will copy the payload from non-volatile storage to DDR and then copy this data to the memory location(s) that were specified when the payload was generated.

<a name="info"></a>

## Role of the HSS Payload Generator

The HSS Payload Generator creates a payload that will be copied from non-volatile memory to volatile memory by the HSS on system boot. It merges and compresses binary files and headers describing where the payload should be copied into a single HSS payload binary file.

<a name="download"></a>

## Downloading the HSS Payload Generator

The HSS Payload Generator source code is bundled as a tool with the Hart Software Services, it's source code is available from the HSS repository on GitHub [here](https://mi-v-ecosystem.github.io/redirects/tool-hss-payload-generator).

<a name="executables"></a>

### Pre-built executables

Pre-compiled executables of the HSS Payload Generator for Windows and Linux are available as assets with the HSS repository releases [here](https://mi-v-ecosystem.github.io/redirects/releases-hart-software-services).

<a name="payloads"></a>

## Generating payloads

The HSS Payload Generator uses a configuration yaml file and command line arguments to configure the payload being built.

To generate a payload the following example command could be used:

```sh
./hss-payload-generator -c test/config.yaml output.bin
```

- `-c` specifies the configuration file being passed.
- `test/config.yaml` specifies that in this case the configuration file is in a sub-folder called `test` and called `config.yaml`.
- `output.bin` specifies the name of the output payload file.

<a name="linux-payload"></a>

### Generating a payload with a Linux image

The HSS Payload Generator is automatically cloned and run as part of the build process when Linux is built using the PolarFire SoC Buildroot SDK or the PolarFire SoC Yocto BSP

<a name="config-bare-metal-payload"></a>

### Configuring a bare metal application to be a payload

A YouTube video is available showing the configuration of a bare metal application as a payload:

[![Watch the video](https://img.youtube.com/vi/5Q3GZVD72GQ/0.jpg)](https://www.youtube.com/watch?v=5Q3GZVD72GQ&list=PL9B4edd-p2ahGOmnvJvFLvSID3N3-rJC6&index=2)

The main steps involved are, in the `mss_sw_config.h` file:

1. Ensure the `MPFS_HAL_FIRST_HART` define is set to a value greater than 0:
  - 0: Sets the E51 as the main hart of the system, as this is a payload the E51 will be running the HSS.
  - 1: Sets U54_1 as the main hart of the system.
  - 2: Sets U54_2 as the main hart of the system.
  - 3: Sets U54_3 as the main hart of the system.
  - 4: Sets U54_4 as the main hart of the system.
2. Disable the `MPFS_HAL_HW_CONFIG` define - this define enables the configuration of MSS I/O MUXs and DDR training in the HAL, as this is a payload this configuration will already have been completed by the HSS.
3. Ensure that `IMAGE_LOADED_BY_BOOTLOADER` define is set to 1.
4. If not using an SMP bare metal application. ensure that `MPFS_HAL_SHARED_MEM_ENABLED` is not defined.

Be especially careful if targetting the LIM to configure linker scripts appropriately: the default linker script should be adjusted if a bare metal application is going to target the LIM.
This is due to the fact that that the HSS uses a region of the LIM starting at 0x08000000 and the default bare metal linker scripts also target 0x08000000.
To resolve this, the map file for the HSS can be consulted to determine the end address of the region used by the HSS. Currently the HSS (plus OpenSBI) requires about 400KiB of LIM (0x60000), but this may change.

<a name="bare-metal-payload"></a>

### Generating a payload with a bare metal application

Once the application has been built, the resulting ELF file can be used to generate the HSS payload. An example configuration yaml file is shown below:

  set-name: 'PolarFire-SoC-HSS::bare-metal-image'
  hart-entry-points: {u54_1: '0x8030000', u54_2: '0x8030000', u54_3: '0x8030000', u54_4: '0x8030000'}
  payloads:
    bare-metal-app.elf: {exec-addr: '0x8030000', owner-hart: u54_1, secondary-hart: u54_2, secondary-hart: u54_3, secondary-hart: u54_4 priv-mode: prv_m, skip-opensbi: true}

- **set-name**: this command is used to define a name for the payload image.
- **hart-entry-points**: this is the address each hart will start executing from once the payload has been loaded into memory by the HSS.
- **payloads**: this command specifies the executables to be included in this image.
  - **bare-metal-app.elf**: this is the path and name of an executable (paths can be given relative to the hss-payload-generator executable).
  - **exec-addr**: this is the memory location this code should be loaded into.
  - **owner-hart**: this is the main hart that will be executing this payload.
  - **secondary-hart**: this can optionally be used to specify other harts associated with this payload.
  - **priv_mode**: this specifies the privilege mode this payload will execute in, available options are:
    - priv_u - user mode.
    - priv_s - supervisor mode.
    - priv_m - machine mode.
  - **skip_opensbi**: this specifies whether the bare metal application wants OpenSBI to start it, or whether it wants to be started directly by the HSS. See [More detail on the use of OpenSBI](#opensbi) for additional information.

This payload can then be generated by entering:

```sh
./hss-payload-generator -c config.yaml output.bin
```

The `bare-metal-app.elf` and `confg.yaml` files are in the same directory as the "hss-payload-generator" executable in this example. However, these files can be stored in any local directory and relative or full paths can be provided to the HSS Payload Generator.

The third argument in the command line above, `output.bin` is used as the output filename for the generated payload.This file can be generated in a separate folder by providing a full or relative path.

<a name="opensbi"></a>

#### More detail on the use of OpenSBI

For PolarFire SoC, the RISC-V Supervisor Binary Interface (SBI) is the recommended interface between platform-specific firmware running in M-mode and a bootloader or a general-purpose OS executing in S-mode or HS-mode.

The HSS provides a RISC-V SBI interface for applications through its incorporation of the OpenSBI library.  This ensures an initialised environment prior to starting the application. It does mean that the application will need to conform to the SBI interface, including the use of the Hart State Machine (HSM) when bringing up secondary harts in SMP.

For a bare metal application that isn't concerned with OpenSBI, the option `skip-opensbi`, if true, will cause the payload on that hart to be invoked using a simple `mret` rather than an OpenSBI `sbi_init()` call. This means the hart will start running the bare metal code irrespective of any OpenSBI HSM considerations. Note that this also means the hart cannot use `ECALL`s to invoke OpenSBI functionality.  The `skip-opensbi` option is optional, and defaults to false.
