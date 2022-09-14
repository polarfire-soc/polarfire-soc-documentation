# Booting from an external QSPI flash memory device

## Table of Contents

- [Booting from an external QSPI flash memory device](#booting-from-an-external-qspi-flash-memory-device)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Requirements](#requirements)
    - [Tested on](#tested-on)
    - [Pre-Requisites](#pre-requisites)
  - [Programming the external QSPI flash memory using the HSS](#programming-the-external-qspi-flash-memory-using-the-hss)
    - [Using USBDMSC service](#using-usbdmsc-service)
  - [Boot Sequence](#boot-sequence)

<a name="introduction"></a>

## Introduction

This document provides a brief overview of programming an external QSPI Flash memory connected to the MSS QSPI peripheral.

<a name="requirements"></a>

## Requirements

The following flash memory devices have been tested with PolarFire SoC:

- Winbond W25N01GVZEIG/IT NAND Flash Memory
  - This part is available on a Mikroe [Flash 5 click board](https://www.mikroe.com/flash-5-click)
  - Depending on the PolarFire SoC board being used the QSPI I/O may be routed via the FPGA fabric to a Raspberry Pi interface, in this case a [Pi 3 Click Shield](https://www.mikroe.com/pi-3-click-shield) will also be required to mount the Flash 5 click board.

<a name="tested-on"></a>

### Tested on

The steps outlined below have been tested with the Winbond memory mentioned above on the PolarFire SoC Icicle Kit.

The Icicle Kit Reference Design routes QSPI I/O through the FPGA fabric which means a Pi 3 Click Shield is also required.

<a name="pre-requisites"></a>

### Pre-Requisites

Ensure you are using:

- Icicle Kit reference design v2022.03 or later

The Winbond NAND flash memory mentioned above can be connected to the Icicle Kit by using a Mikroe [Flash 5 click board](https://www.mikroe.com/flash-5-click) and a [Pi 3 Click shield](https://www.mikroe.com/pi-3-click-shield) as follows:

- Connect the Flash 5 Click board to slot 1 on the Pi 3 click shield

- Connect the Pi 3 click shield to the Icicle Kit using the Raspberry Pi 4 Interface (J26)

- Make sure J46 jumper is closed on the Icicle Kit

<a name="programming-the-external-QSPI-flash-memory-using-the-HSS"></a>

## Programming the external QSPI flash memory using the HSS

In regards to QSPI the Hart Software Services (HSS) can be used to:

- Write an HSS payload or Linux image to the QSPI flash memory
- Boot from QSPI by reading an HSS payload located in the QSPI flash memory

<a name="using-usbdmsc-service"></a>

### Using USBDMSC service

An external QSPI flash memory can be programmed using the HSS's USBDMSC service. For more information on how to program a Linux image to a QSPI flash memory please refer to the [Updating a PolarFire SoC FPGA Design and Linux Image](https://mi-v-ecosystem.github.io/redirects/updating-icicle-kit_updating-icicle-kit-design-and-linux) documentation.

<a name="boot-sequence"></a>

## Boot Sequence

The HSS will attempt to initialize the QSPI flash memory device. If no flash memory is connected, the HSS will attempt to initialize the SD card. If no card is inserted, the HSS will attempt to initialize the eMMC.

If a valid payload is found in any of the boot sources mentioned above, its contents will be unpacked into its destination memory (LIM, DDR, fabric memory etc) and owner U54 harts will be taken out of WFI to run the payload.
