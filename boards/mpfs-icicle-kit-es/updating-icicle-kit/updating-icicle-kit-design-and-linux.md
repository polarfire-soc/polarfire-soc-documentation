# Updating PolarFire SoC  Icicle-Kit FPGA Design and Linux Image

- [Programming Files and Linux Images Links](#Links)
- [Jumper Settings](#Jumpers)
- [Serial Ports](#Serial-Ports)
- [Linux Credentials](#Credentials)
- [Programming The PolarFire SoC Design](#Programming-Design)
- [Programming the Linux Image](#Programming-Linux-Image)
    - [eMMC](#eMMC)
    - [SD Card](#SD-Card)
- [Tools and References](#Tools-References)

This document provides links to files and instructions to provision an Icicle Kit to boot Linux. To achieve this:
- The PolarFire SoC Icicle Kit reference design, including the HSS boot loader must be programmed to the kit
- The on board eMMC or an SD card need to be programmed with a Linux image

<a name="Links"></a>
##  Programming Files and Linux Images Links
All programming files and Linux images are now being provided as assets with each release of their respective repositories.

Icicle Kit Reference Design programming files (including the HSS) are available as assets [here](https://github.com/polarfire-soc/icicle-kit-reference-design/releases).

Linux images are available as assets with each release of the Yocto build system [here](https://github.com/polarfire-soc/meta-polarfire-soc-yocto-bsp/releases).

**Note:**
From release 2020.11, the SoC-FPGA programming files are identical for SD card and eMMC. This means Linux can be booted from either an SD card or eMMC using the HSS and Icicle Kit reference design release 2020.11 or greater.

## Jumper Settings
The Icicle Kit jumper settings required to boot Linux are as follows:

![](./images/icicle_jumper_settings.jpg)

|   Jumper  |  Setting | Description            |
| --------- | -------- | -----------------------|
|    J15    |   Open   | USB device mode selection. Open: USB client. Closed: USB host              |
|    J17    |   Open   | USB device mode selection. Open: USB client. Closed: USB host              |
|    J24    |  Closed  | Required for Embedded FlashPro6 (eFP6)      |
|    J28    |  Closed  | Transceiver external reference clock. Closed: ground (no clock provided).  |
|    J31    |   Open   | DDR controller reference voltage.                 |
|           |          | Open: no external reference.                      |
|           |          | Closed: External reference provided.              |
|    J34    |  2 & 3   | Bank 4 reference voltage                          |
|           |          | 1&2 = 3v3                                         |
|           |          | 2&3 = 1v8                                         |
|    J35    |  2 & 3   | Bank 4 I/O auxiliary supply voltage.              |
|           |          | 1&2 = 3v3                                         |
|           |          | 2&3 = 2v5                                         |
|    J43    |  2 & 3   | U44 reference voltage                             |
|           |          | 1&2 = 3v3                                         |
|           |          | 2&3 = bank 4 reference voltage                    |
|    J45    |  1 & 2   | Core voltage (VDD) set to 1.0 |

### FlashPro Jumpers

|   Jumper  |  Description                                                         |
| --------- | -------------------------------------------------------------------- |
|    J9     |   Used to select between Embedded FlashPro6 and external FlashPro header.            |
|           |     Closed: eFP6 connected to J33 micro USB port. |
|           |     Open: External FlashPro connected to J23 header.              |
|    J21    |   JTAG nTRST interface pull down enable. Leave open.                 |

<a name="Serial-Ports"></a>
## Serial Ports
Please note that the first two serial ports available from the J11 USB-UART connector are used when booting Linux. The first UART displays zero stage bootloader and U-Boot messages, the second UART displays Linux messages and a console.

Serial ports settings: 115200 baud, 8-bit, no flow control.

<a name="Credentials"></a>
## Linux Credentials
You can login to the board using 'root' as user name, there is no password by default. If one is requested 'microchip' is used by default.

<a name="Programming-Design"></a>
## Programming The PolarFire SoC Design
Please use FlashPro Express to program the PolarFire SoC design to the Icicle Kit.

<a name="Programming-Linux-Image"></a>
## Programming the Linux Image
<a name="eMMC"></a>
### eMMC
The Icicle Kit's eMMC content is written by the Hart Software Services (HSS) using the `usbdmsc` command. The HSS `usbdmsc` command exposes the eMMC as a USB mass storage device through the Icicle Kit's USB-OTG J16 connector located beside the SD card slot.

#### eMMC content update procedure

1. Connect the J11 USB-UART connector to your host PC. This is the micro-USB connector on the same side as the Ethernet connectors. This connection will give you access to 4 of the PolarFire SoC UARTs
2. Open a terminal application to interact with the HSS through UART0. Settings are 115200 baud, 8 data bits, 1 stop bit, no parity, and no flow control.
3. Power cycle the board.
4. Type a key in the terminal application to stop the HSS from booting. This will give you access to the HSS command line interface.
5. Type `usbdmsc` in the HSS command line interface. This will expose the eMMC as a mass storage device through the USB-OTG connector.
6. Connect the J16 USB-OTG connector to your host PC. The eMMC content will be transfered to the Icicle Kit through this connection.
7. The Icicle Kit should now appear as mass storage device/drive on your host PC.
8. Download the zip file for the Linux image you want to program to the Icicle Kit from the links provided in the table of the top of this document.
9. Download and install [USBImager](https://bztsrc.gitlab.io/usbimager/).
10. Start USBImager 

    ![](./images/start.png)

11. Select *Image file*. 

    ![](./images/select-file.png)

12. Select the *Device*. 

    ![](./images/select-device.png)

13. Click *Write*. 

    ![](./images/write.png)

14. Once writing has completed, remove the J16 USB-OTG connector from the host PC or press `CTRL+C` in the HSS command line interface. 
15. Type `boot` to boot the newly copied Linux image.

<a name="SD-Card"></a>
### SD Card
1. Put an SD card into the SD card reader of your host PC and use the instructions below.

2. Download the zip file for the Linux image you want to program to the Icicle Kit from the links provided in the table of the top of this document.

3. Download and install [USBImager](https://bztsrc.gitlab.io/usbimager/).

4. Start USBImager 

    ![](./images/start.png)

5. Select *Image file*. 

    ![](./images/select-file.png)

6. Select the *Device*. 

    ![](./images/select-device.png)

7. Click *Write*. 

    ![](./images/write.png)

8. Once writing has completed, eject the SD-card from the host PC. 
9. Insert it in the Icicle Kit's SD card slot and power cycle the board. You should see boot messages coming from the first two UARTs.

<a name="Tools-References"></a>
## Tools and References
[FlashPro Express Installer and Release Notes Download](https://www.microsemi.com/product-directory/programming/4977-flashpro#software)

**Note:** FlashPro Express is installed with Libero and does not need to be installed separately. If Libero is not installed FlashPro Express is bundled with the Programming and Debug Tools. FlashPro and FlashPro Express are different tools.

[USBImager](https://bztsrc.gitlab.io/usbimager/)

[USBImager Manual](https://gitlab.com/bztsrc/usbimager/-/blob/master/usbimager-manual.pdf)

[FlashPro Express User Guides](https://www.microsemi.com/product-directory/programming/4977-flashpro#documents)
