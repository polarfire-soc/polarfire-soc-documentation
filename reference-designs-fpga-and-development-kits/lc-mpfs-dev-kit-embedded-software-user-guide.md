# LC-MPFS-DEV-KIT Embedded Software User Guide

## Overview

The LC-MPFS-DEV-KIT consists of SiFive's U540 processor and Microchip’s PolarFire FPGA on a single board. The LC-MPFS-DEV-KIT is a reduced version of the HiFive Unleashed platform. The LC-MPFS-DEV-KIT enables users to create a Linux system running on the RISC-V core complex, with a large FPGA fabric accessible through the memory map. The PolarFire FPGA is shipped with a pre-configured bitstream which enables peripherals such as GPIO, UART, SPI, and I2C on the PolarFire FPGA fabric.

## Hardware Features

This section describes the features of the LC-MPFS-DEV-KIT hardware with the block diagram.

The LC-MPFS-DEV-KIT consists of the following:

- SiFive Freedom U540 SoC
- 8 GB DDR4 with ECC
- Gigabit Ethernet port
- 32 MB Quad SPI flash connected to U540 SoC
- 1 GB SPI flash MT25QL01GBBB8ESF-0SIT connected to the PolarFire FPGA System controller
- MicroSD card for removable storage
- 300 kLE PolarFire FPGA in an FCG1152 package (MPF300T-1FCG1152)

![LC-MPFS-DEV-KIT Board](./images/lc-mpfs-dev-kit-user-guide/lc-mpfs-dev-kit.jpg)

### Solution Versions

The latest revisions of the FPGA programming files are available [here](http://soc.microsemi.com/download/rsc/?f=Libero_Project_LC-MPFS-DEV-KIT).

## Linux Build Systems

The Linux image for the HiFive Unleashed Platform can be built using either the Buildroot or Yocto build systems. The table below contains links to these build systems.

| Build System | Location                                                                                                   |
| ------------ | ---------------------------------------------------------------------------------------------------------- |
| Buildroot    | [Polarfire SoC Buildroot SDK](https://mi-v-ecosystem.github.io/redirects/repo-polarfire-soc-buildroot-sdk) |
| Yocto        | [Polarfire SoC Yocto SDK](https://mi-v-ecosystem.github.io/redirects/repo-meta-polarfire-soc-yocto-bsp)    |

### Polarfire SoC Buildroot SDK for the HiFive Unleashed Expansion Board

The Polarfire SoC Buildroot SDK is based on the SiFive freedom-u-sdk with modifications to the
device tree to support the HiFive Unleashed Expansion board. It also includes drivers for Microsemi
PCIe, I2C, SPI, MMUART, and GPIO peripherals. See the build systems table above to download the Polarfire SoC Buildroot SDK.
The build procedure is described in the Linux build system's README.

## Board Setup

The following instructions guide you through setting up the LC-MPFS-DEV-KIT.

1. Switch off the power button on the LC-MPFS-DEV-KIT.  
      ![Power Button](./images/lc-mpfs-dev-kit-user-guide/power-on.png)

2. Set the pins in the DIP switch to select MSEL of 1011 (MSEL2 = 0).  
      ![DIP Switch Setting](./images/lc-mpfs-dev-kit-user-guide/dip-switch.png)

      ```ascii
            USB   LED    Mode Select                  Ethernet
      +===|___|==****==+-+-+-+-+-+-+=================|******|====
      |                | | | | |X| |                 |      |
      |                | | | | | | |                 |      |
      |        HFXSEL->|X|X|X|X| |X|                 |______|
      |                +-+-+-+-+-+-+
      |        RTCSEL-----/ 0 1 2 3 <--MSEL
      |
      ```

3. To prepare the SD-card programmed with the bootloader and Linux images, see the Linux build instructions in the Builroot/Yocto top level README for details on creating the content of this SD card.
4. Insert the SD card into the SD card slot J10.
5. Connect the micro USB cable from J7 to the Host PC. The USB connector has two serial interfaces: the higher index serial port is used for the Linux serial console and the lower index serial port is used for JTAG debug.  
      ![USB Connector](./images/lc-mpfs-dev-kit-user-guide/usb-connector.png)

6. The LC-MPFS-DEV-KIT is now configured as seen in Libero Block Diagram.
7. Ensure the push-button is switched on, connect the power supply to the board, and slide the power switch SW3 as shown in the following figure.  
      ![Power on the Device](./images/lc-mpfs-dev-kit-user-guide/power-on.png)

8. Configure the serial terminal in the Host PC for 115200 baud, 8 data bits, no stop bits, no parity, and no flow control. Push reset button (near the power button) on the LC-MPFS-DEV-KIT.

### Linux Boot and Login Credentials

The Linux boot process can be observed by connecting a serial terminal to the USB port on the HiFive
Unleashed board. Settings are 115200 baud, 8 data bits, 1 stop bit, no parity, and no flow control.
The root password is “microchip” for Buildroot & Yocto `mpfs-dev-cli`. No password is set for Yocto `core-image*`. The console should look similar to the following figure.

![Console Image for Boot](./images/lc-mpfs-dev-kit-user-guide/console-image-for-boot.png)

## Programming Guide

The following sections explain the step-by-step procedure to download the FPGA bitstream onto the PolarFire FPGA.

### Programming the FPGA using FlashPro

#### Windows Environment

To program the PolarFire SoC device with the .job programming file (using FlashPro in Windows environment), perform the following steps.  
The link to the .job file is given in Software Versions. Ensure that the jumpers J13, J21, J28, and J31 are plugged in.
Note: The power supply switch must be switched off while making the jumper connections.

1. Connect the power supply cable to the J3 connector on the board.
2. Connect the FlashPro4 to a PC USB port and to the connector J24 (FP4 header) of the LC-MPFS-DEV-KIT hardware.
3. Power on the board using the SW3 slide switch.
4. On the host PC, launch the FlashPro Express software.
5. Click New or select New Job Project from FlashPro Express Job from Project menu to create a new job project, as shown in the following figure.
      ![New Flash Pro Project](./images/lc-mpfs-dev-kit-user-guide/fp-new.png)

6. Enter the following in the New Job Project from FlashPro Express Job dialog box:
      Programming job file: Click Browse, and navigate to the location where the .job file is located and select the file. The default location is `<download_folder>\mpf_ac466_eval\splash_df\Programming_Job`.
      FlashPro Express job project location: Click Browse and navigate to the location where you want to save the project.  
      ![New Flash Pro Project](./images/lc-mpfs-dev-kit-user-guide/fp-prompt.png)

7. Click OK. The required programming file is selected and ready to be programmed in the
8. The FlashPro Express window appears as shown in the following Confirm that a programmer number appears in the Programmer field. If it does not, confirm the board connections and click Refresh/Rescan Programmers.  
      ![Scanning and Running the Programmer](./images/lc-mpfs-dev-kit-user-guide/fp-program.png)

9. Click RUN. When the device is programmed successfully, a RUN PASSED status is displayed as shown in the following figure.  
      ![Scanning and Running the Programmer](./images/lc-mpfs-dev-kit-user-guide/fp-programmed.png)

### Linux Environment

To program the PolarFire SoC device with the .job programming file (using FlashPro5 programmer in Linux environment), perform the following steps. The link to the .job file can be found in Software Versions.

1. Ensure that the jumpers J13, J21, J28, and J31 are plugged in.
      Note: The power supply switch must be switched off while making the jumper connections.
2. Connect the power supply cable to the J3 connector on the board.
3. Connect the FlashPro5 to a PC USB port and to the connector J24 (FP4 header) of the board.
4. Power on the board using the SW3 slide switch.
5. On the host PC, launch the FlashPro Express (FP Express) software.
6. From the Project menu, choose Create Job Project from Programming Job.
7. Click Browse to load the Programming Job File and specify your FlashPro Express job project location. Click OK to continue.
8. Save the FlashPro Express job project.
9. Set the Programming Action in the dropdown menu to PROGRAM.
10. Click RUN. Detailed individual programmer and device status information appears in the Programmer List. Your programmer status (PASSED or FAILED) appears in the Programmer Status Bar.

See the [FlashPro Express User Guide](https://www.microchip.com/en-us/products/fpgas-and-plds/fpga-and-soc-design-tools/programming-and-debug/flashpro-and-flashpro-express#Documentation) for more information.

## Building and Loading the Linux Image

For instructions on how to build and load a Linux image, see the Linux build instructions in the Buildroot/Yocto top level README.

### FPGA Design in Libero

The Libero project creates a Processor Subsystem in the FPGA fabric for the U540 processor. The Processor Subsystem supports the following features:

- Chiplink interface to communicate with U540SoC
- SRAM memory of size 64KB
- Peripheral controllers: SPI, MMUART, I2C and GPIO
- AXI4 slave interface to connect user AXI4 complaint slaves

In this Libero design, AXI slaves are connected to the Processor Subsystem using core AXI4 Interconnect.

For example, 8 LSRAM blocks (each of size 64KB) are connected as AXIs laves through the core AXI4 Interconnect.The ChipLink interface uses 125 MHz clock and the AXI interface uses 75 MHz.  
The following figure shows the high-level block diagram of the Libero project implemented on the PolarFire FPGA.  
![LC-MPFS-DEV-KIT Board Block Diagram](./images/lc-mpfs-dev-kit-user-guide/updated-lc-libero-design.png)

#### Memory Map

The IP cores on the LC-MPFS-DEV-KIT are accessible from the RISC-V U540 memory map as listed in the following table.

| Peripheral | Start Address | End Address | Interrupt |
| --- | --- | --- | --- |
| GPIO | 0x2000103000 | 0x2000104000 | 7 |
| SPI_0 | 0x2000107000 | 0x2000108000 | 34 |
| I2C_0 | 0x2000100000 | 0x2000101000 | 35 |
| MMUART_0 | 0x2000104000 | 0x2000105000 | |
| SRAM | 0x2030000000 | 0x203FFFFFFF | |
| AXI_MS0 | 0x2030000000 | 0x203FFFFFFF | |
| AXI_MS1 | 0x2600000000 | 0x263FFFFFFF | |

#### GPIO Pinout

The GPIO implemented in the design is pinned out as a starting point for your custom design implementation. The details of the GPIO is listed in GPIO Pinout.

| GPIO | Function |
| --- | --- |
| 0 | LED4 |
| 1 | LED5  |
| 2 | Not connected |
| 3 | Not connected |
| 4 | SWITCH 9 |
| 5 | SWITCH 10 |
| 6 | Not connected |

## Reference

Visit the following links for further reference reading materials.

### Recommended Reading

[RISC-V User-level ISA Specification](https://riscv.org/specifications/)  
[RISC-V Draft Privileged ISA Specification](https://github.com/riscv/riscv-isa-manual/releases/download/Priv-v1.12/riscv-privileged-20211203.pdf)  
[SiFive FU540-C000 User Manual](https://www.sifive.com/documentation/chips/freedom-u540-c000-manual/)  
[TU0844 Libero SoC PolarFire v2.2 Design Flow Tutorial](https://www.microsemi.com/document-portal/doc_download/1243632-tu0844-libero-soc-polarfire-v2-2-design-flow-tutorial)  
[HiFive Unleashed Getting Started Guide](https://static.dev.sifive.com/pdfjs-1.9.426-dist/web/viewer.html?file=https://static.dev.sifive.com/HiFive-Unleashed-Getting-Started-Guide-v1p1.pdf)  

### Other References

[PolarFire FPGA Documentation](https://www.microchip.com/en-us/products/fpgas-and-plds/fpgas/polarfire-fpgas)  
[Libero SoC PolarFire Documentation](https://www.microchip.com/en-us/products/fpgas-and-plds/fpga-and-soc-design-tools/fpga/libero-software-later-versions)  
[FlashPro User Guide for PolarFire](https://www.microsemi.com/document-portal/doc_download/137626-flashpro-user-guide-for-polarfire)  
[FlashPro Express User Guide for PolarFire](https://www.microchip.com/en-us/products/fpgas-and-plds/fpga-and-soc-design-tools/programming-and-debug/flashpro-and-flashpro-express#Documentation)  
[PolarFire SoC Information](https://www.microchip.com/en-us/products/fpgas-and-plds/system-on-chip-fpgas/polarfire-soc-fpgas)  
[Schematics of LC-MPFS-DEV-KIT](https://www.microsemi.com/document-portal/doc_download/1244485-lc-mpfs-dev-kit-schematics)  

## Technical Support

For technical queries, visit the [Microsemi SoC Customer Portal](https://microchipsupport.force.com/s/), select “PolarFire SoC” under Product Family, “MPFSXXXX” under Device Family and type in the query. Microchip’s technical support team will create a ticket, address the query and track it to completion.
