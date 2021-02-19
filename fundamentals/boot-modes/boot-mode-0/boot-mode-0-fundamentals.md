# PolarFire SoC Boot Mode 0 Fundamentals

##Overview
PolarFire SoC boot mode 0 is intended to be used where either the Microprocessor Subsystem (MSS) is not used or when debugging software through JTAG. Boot mode 0 puts the MSS in a mode where all harts execute a loop waiting for a reset vector to be set by the debugger through JTAG. It prevents the MSS from executing invalid code until the debugger loads a valid executable in memory.

## Boot Mode 0 Sequence
The PolarFire SoC System Controller executes first before the Coreplex harts when the device comes out of power-on reset. 

The System Controller configures the PolarFire SoC device before releasing the Coreplex out of reset. It executes ROM code which configures the Coreplex based on configuration data structures stored in its Private Non-Volatile Memory (pNVM). In boot mode 0, the System Controller only uses the U_MSS_BOOT_CFG configuration item stored in pNVM to control the Coreplex boot process: The U_MSS_BOOTMODE configuration item determines whether boot mode 0, 1, 2 or 3 is used.

For boot mode 0, the System Controller:

- Loads a code loop into the pseudo BOOTROM system registers. Please note that the reset value of the system registers' reset vectors point to the base address of the pseudo BOOTROM
- Sets the value of the TEMP1 system register to zero.
- Releases the Coreplex reset causing all harts to execute the code found in the pseudo BOOTROM system registers. The loop will keep executing until the content of the TEMP1 system register remains set to zero.

![](./images/boot-mode-0.png) 

The Coreplex harts will remain executing the pseudo BOOTROM loop until the value of the TEMP1 system register is modified by the debugger to point to a vector table indexed on the hart's ID. Each hart will then execute code from the address found in its associated entry within the vector table.

Please note that the System Controller's pNVM and sNVM content can only be modified through a programming bitstream. Neither pNVM nor sNVM content is directly accessible from the Coreplex.

