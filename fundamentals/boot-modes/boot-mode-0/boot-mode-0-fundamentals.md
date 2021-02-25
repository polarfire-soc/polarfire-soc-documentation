# PolarFire SoC Boot Mode 0 Fundamentals

## Overview
PolarFire SoC boot mode 0 is intended to handle blank devices. It prevents the MSS from executing invalid code until the debugger loads a valid executable in memory. Boot mode 0 puts the MSS in a mode where all harts execute a loop waiting for the debugger to connect through JTAG or for the device to be  programmed.

Boot mode 0 can also be useful when debugging low level software through JTAG in order to prevent the system state from being modified between the system being powered up and the debug session starting.


## Boot Mode 0 Sequence
The PolarFire SoC System Controller executes first before the Coreplex harts when the device comes out of power-on reset. 

The System Controller configures the PolarFire SoC device before releasing the Coreplex out of reset. It executes ROM code which configures the Coreplex based on configuration data structures stored in its private Non-Volatile Memory (pNVM). In boot mode 0, the System Controller only uses the U_MSS_BOOT_CFG configuration item stored in pNVM to control the Coreplex boot process: The U_MSS_BOOTMODE configuration item determines whether boot mode 0, 1, 2 or 3 is used.

The mode 0 boot sequence is as follows:

- The reset value of the pseudo BOOTROM system registers is a code loop. 
- The reset value of the system registers' reset vectors point to the base address of the pseudo BOOTROM
- The reset value of the TEMP1 system register is zero.
- The System Controller releases the Coreplex reset causing all harts to execute the code found in the pseudo BOOTROM system registers. The loop will keep executing until the content of the TEMP1 system register remains set to zero.






![](./images/boot-mode-0.png) 

The Coreplex harts will remain executing the pseudo BOOTROM loop until the debugger sets the hart's program counters to new values.

Notes:

- The System Controller's pNVM content can only be modified through a programming bitstream. Neither pNVM nor sNVM content are directly accessible from the Coreplex.
- The MSS/Coreplex default clock configuration is 80MHz using the SCB clock source.



