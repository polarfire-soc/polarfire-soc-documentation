# MPFS Icicle Kit User Guide

## Table of Contents

- [MPFS Icicle Kit User Guide](#mpfs-icicle-kit-user-guide)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Connectors](#connectors)
  - [Jumpers](#jumpers)
    - [Default Jumper Settings](#default-jumper-settings)
    - [FlashPro Jumpers](#flashpro-jumpers)
  - [Coming out of the box](#coming-out-of-the-box)
  - [Updating](#updating)
  - [References](#references)

<a name="introduction"></a>

## Introduction

The PolarFire SoC Icicle Kit features an MPFS250T_ES PolarFire SoC device.
Microchip's PolarFire SoC devices combine a RISC-V 5x core Microprocessor Subsystem capable of running Linux and the PolarFire FPGA fabric in a single device.
This powerful combination enables the partitioning of user designs between the Microprocessor Subsystem (MSS) and the FPGA fabric.
Microchip's Libero SoC enables the rapid development of RTL based designs for PolarFire SoC and many other device families.
Libero SoC provides a wide range of IP for a variety of applications such as video and imaging, signal processing, wired and wireless communications, and networking.
Microchip's SoftConsole IDE enables the rapid development of C/C++ source code based applications targeted for all Microchip FPGA and SoC device families.

<a name="connectors"></a>

## Connectors

![](./images/icicle-kit-connectors.PNG)

<a name="jumpers"></a>

## Jumpers

![](./images/icicle-kit-jumper-settings.jpg)

<a name="default-jumper-settings"></a>

### Default Jumper Settings

The Icicle Kit jumper settings required to boot Linux are as follows:

| Jumper | Setting | Description                                                               |
|:-------|:--------|:--------------------------------------------------------------------------|
| J15    | Open    | USB device mode selection. Open: USB client. Closed: USB host             |
| J17    | Open    | USB device mode selection. Open: USB client. Closed: USB host             |
| J24    | Closed  | Required for Embedded FlashPro6 (eFP6)                                    |
| J28    | Closed  | Transceiver external reference clock. Closed: ground (no clock provided). |
| J31    | Open    | DDR controller reference voltage.                                         |
|        |         | Open: no external reference.                                              |
|        |         | Closed: External reference provided.                                      |
| J34    | 2 & 3   | Bank 4 reference voltage                                                  |
|        |         | 1&2 = 3v3                                                                 |
|        |         | 2&3 = 1v8                                                                 |
| J35    | 2 & 3   | Bank 4 I/O auxiliary supply voltage.                                      |
|        |         | 1&2 = 3v3                                                                 |
|        |         | 2&3 = 2v5                                                                 |
| J43    | 2 & 3   | U44 reference voltage                                                     |
|        |         | 1&2 = 3v3                                                                 |
|        |         | 2&3 = bank 4 reference voltage                                            |
| J45    | 2 & 3   | Core voltage (VDD) set to 1.05v                                           |
| J46    | Closed  | RPi 3v3 enable                                                            |

<a name="flashpro-jumpers"></a>

### FlashPro Jumpers

| Jumper | Description                                                             |
|:-------|:------------------------------------------------------------------------|
| J9     | Used to select between Embedded FlashPro6 and external FlashPro header. |
|        | Closed: eFP6 connected to J33 micro-USB port.                           |
|        | Open: External FlashPro connected to J23 header.                        |
| J21    | JTAG nTRST interface pull down enable. Leave open.                      |
| J24    | VBUS source. Leave closed.                                              |

<a name="coming-out-of-the-box"></a>

## Coming out of the box

The board is pre-programmed with a reference design booting Linux from eMMC.

Connect the Icicle Kit's *USB- UART Terminal* connector (J11) to your host computer. Use the terminal software of your choice (Putty ExtraPutty, minicom, screen) to open serial connections to the four UARTs available though the J11 USB connector. **Serial port settings: 115220 baud, no flow control, no parity**. Power cycle the board. This should result in boot messages appearing on two of the serials ports.

![](./images/icicle-kit-terminals.png)

MMUART0 displays the Hart Software Service (HSS) boot messages. MMUART1 displays U-Boot messages, Linux boot messages and provides a Linux prompt. The default user name is "root". No password is required. If a password is requested at any stage "microchip" is used as default password.

<a name="updating"></a>

## Updating

Consult the [Updating MPFS Kit](https://mi-v-ecosystem.github.io/redirects/boards-mpfs-generic-updating-mpfs-kit) document on the steps to update a PolarFire SoC development kit to the latest reference design and Linux images.

Pre-generated FPGA programming job files for the Icicle Kit can be found in the releases section of the [Icicle Kit Reference Design](https://mi-v-ecosystem.github.io/redirects/repo-icicle-kit-reference-design) repository.

Linux images for the Icicle Kit are available from the releases section of the [Meta PolarFire SoC Yocto BSP](https://mi-v-ecosystem.github.io/redirects/releases-meta-polarfire-soc-yocto-bsp).

<a name="references"></a>

## References

[PolarFire SoC Icicle Kit Quick Start Guide](https://www.microsemi.com/products/fpga-soc/polarfire-soc-icicle-quick-start-guide#overview)

[PolarFire SoC Icicle Kit Schematics](https://ww1.microchip.com/downloads/aemDocuments/documents/FPGA/ProductDocuments/ReferenceManuals/mpfs-icicle-kit-es-schematics.pdf)

[PolarFire SoC Icicle Kit Product Page](https://www.microchip.com/en-us/development-tool/MPFS-ICICLE-KIT-ES)
