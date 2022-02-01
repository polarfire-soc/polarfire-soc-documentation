# PolarFire SoC Boot Mode 2 Fundamentals

## Overview

PolarFire SoC boot mode 2 is intended for implementing user-defined secure boot authentication.

## Boot Mode 2 Sequence

On power up, the PolarFire SoC System Controller starts up and holds the MSS in reset until it has completed configuring the device. It executes ROM code which configures the Core Complex based on configuration data structures stored in its private Non-Volatile Memory (pNVM).

In boot mode 2, the System Controller uses:

- U_MSS_BOOTMODE: determines whether boot mode 0, 1, 2 or 3 is used.
- U_MSS_BOOTCFG: contains boot configuration specific to the requested boot mode. In the case of boot mode 2, it contains the page offset at which the user boot loader image is stored in secure non volatile memory.

The boot mode 2 sequence is as follows:

1. Registers have the following reset values:
  - Pseudo BOOTROM system registers is a code loop.
  - System registers' reset vectors point to the base address of the pseudo BOOTROM
  - TEMP1 system register is zero.
2. The System Controller releases the Core Complex reset:
  - This causes all harts to execute the code found in the pseudo BOOTROM system registers.
  - The loop will keep executing until the content of the TEMP1 system register remains set to zero.
3. The System Controller copies the User Boot Loader (UBL) image from sNVM to on-chip RAM:
  - The location of the UBL image in sNVM is specified in the U_MSS_BOOTCFG
  - The target address where the UBL is to be copied within on-chip RAM is found at the top of the UBL image.
  - The length to copy is also found at the top of the UBL image
4. The System Controller sets the value of the TEMP1 System Register to the address of the UBL vector table within on-chip RAM:
  - This results in the Core Complex harts executing the user boot loader.

The user boot loader typically authenticates the content of the eNVM before executing it.

![](./images/boot-mode-2.png)

Notes:

- The sNVM used to store the user boot loader must be marked as ROM in Libero in order to avoid it being modified by anything other than a suitable programming bitstream.
- A digest check is performed by the System Controller to verify the validity of the boot configuration data when copying. The system controller will set the device to boot mode 0 and also set the boot_fail tamper flag if the re-computed hash of the boot configuration data does not match the computed hash of the user bootloader.
- The MSS/Core Complex default clock configuration is 80MHz using the SCB clock source.
- The System Controller's pNVM and sNVM content can only be modified through a programming bitstream. Neither pNVM nor sNVM content are directly accessible from the Core Complex.
- The user boot loader image length held in sNVM is the number of 32 bits words of the executable unlike the boot mode 3 SBIC image length which is a byte count.
