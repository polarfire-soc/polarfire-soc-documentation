# Using U-Boot to TFTPboot a bare metal ELF on Microchip® PolarFire® SoC

- [Prerequisites](#prerequisites)
- [Configure and Build U-Boot](#configure-and-build-u-boot)
- [Payload Generator](#payload-generator)
- [Steps to TFTPboot a Bare Metal ELF File](#steps-to-tftpboot-a-bare-metal-elf-file)
  - [Connect to U-Boot](#connect-to-u-boot)
  - [Set Environment Variables](#set-environment-variables)
    - [Set the IP address of your TFTP server](#set-the-ip-address-of-your-tftp-server)
    - [Set the IP address of your target device](#set-the-ip-address-of-your-target-device)
  - [Load the ELF File](#load-the-elf-file)
  - [Execute the ELF File](#execute-the-elf-file)
  - [Verify Execution](#verify-execution)
- [Conclusion](#conclusion)

This guide will walk you through the process of using U-Boot to TFTPboot a bare metal ELF (Executable and
Linkable Format) file onto your Microchip PolarFire SoC device. TFTPboot-ing allows you to transfer and
execute programs over the network, making it a useful tool for embedded systems and development
environments.

It is particularly useful if you do not have (or want) a full Linux system on your board, but instead wish
to load and execute a bare metal ELF file via the network.

This guide assumes you will be building and installing on an Ubuntu host machine, and uses conventions for that Linux distribution.  If you are using a different Linux distribution, or on a different operating system, instructions may be slightly different.

<a name="prerequisites"></a>

## Prerequisites

- A TFTP server on your network, serving the ELF file you want to load.

- Basic knowledge of U-Boot commands.

### Install a TFTP server on Ubuntu (Linux)

The Ubuntu Linux distribution provides the `tftpd-hpa` server, an enhanced version of the BSD TFTP package with several bugfixes and enhancements.

Install it as follows:

```bash
$ sudo apt install tftpd-hpa
```

You can check that it is running using the following command:

```bash
$ sudo systemctl status tftpd-hpa
```

If you need to, you can find your IP address using the following command:

```bash
$ ip a
```

Now, copy your bare metal ELF file to `/var/lib/tftpboot` (or to `/srv/tftp` if `/src` exists and `/etc/default/tftpd-hpa` is configured to use it as its `TFTP_DIRECTORY`)  and ensure its file permissions make it readable.  For example, assuming it is called `my_bare_metal_app.elf` and is in your current directory, use the following command:

```bash
$ sudo cp my_bare_metal_app.elf /var/lib/tftpboot
$ sudo chmod 644 /var/lib/tftpboot/my_bare_metal_app.elf
```

<a name="configure-and-build-u-boot">

## Configure and Build U-Boot

Build U-Boot for the Icicle Kit by cloning from Git, configuring it and
compiling it.

For Linux distributions other than Ubuntu, ensure that the RISC-V cross compiler tools are installed and in your path - for example, asusming that `<RISCV_TOOLCHAIN_INSTALL_PATH>` specifies where the tools are installed:

```bash
$ export PATH=<RISCV_TOOLCHAIN_INSTALL_PATH>:$PATH
```

For Ubuntu, install package riscv64-linux-gnu-gcc and run the following commands.

This will create a `u-boot.bin` file, which you'll need for the next step to prepare the boot image.

```bash
$ git clone https://github.com/polarfire-soc/u-boot.git
$ make microchip_mpfs_icicle_defconfig
$ export CROSS_COMPILE=riscv64-linux-gnu-
$ make
```

<a name="payload-generator">

## Payload Generator

The Hart Software Services (HSS) Payload Generator is a tool that takes an application image and formats it
as a bootable image for PolarFire SoC.

Use the HSS Payload Generator binary assets from Github =>
[https://github.com/polarfire-soc/hart-software-services/releases/download/v2023.06/hss-payload-generator.zip](https://github.com/polarfire-soc/hart-software-services/releases/download/v2023.06/hss-payload-generator.zip)

Create a file named `config.yaml` with the following contents:

```yaml
##
## HSS Payload Generator
##

set-name: 'PolarFire-SoC-HSS::U-Boot'
hart-entry-points: {u54_1: '0x80200000', u54_2: '0x80200000', u54_3: '0x80200000', u54_4: '0x80200000'}
payloads:
  u-boot.bin: {exec-addr: '0x80200000', owner-hart: u54_1, secondary-hart: u54_2, secondary-hart: u54_3, secondary-hart: u54_4, priv-mode: prv_s}
```

Run the `hss-payload-generator` to create the `payload.bin` file:

```bash
$ ./hss-payload-generator -c config.yaml payload.bin
```

With this, you now have a payload.bin file that can be programmed into the eMMC (or SDCard), the SPI flash
connected to the MSS QSPI controller (for "boot from QSPI"), or in the SPI Flash connected to the system
controller (for "boot from SPI Flash"):

- to boot from QSPI, following the instructions at [booting-from-qspi.md](./booting-from-qspi.md)
- to boot U-Boot from eMMC (or SDCard), follow the instructions at [reference-designs-fpga-and-development-kits/updating-mpfs-kit.md\#emmc](../reference-designs-fpga-and-development-kits/updating-mpfs-kit.md\#emmc)
 to write the payload.bin file directly to the eMMC or SDCard - to program into the SPI Flash using Libero,
 follow the instructions at [programming-the-spi-flash-on-a-polarfire-soc-board-and-booting-via-hss.md](./programming-the-spi-flash-on-a-polarfire-soc-board-and-booting-via-hss.md)
- for more information on the HSS Payload Generator, see [hss-and-u-boot/hss-payloads.md\#info](../hss-and-u-boot/hss-payloads.md\#info)

<a name="steps-to-tftpboot-a-bare-metal-elf-file">

## Steps to TFTPboot a Bare Metal ELF File

<a name="connect-to-u-boot">

### Connect to U-Boot

Ensure your target device is connected to a terminal or serial console so you can interact with U-Boot.
Typically, the HSS uses MMUART0, and the U-Boot/application console is available on MMUART1.

Power-on your board, and press ENTER as soon as you see the U-Boot messages to interrupt it booting. You
should see the U-Boot command prompt, which will look something like the following:

```text
U-Boot 2022.01-linux4microchip+fpga-2023.06 (Jun 28 2023 - 07:35:32 +0000)

CPU:   rv64imafdc
Model: Microchip PolarFire-SoC Icicle Kit
DRAM:  1.8 GiB
MMC:   mmc@20008000: 0
Loading Environment from nowhere... OK
In:    serial@20100000
Out:   serial@20100000
Err:   serial@20100000
Net:   eth0: ethernet@20112000
Hit any key to stop autoboot:  0
RISC-V #
RISC-V #
```

<a name="set-environment-variables">

### Set Environment Variables

Before you can TFTPboot a file, you need to set the necessary environment variables in U-Boot.

<a name="set-the-ip-address-of-your-target-device">

#### Set the IP address of your target device

```text
RISC-V # setenv ipaddr <TARGET_DEVICE_IP>
```

Alternatively, you can run DHCP to automatically assign a target IP address.

Replace `<TARGET_DEVICE_IP>` with the IP address of your target device.

<a name="set-the-ip-address-of-your-tftp-server">

#### Set the IP address of your TFTP server

```text
RISC-V # setenv serverip <TFTP_SERVER_IP>
```

Replace `<TFTP_SERVER_IP>` with the IP address of your TFTP server.

<a name="load-the-elf-file">

### Load the ELF File

Use the tftp command to load the ELF file from the TFTP server into memory. You'll need to specify the
destination address in memory where the file should be loaded:

```text
RISC-V # tftpboot <LOAD_ADDRESS> <ELF_FILENAME>
```

where:

- `<LOAD_ADDRESS>` is the memory address where you want to load the ELF
  file.
- `<ELF_FILENAME>` is the name of the ELF file you want to load.

For example:

```text
RISC-V # tftpboot 0x80000000 my_bare_metal_app.elf
```

Note that U-Boot is typically of the other of 512KiB or greater in size, and it relocates itself to the top
of RAM upon start-up. Keep this in mind when linking your ELF file to make sure that its destination load
address will not clash with U-Boot.

<a name="execute-the-elf-file">

### Execute the ELF File

Once the ELF file is loaded into memory, you can execute it using the bootelf command:

```text
RISC-V # bootelf <LOAD_ADDRESS>
```

Replace `<LOAD_ADDRESS>` with the same memory address you used in the tftpboot command.

<a name="verify-execution">

### Verify Execution

After executing the ELF file, monitor the output on your terminal or serial console to verify that your bare
metal application is running correctly.

<a name="conclusion">

## Conclusion

You should now have successfully TFTPboot-ed a bare metal ELF file onto your target device using U-Boot.
This process allows you to load and run custom applications on your embedded system or development
environment.  You can now develop and test your bare metal software with ease!

For more advanced usage and customization, please refer to the U-Boot documentation at
[https://u-boot.readthedocs.io/en/latest/](https://u-boot.readthedocs.io/en/latest/).
