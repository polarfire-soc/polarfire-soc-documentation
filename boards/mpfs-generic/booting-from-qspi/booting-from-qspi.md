# Booting from an external QSPI flash memory device

## Table of Contents

- [Booting from an external QSPI flash memory device](#booting-from-an-external-qspi-flash-memory-device)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Requirements](#requirements)
  - [Tested on](#tested-on)
  - [Pre-Requisites](#pre-requisites)
    - [Hart Software Services Configuration](#hart-software-services-configuration)
    - [Using a Winbond W25N01GV NAND flash memory](#using-a-winbond-w25n01gv-nand-flash-memory)
    - [Using a Micron MT25QL256 NOR flash memory](#using-a-micron-mt25ql256-nor-flash-memory)
  - [Building a Linux image for a NAND or NOR flash memory device](#building-a-linux-image-for-a-nand-or-nor-flash-memory-device)
  - [Programming the external QSPI flash memory using the HSS](#programming-the-external-qspi-flash-memory-using-the-hss)
    - [Using USBDMSC service](#using-usbdmsc-service)
  - [Boot Sequence](#boot-sequence)

<a name="introduction"></a>

## Introduction

This document provides a brief overview of programming an external QSPI Flash memory connected to the MSS QSPI peripheral.

<a name="requirements"></a>

## Requirements

The following flash memory devices are officially supported on PolarFire SoC:

| Manufacturer  | Part No. | Type | Density       |
|---------------|----------|------|---------------|
| Winbond       | W25N01GV | NAND | 1Gb (128MB)   |
| Micron*       | MT25QL256| NOR  | 256Mb (32MB)  |

*Micron QSPI is only supported by the Meta PolarFire SoC Yocto BSP.

Depending on the PolarFire SoC board being used the QSPI I/Os may be routed via the FPGA fabric to a Raspberry Pi interface, in this case an adapter daughter board may be required to connect a QSPI flash. See the individual sections below for more information.

<a name="tested-on"></a>

## Tested on

The steps outlined below have been tested with the supported memories mentioned above on the PolarFire SoC Icicle Kit.

The Icicle Kit Reference Design routes QSPI I/O through the FPGA fabric which means either a Pi 3 Click Shield or a PMOD HAT Adapter will also required depending on the interface of the memory.
See the individual sections below for more infomation.

<a name="pre-requisites"></a>

## Pre-Requisites

<a name="hart-software-services-configuration"></a>

### Hart Software Services Configuration

From release v2022.09 the HSS will no longer support booting from QSPI in the default build.
This means that the configuration option `SERVICE_QSPI` must be enabled in the HSS configuration to boot from QSPI flash.

- If the Winbond W25N01GV QSPI flash is being used `SERVICE_QSPI_WINBOND_W25N01GV` must also be enabled in the build.
  - The HSS includes an example configuration file in the boards/mpfs-icicle-kit-es/def_config_examples/ directory called `def_config_emmc_qspi-winbond` with the Winbond QSPI flash enabled.

- If the Micron MQ25T QSPI flash is being used `SERVICE_QSPI_MICRON_MQ25T` must also be enabled in the build.
  - The HSS includes an example configuration file in the boards/mpfs-icicle-kit-es/def_config_examples/ directory called `def_config_emmc_qspi-micron` with the Micron QSPI flash enabled.

Note: if you are using a reference design job file for a development kit, these job files bundle a HSS eNVM client.
This means you must first program the FPGA job to the target board and then update the HSS using the boot mode programmer in SoftConsole.
Alternatively the FPGA design can be generated in Libero SoC and a custom HSS build added as an eNVM client in the Libero design flow.
For more information refer to the [PolarFire SoC Software Tool Flow](https://mi-v-ecosystem.github.io/redirects/software-development_polarfire-soc-software-tool-flow) documentation.

<a name="using-a-winbond-w25n01gv-nand-flash-memory"></a>

### Using a Winbond W25N01GV NAND flash memory

Ensure you are using:

- Icicle Kit reference design v2022.09 or later with the standard configuration.
- A HSS built with `SERVICE_QSPI` and `SERVICE_QSPI_WINBOND_W25N01GV` enabled or built using the `def_config_emmc_qspi-winbond` configuration file.

The Winbond W25N01GV NAND flash memory can be connected to the Icicle Kit by using a Mikroe [Flash 5 click board](https://www.mikroe.com/flash-5-click) and a [Pi 3 Click shield](https://www.mikroe.com/pi-3-click-shield) as follows:

- Connect the Flash 5 Click board to slot 1 on the Pi 3 click shield

- Connect the Pi 3 click shield to the Icicle Kit using the Raspberry PI 4 Interface (J26)

- Make sure J46 jumper is closed on the Icicle Kit

<a name="using-a-micron-mt25ql256-nor-flash-memory"></a>

### Using a Micron MT25QL256 NOR flash memory

Ensure you are using:

- Icicle Kit reference design v2022.09 or later with the "MICRON_QSPI" configuration.
- A HSS built with `SERVICE_QSPI` and `SERVICE_QSPI_MICRON_MQ25T` enabled or built using the `def_config_emmc_qspi-micron` configuration file..

The Micron MT25QL256 NOR flash memory can be connected to the Icicle Kit by using a [Digilent Pmod SF3](https://digilent.com/reference/pmod/pmodsf3/start) and a [Pmod HAT Adapter](https://digilent.com/shop/pmod-hat-adapter-pmod-expansion-for-raspberry-pi) as follows:

- Connect the Pmod SF3 to slot JA on the Pmod HAT adapter

- Connect the Pmod HAT adapter to the Icicle Kit using the Raspberry PI 4 Interface (J26)

- Make sure J46 jumper is closed on the Icicle Kit

<a name="building-a-linux-image-for-a-nand-or-nor-flash-memory-device"></a>

## Building a Linux image for a NAND or NOR flash memory device

The [Microchip PolarFire SoC Yocto BSP](https://github.com/polarfire-soc/meta-polarfire-soc-yocto-bsp) layer allows building a minimal Linux image that can be programmed to the officially supported NAND and NOR flash memory devices.

Linux images that are suitable for programming to a NAND or NOR memory device have a `.nand.mtdimg` or `.nor.mtdimg` file extension respectively. These images can be programmed to the QSPI flash memory using the HSS.

<a name="programming-the-external-qspi-flash-memory-using-the-hss"></a>

## Programming the external QSPI flash memory using the HSS

The Hart Software Services (HSS) can be used to:

- Write a HSS payload or Linux image to the officially supported QSPI flash memory devices
- Boot from QSPI by reading a HSS payload located in the QSPI flash memory

<a name="using-usbdmsc-service"></a>

### Using USBDMSC service

An external QSPI flash memory can be programmed using the HSS's USBDMSC service.
For more information on how to program a Linux image to a QSPI flash memory please refer to the [Updating a PolarFire SoC FPGA Design and Linux Image](https://mi-v-ecosystem.github.io/redirects/updating-icicle-kit_updating-icicle-kit-design-and-linux) documentation.

<a name="boot-sequence"></a>

## Boot Sequence

The HSS will attempt to initialize the QSPI flash memory device. If no flash memory is connected, the HSS will attempt to initialize the SD card. If no card is inserted, the HSS will attempt to initialize the eMMC.

If a valid payload is found in any of the boot sources mentioned above, its contents will be unpacked into its destination memory (LIM, DDR, fabric memory etc) and owner U54 harts will be taken out of WFI to run the payload.
