# Updating PolarFire SoC  Icicle-Kit FPGA Design and Linux Image

- [Programming Files and Linux Images Links](#Links)
- [Jumper Settings](#Jumpers)
- [Serial Ports](#Serial-Ports)
- [Linux Credentials](#Credentials)
- [Programming The PolarFire SoC Design](#Programming-Design)
- [Programming the Linux Image](#Programming-Linux-Image)
    - [SD Card](#SD-Card) 
        - [Linux Host](#Linux-Host)
            - [Unmount any partitions from the SD card](#Unmount)
            - [Copy the wic image to the SD card](#Copy-wic)
        - [Windows Host](#Windows-Host)
- [Tools](#Tools)
- [Reference](#Reference)

This document provides links to files and instructions to provision an Icicle Kit to boot Linux.

Two items must be programmed to the board:
- PolarFire SoC Icicle Kit reference design - including first stage boot loader
- Linux image for the PolarFire SoC Icicle Kit reference design

<a name="Links"></a> 
##  Programming Files and Linux Images Links
Please note that the Linux images are provided as wic.gz files for Linux hosts and zip files for Windows. Please ensure you download the zip files if programming the board from a Windows host.

| Description                                                       | SD card |
| ----------------------------------------------- | --------- |
| SoC-FPGA + zero-stage bootloader FlashPro Express programming file | [FlashPro Express programming file](https://microchiptechnology-my.sharepoint.com/:u:/g/personal/cyril_jean_microchip_com/EfEZUeJqez5LjbgE7VLKQlIB7gD3O55D9AY2c-dymtsEmA?e=bTldiZ)  |
|  Minimal Linux image                        | [Minimal Linux Image (wc.gz)](https://microchiptechnology-my.sharepoint.com/:u:/g/personal/cyril_jean_microchip_com/EY7sfoD-Ud9Bqa0hTxnJCvABvZc4sZir87mYrWKxuN34vQ?e=lFas9R)  |
| Linux image with development tools | [Development Linux image (wic.gz)](https://microchiptechnology-my.sharepoint.com/:u:/g/personal/cyril_jean_microchip_com/EbPy2CSsm29ApA0y-wvGi2kBgLsGsP3g4HIdbVnh2RbkqQ?e=T4DtX3)  |
| Minimal Linux image zip (Windows host)| [Minimal Linux Image for Windows host (zip)](https://microchiptechnology-my.sharepoint.com/:u:/g/personal/cyril_jean_microchip_com/EbmNJM8GO-NMlBtx_azAqA4B6Ruw_kOQoS5owypxR20hBA?e=tYSAOY)  |
| Linux image with development tools zip (Windows host)| [Development Linux image for Windows host (zip)](https://microchiptechnology-my.sharepoint.com/:u:/g/personal/cyril_jean_microchip_com/EXZmE6wahFJFqNVNOT1Yi2MBSfD0pCtGRLgqsch-kgnH0w?e=ZMYqS5)  |


<a name="Jumpers"></a>
## Jumper Settings
For this design, the jumper settings must be set as follows:

### Design Specific Jumpers
|   Jumper  |  Setting |
| --------- | -------- |
|    J15    |   Open   |
|    J17    |   Open   |
|    J24    |   Open   |
|    J28    |  Closed  |
|    J31    |   Open   |
|    J34    |  1 & 2   |
|    J35    |  1 & 2   |
|    J43    |  1 & 2   |
|    J45    |  1 & 2   |
|    J46    |  Closed  |

### FlashPro Jumpers

|   Jumper  |  Description                                                         |
| --------- | -------------------------------------------------------------------- |
|    J9     |   Select between Embedded FlashPro and external FlashPro.            |
|           |     J9 closed: On-board Embedded FlashPro connected to J33 USB port. |
|           |     J9 open: External FlashPro connected to J23 header.              |
|    J21    |   JTAG nTRST interface pull down enable. Leave open.                 |

<a name="Serial-Ports"></a>
## Serial Ports
Please note that the first two serial ports available from the J11 USB-UART connector are used when booting Linux. The first UART displays first stage bootloader messages, the second UART displays U-Boot and Linux messages.

Serial ports settings: 115200 baud, 8-bit, no flow control.

<a name="Credentials"></a>
## Linux Credentials
You can login to the board using 'root' as user name and 'microchip' as password. Please note that the password is not always required.


<a name="Programming-Design"></a>
## Programming The PolarFire SoC Design
Please use FlashPro Express to program the PolarFire SoC design to the Icicle Kit.

<a name="Programming-Linux-Image"></a>
## Programming the Linux Image
<a name="SD-Card"></a>
### SD Card 
Put an SD card into the SD card reader of your host machine and use the instructions below depending on your host computer's operating system.

<a name="Linux-Host"></a>
#### Linux Host

<a name="Unmount"></a>
##### Unmount any partitions from the SD card
Find out which partitions were mounted, if any, for the SD card. This can be done by using the following command before and after plugging the SD card into your host computer:
```
ls /dev/sd*
```

Identify which new partitions appear after pluging in the SD card. For example:
```
/dev/sdc  /dev/sdc1  /dev/sdc2  /dev/sdc3
```
Use the following command to unmount the SD card replacing /dev/sdX with the drive letter found in the previous step. For example, "/dev/sd**c**?" from the example above.


```
sudo umount /dev/sdX?
```

<a name="Copy-wic"></a>
##### Copy the wic image to the SD card
Copy the Linux image of your choice to the SD card using the command below, replacing /dev/sdX with actual SD card drive name and &lt;linux-image&gt; with the name of the Linux image you downloaded from the table at the top of this document.

**!!! You must be extremely careful when changing sdX as selecting the wrong target may damage your host computer!!!**

```
zcat <linux-image>.wic.gz | sudo dd of=/dev/sdX bs=512 iflag=fullblock oflag=direct conv=fsync status=progress
```
Once writing has completed, eject the SD-card from the host PC. Insert it in the Icicle Kit's SD card slot and power cycle the board. You should see boot messages coming from the first two UARTs.

<a name="Windows-Host"></a>
#### Windows Host
Download and unzip the zip file for the Linux image you want to program to the Icicle Kit from the links provided in the table of the top of this document.

Download and install [Win32DiskImager](https://sourceforge.net/projects/win32diskimager/). 

Start Win32DiskImager

![](./images/start.png) 

Select the *Image File*.

![](./images/select-file.png) 

Please note you need to change the *Disk Image* type to \*.*. You must select a file of type .wic (not .wic.gz)

Select the *Device*.

![](./images/select-device.png) 

Click *Write*.

![](./images/progress.png) 

Once writing has completed, eject the SD-card from the host PC. Insert it in the Icicle Kit's SD card slot and power cycle the board. You should see boot messages coming from the first two UARTs.

<a name="Tools"></a>
## Tools
[FlashPro Express Linux Installer](https://www.microsemi.com/document-portal/doc_download/1244910-download-programming-and-debug-v12-4-for-linux) 

[FlashPro Express Windows installer](https://www.microsemi.com/document-portal/doc_download/1244911-download-programming-and-debug-v12-4-for-windows) 

[Win32DiskImager](https://sourceforge.net/projects/win32diskimager/) 

<a name="Reference"></a>
## Reference
[FlashPro Express User Guide for PolarFire](https://www.microsemi.com/document-portal/doc_download/137627-flashpro-express-user-guide-for-polarfire) 

[FlashPro Express Release Notes](https://www.microsemi.com/document-portal/doc_download/1244868-programming-and-debug-tools-v12-4-release-notes) 



