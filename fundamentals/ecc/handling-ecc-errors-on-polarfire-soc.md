# Handling ECC errors on PolarFire SoC

## Table of contents

- [Handling ECC errors on PolarFire SoC](#handling-ecc-errors-on-polarfire-soc)
  - [Table of contents](#table-of-contents)
  - [Introduction](#introduction)
    - [ECC description](#ecc-description)
  - [ECC on PolarFire SoC](#ecc-on-polarfire-soc)
    - [Recieving ECC errors](#recieving-ecc-errors)
    - [Handling ECC errors](#handling-ecc-errors)
    - [On chip memories](#on-chip-memories)
      - [On chip memories ECC](#on-chip-memories-ecc)
        - [L1 cache ECC errors](#l1-cache-ecc-errors)
        - [L2 cache ECC errors](#l2-cache-ecc-errors)
        - [PCIe ECC errors](#pcie-ecc-errors)
        - [Fabric ECC errors](#fabric-ecc-errors)
    - [Off chip memories](#off-chip-memories)
      - [DDR ECC](#ddr-ecc)
    - [On chip peripherals](#on-chip-peripherals)
      - [Enabling on chip peripheral ECC interrupts](#enabling-on-chip-peripheral-ecc-interrupts)
      - [Handling on chip peripheral ECC interrupts](#handling-on-chip-peripheral-ecc-interrupts)
      - [Peripheral correctable errors](#peripheral-correctable-errors)
      - [Peripheral un-correctable errors](#peripheral-un-correctable-errors)

<a name="introduction"></a>
## Introduction

This document will outline the configuration of Error Correcting Codes (ECC) on PolarFire SoC and how errors are and could be handled.

<a name="ecc-description"></a>
### ECC description

Error Correction Code (ECC) is a method of protecting data from single bit and double bit errors.
A single bit error (also known as correctable error) occurs when a bit in a memory location is flipped (either from 1 to 0 or from 0 to 1).
A double bit error (also known as uncorrectable error) occurs when more than one bit in a memory location is flipped.
The bit flip for a double bit error can be any bit in the given memory location, the bits do not have to be beside one and other.

Bit flips are detected by using parity bits in data, these additional bits reflect the status of the bits in data - like a checksum.
If a bit flip occurs then the parity of data returned does not match the parity stored with data.

When a single bit error occurs data can be corrected based on the parity.
The corrected data is returned as part of the read and bad data is discarded.
Corrected data is automatically written back to memory to prevent a single bit error propigating to a double bit error so no user action is needed.

When a double bit error occurs data cannot be corrected.
The bad data will be returned with the read.
As nothing can be done to fix the data it will not be written back to memory, it is up to the user to take an action based on the ECC error.

<a name="ecc-on-polarfire-soc"></a>
## ECC on PolarFire SoC

ECC is available for all memories susceptable to bit flips and certain peripherals in the MSS.

<a name="recieving-ecc-errors"></a>
### Recieving ECC errors

ECC errors are generated on a parity mismatch. An interrupt will be raised if an ECC error is detected.
There are different interrupts for different types of ECC error and the different memories / peripherals where they can be found.

<a name="handling-ecc-errors"></a>
### Handling ECC errors

Depdending on the ECC error that has occured users may want to take corrective / preventative action to stop bad data from propigating through the system.

In the case of a single bit error bad data is corrected and returned with the read and also written back to memory.
This means the effect of single bit errors is quite small as the bad data can't reach any part of the system and is corrected when found.

In the case of a double bit error no corrective action can be taken. This means bad data will be returned with the read and should not be used.
It isn't possible in most cases (especially in the case of instruction data) to identify what data should be re-read and from what source without a significant overhead.
This means the best and safest solution in the case of a double bit error is to restart the system that has read the data.
This prevents bad data from propigating and also ensures any additional bad data will be overwritten when the system reboots.

One way of achiving this is through ECC interrupts - an interrupt can be raised for either a single or a double bit error.
For a single bit error this could just be an acknowledgement and potentially signal that memory should be scrubbed for additional errors.
For a double bit error a `while (1)` loop can be placed in the interrupt handler which would stall the system until the watchdog triggers causing a reset.

When the Hart Software Services (HSS) is used in a system it will automatically monitor these interrupts.
The HSS will automatically scrub memories periodically to ensure single bit errors don't propigate to double bit errors.
If the HSS detects a double bit error it will restart the affected context.

<a name="on-chip-memories"></a>
### On chip memories

There are several on chip memories available that are susceptable to bit flips and are therefore protected by ECC:

- L1 cache (I-cache, D-cache, I-TIM, D-TIM)
- L2 cache (LIM, Scratchpad, cache)
- PCIe memory
- Fabric SRAM

<a name="on-chip-memories-ecc"></a>
#### On chip memories ECC

ECC is enabled by default for on chip memory and is constantly running.
ECC errors will only be acknowledged by the MSS if the interrupt assocaited with the ECC error is enabled.

For example, a single bit ECC error could occur in the L2 LIM, the corrected data will be returned and written back.
The only way to know this error has occured instantly is to enable the L2 Cache Controller data correction event interrupt on the PLIC (IRQ 3).

<a name="l1-cache-ecc-errors"></a>
##### L1 cache ECC errors

ECC errors for the L1 cache on each hart are reported in the Bus Error Unit (BEU) for the hart.
The BEU is described in the PolarFire SoC Technical Reference Manual.

The BEU will record events related to the L1 instruction and data caches along with TileLink errors.

There is an enable bit in each BEU which configures what errors will generate interrupts.
There is also a count register for all events that have occured to indicate the number of accrued events.
These regsiters can all be cleared by software.

The Hart Software Services (HSS) will automatically configure and monitor the BEU while running to check for ECC errors.

Single bit errors are automatically corrected and written back to memory while double bit errors will just report bad data.

<a name="l2-cache-ecc-errors"></a>
##### L2 cache ECC errors

The L2 cache has several interrupts available on the PLIC:

- Metadata correctable
- Metadata uncorrectable
- Data correctable
- Data uncorrectable

When enabled on the PLIC these interrupts will trigger when the associated event occurs.

I.e if a correctable data error occurs the "Data correctable" interrupt will trigger.

The Hart Software Services will automatically configure the PLIC and monitor the L2 cache interrupts.

Single bit errors are automatically corrected and written back to memory while double bit errors will just report bad data.

<a name="pcie-ecc-errors"></a>
##### PCIe ECC errors

By default PCIe ECC errors are masked by the controller.
The PolarFire PCIe controller can report ECC errors internally and raise an interrupt to the MSS when one is detected.
The PolarFire SoC Linux PCIe driver can return a count of the number of ECC errors that have been generated and their type.

<a name="fabric-ecc-errors"></a>
##### Fabric ECC errors

SRAMs (LSRAM and uSRAM) in the FPGA fabric are protected by ECC.
Each component will have output ECC flags when ECC is enabled.

Note: the fabric SRAM on PolarFire devices does not write back corrected data to memory.
If a single bit error is detected the corrected data will need to be written back to the memory by software.

ECC error flags from fabric memories should be connected to the "F2M" interrupts of the MSS.
There is no handling mechanism in the HSS or in provided software for these memories as they are not static.
If users would like to add checking of fabric memories for ECC they will need to configure their own interrupt handlers for this purpose.

<a name="off-chip-memories"></a>
### Off chip memories

The off chip memory that will be discussed in this document is DDR.

<a name="ddr-ecc"></a>
#### DDR ECC

When using ECC DDR with PolarFire SoC the memory should be zeroized on startup before enabling ECC - this will prevent ECC errors from occuring in un-initialized regions.
There are no direct DDR ECC interrupts on PolarFire SoC.
If a single bit error is detected the DDR controller will automatically write back corrected data to memory.
If a double bit error is detected an AXI read error will be generated, this will result in a trap on the hart reading from DDR.

<a name="on-chip-peripherals"></a>
### On chip peripherals

Several peripherals on PolarFire SoC support ECC. These are:

- MMC
- DDRC
- MAC0
- MAC1
- USB
- CAN0
- CAN1

<a name="enabling-on-chip-peripheral-ecc-interrupts"></a>
#### Enabling on chip peripheral ECC interrupts

By default peripheral ECC interrupts are disabled - there are two steps to enable these interrupts:

1. Set the bit corresponding to the peripheral in the "EDAC_INTEN_CR" register (see the PolarFire SoC Register Map for more information). Note: there are two bits for each peripheral, for example "MMC_1E" is for single bit errors in MMC and "MMC_2E" is for double bit errors in MMC.
2. Enable the "peripheral_ecc_error" and / or "peripheral_ecc_correct" interrupts on the PLIC. Note: there is a single interrupt that will assert for all ECC peripherals, see the section below for more details.

<a name="handling-on-chip-peripheral-ecc-interrupts"></a>
#### Handling on chip peripheral ECC interrupts

<a name="peripheral-correctable-errors"></a>
#### Peripheral correctable errors

When the correable error bit for a peripheral is set in the "EDAC_INTEN_CR" register and the "peripheral_ecc_correct" interrupt is enabled on the PLIC, if a correctable ECC error is detected in the peripheral the "peripheral_ecc_correct" interrupt will assert.
When this happens the "EDAC_SR" register should be read to determine which peripheral has triggered the ECC event.
The ECC correctable interrupt can be cleared by writing 1 to any bits set in the "EDAC_SR" register.
Each peripheral also has an individual count register to highlight the number of correctable ECC errors that have occured on that peripheral.

<a name="peripheral-un-correctable-errors"></a>
#### Peripheral un-correctable errors

When the un-correable error bit for a peripheral is set in the "EDAC_INTEN_CR" register and the "peripheral_ecc_error" interrupt is enabled on the PLIC, if an un-correctable ECC error is detected in the peripheral the "peripheral_ecc_error" interrupt will assert.
When this happens the "EDAC_SR" register should be read to determine which peripheral has triggered the ECC event.
The ECC error interrupt can be cleared by writing 1 to any bits set in the "EDAC_SR" register.

It is up to the user to determine what should be done in the case of an un-correctable error being detected in a peripheral.
