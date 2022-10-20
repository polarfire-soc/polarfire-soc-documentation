# Handling ECC errors on PolarFire SoC

## Table of contents

- [Handling ECC errors on PolarFire SoC](#handling-ecc-errors-on-polarfire-soc)
  - [Table of contents](#table-of-contents)
  - [Introduction](#introduction)
    - [ECC Description](#ecc-description)
  - [ECC on PolarFire SoC](#ecc-on-polarfire-soc)
    - [Memories supporting ECC](#memories-supporting-ecc)
    - [Peripherals supporting ECC](#peripherals-supporting-ecc)
    - [Receiving ECC Errors](#receiving-ecc-errors)
      - [ECC Interrupts](#ecc-interrupts)
    - [Handling ECC Errors](#handling-ecc-errors)
    - [On-chip Memories](#on-chip-memories)
      - [On-chip Memories ECC](#on-chip-memories-ecc)
        - [L1 Cache ECC Errors](#l1-cache-ecc-errors)
        - [L2 Cache ECC Errors](#l2-cache-ecc-errors)
        - [PCIe ECC Errors](#pcie-ecc-errors)
        - [Fabric ECC Errors](#fabric-ecc-errors)
    - [Off-Chip Memory](#off-chip-memory)
      - [DDR ECC](#ddr-ecc)
    - [ECC Support in MSS Peripherals](#ecc-support-in-mss-peripherals)
      - [Enabling ECC Interrupts from MSS Peripherals](#enabling-ecc-interrupts-from-mss-peripherals)
      - [Handling ECC Interrupts from MSS Peripherals](#handling-ecc-interrupts-from-mss-peripherals)
        - [Correctable ECC Errors on MSS Peripherals](#correctable-ecc-errors-on-mss-peripherals)
        - [Un-correctable ECC Errors on MSS Peripherals](#un-correctable-ecc-errors-on-mss-peripherals)

<a name="introduction"></a>

## Introduction

This document will outline the configuration and handling of Error Correcting Codes (ECC) and their assocaited errors on PolarFire.

<a name="ecc-description"></a>

### ECC Description

Error Correction Code (ECC) is a method of protecting data from single bit and double bit errors.
A single bit error (also known as correctable error) occurs when a bit in a memory location is flipped (either from 1 to 0 or from 0 to 1).
A double bit error (also known as uncorrectable error) occurs when more than one bit in a memory location is flipped.
The bit flips for a double bit error can be any bits in the given memory location, the bits do not have to be beside one and other.

Bit flips are detected by using additional parity bits in data, which function like a checksum.
If a bit flip occurs then the parity of data returned does not match the parity stored with data.

When a single bit error occurs data can be corrected based on the parity.
The corrected data is returned as part of the read and bad data is discarded.
Corrected data is automatically written back to memory to prevent a single bit error propagating to a double bit error so no user action is needed.

When a double bit error occurs data cannot be corrected.
The bad data will be returned with the read.
As nothing can be done to fix the data it will not be written back to memory, it is up to the user to take an action based on the ECC error.

<a name="ecc-on-polarfire-soc"></a>

## ECC on PolarFire SoC

ECC is available for all memories susceptable to bit flips and certain peripherals in the MSS.

<a name="memories-supporting-ecc"></a>

### Memories supporting ECC

| Memory      | Error flag                                              |
|-------------|---------------------------------------------------------|
| L1 Cache    | BEU interrupt                                           |
| L2 Cache    | PLIC interrupt                                          |
| PCIe        | Interrupt from the fabric (typically taken by the PLIC) |
| Fabric SRAM | Interrupt from the fabric (typically taken by the PLIC) |

<a name="peripherals-supporting-ecc"></a>

### Peripherals supporting ECC

| Peripheral | Error flag                      |
|------------|---------------------------------|
| MMC        | Aggregated in EDAC_SR register  |
| DDRC       | Aggregated in EDAC_SR register  |
| MAC 0      | Aggregated in EDAC_SR register  |
| MAC 1      | Aggregated in EDAC_SR register  |
| USB        | Aggregated in EDAC_SR register  |
| CAN 0      | Aggregated in EDAC_SR register  |
| CAN 1      | Aggregated in EDAC_SR register  |

<a name="receiving-ecc-errors"></a>

### Receiving ECC Errors

ECC errors are generated on a parity mismatch. An interrupt will be raised if an ECC error is detected.
There are different interrupts for different types of ECC error and the different memories / peripherals where they can be found.

<a name="ecc-interrupts"></a>

#### ECC Interrupts

The table below lists the different interrupts associated with ECC along with their PLIC and local interrupt numbering:

| Interrupt name                  | Description                                                                         | PLIC IRQ number | Local IRQ number | Local IRQ on hart |
|---------------------------------|-------------------------------------------------------------------------------------|-----------------|------------------|-------------------|
| L2 cache metadata correct       | A single bit correctable error has been detected and corrected in L2 cache metadata | 1               |                  |                   |
| L2 cache metadata uncorrectable | An uncorrectable double bit error has been detected in L2 cache metadata            | 2               |                  |                   |
| L2 cache data correct           | A single bit correctable error has been detected and corrected in L2 cache data     | 3               |                  |                   |
| L2 cache data uncorrectable     | An uncorrectable double bit error has been detected in L2 cache data                | 4               |                  |                   |
| peripheral_ecc_error            | An uncorrectable double bit error has been detected in a peripheral                 | 78              | 14               | E51               |
| peripheral_ecc_correct          | A single bit correctable error has been detected and corrected in a peripheral      | 79              | 13               | E51               |
| BEU hart 0                      | A BEU event has occurred on the E51                                                 | 182             | Trap, see below  | E51               |
| BEU hart 1                      | A BEU event has occurred on the U54_1                                               | 183             | Trap, see below  | U54_1             |
| BEU hart 2                      | A BEU event has occurred on the U54_2                                               | 184             | Trap, see below  | U54_2             |
| BEU hart 3                      | A BEU event has occurred on the U54_3                                               | 185             | Trap, see below  | U54_3             |
| BEU hart 4                      | A BEU event has occurred on the U54_4                                               | 186             | Trap, see below  | U54_4             |

Note: the BEU local interrupt has a different behaviour to the standard local interrupts, it will present as a trap with `MCAUSE` of 128.

<a name="handling-ecc-errors"></a>

### Handling ECC Errors

Depdending on the ECC error that has occurred, users may want to take corrective / preventative action to stop bad data from propagating through the system.

In the case of a single bit error, bad data is corrected and the corrected data is returned with the read.
The corrected data will automatically be written back to memory.
This means the effect of single bit errors is quite small as the bad data can't reach any part of the system and is corrected when found.

In the case of a double bit error no corrective action can be taken. This means bad data will be returned with the read and should not be used.
It isn't possible in most cases (especially in the case of instruction data) to identify what data should be re-read and from what source without a significant overhead.
This means the best and safest solution in the case of a double bit error is to restart the system that has read the data.
This prevents bad data from propagating and also ensures any additional bad data will be overwritten when the system reboots.

One way of achieving this is through ECC interrupts - an interrupt can be raised for either a single or a double bit error.
For a single bit error this could just be an acknowledgement and potentially signal that memory should be scrubbed for additional errors.
For a double bit error a `while (1)` loop can be placed in the interrupt handler which would stall the system until the watchdog triggers causing a reset.

When the Hart Software Services (HSS) is used in a system it will automatically monitor these interrupts.
The HSS will automatically scrub memories periodically to ensure single bit errors don't propigate to double bit errors.
If the HSS detects a double bit error it will restart the affected context.

<a name="on-chip-memories"></a>

### On-chip Memories

There are several on-chip memories available that are susceptable to bit flips and are therefore protected by ECC:

- L1 cache (I-cache, D-cache, I-TIM, D-TIM)
- L2 cache (LIM, Scratchpad, cache)
- PCIe memory
- Fabric SRAM

<a name="on-chip-memories-ecc"></a>

#### On-chip Memories ECC

ECC is enabled by default for on-chip memory and is constantly running.
ECC errors will only be acknowledged by the MSS if the interrupt assocaited with the ECC error is enabled.

For example, if a single bit ECC error occurs in the L2 LIM, the corrected data will be returned and written back.
The only way to know this error has occured instantly is to enable the L2 Cache Controller data correction event interrupt on the PLIC (IRQ 3).

<a name="l1-cache-ecc-errors"></a>

##### L1 Cache ECC Errors

ECC errors for the L1 cache of each hart are reported in the Bus Error Unit (BEU) on a per hart basis.
The BEU is described in the PolarFire SoC [Technical Reference Manual](https://ww1.microchip.com/downloads/aemDocuments/documents/FPGA/ProductDocuments/ReferenceManuals/PolarFire_SoC_FPGA_MSS_Technical_Reference_Manual_VC.pdf).

The BEU will record events related to the L1 instruction and data caches along with TileLink errors.

There is an enable bit in each BEU which configures what errors will generate interrupts.
There is also a count register for all events that have occured to indicate the number of accrued events.
These regsiters can all be cleared by software.

The Hart Software Services (HSS) will automatically configure and monitor the BEU while running to check for ECC errors.

Single bit errors are automatically corrected and written back to memory while double bit errors will just report bad data.

<a name="l2-cache-ecc-errors"></a>

##### L2 Cache ECC Errors

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

##### PCIe ECC Errors

By default PCIe ECC errors are masked by the controller.
The PolarFire PCIe controller can report ECC errors internally and raise an interrupt to the MSS when one is detected.
The PolarFire SoC Linux PCIe driver can return a count of the number of ECC errors that have been generated and their type.

<a name="fabric-ecc-errors"></a>

##### Fabric ECC Errors

SRAMs (LSRAM and uSRAM) in the FPGA fabric are protected by ECC.
When ECC is enabled each component will have output pins which act as ECC flags.

Note: the fabric SRAM on PolarFire devices does not write back corrected data to memory.
If a single bit error is detected the corrected data will need to be written back to the memory by software.

ECC error flags from fabric memories should be connected to the "F2M" interrupts of the MSS.
There is no handling mechanism in the HSS or in provided software for these memories as they are not static.
If users would like to add checking of fabric memories for ECC they will need to configure their own interrupt handlers for this purpose.

<a name="off-chip-memory"></a>

### Off-Chip Memory

The off-chip memory that will be discussed in this document is DDR connected to the MSS DDR controller.

<a name="ddr-ecc"></a>

#### DDR ECC

When using DDR with sideband ECC configured, the DDR memory needs to be initialized.
This should take place after DDR training has completed but before the DDR is used by an application.
An initialization function will be automatically be run when sideband DRR is configured. This will be available from the 2022.12 release of the MPFS HAL and included in the 2023.2 release of the Hart Software Services.

Note: some non-typical DDR has built in ECC.
It is up to the user to check the data sheet regarding initialization requirements in this case.
The MPFS HAL DDR initialization routine can be used when required.

There are no direct DDR ECC interrupts on PolarFire SoC.

If a single bit error is detected the DDR controller will automatically write back corrected data to memory.

If a double bit error is detected an error will be generated in the BEU, this will result in a trap on the hart reading from DDR.

<a name="ecc-support-in-mss-peripherals"></a>

### ECC Support in MSS Peripherals

Several peripherals on PolarFire SoC support ECC. These are:

- MMC
- DDRC
- MAC0
- MAC1
- USB
- CAN0
- CAN1

<a name="enabling-ecc-interrupts-from-mss-peripherals"></a>

#### Enabling ECC Interrupts from MSS Peripherals

By default peripheral ECC interrupts are disabled - there are two steps to enable these interrupts:

1. Set the bit corresponding to the peripheral in the "EDAC_INTEN_CR" register (see the PolarFire SoC Register Map -> "g5_mss_top_sysreg" for more information). Note: there are two bits for each peripheral, for example "MMC_1E" is for single bit errors in MMC and "MMC_2E" is for double bit errors in MMC.
2. Enable the "peripheral_ecc_error" and / or "peripheral_ecc_correct" interrupts on the PLIC. Note: there is a single interrupt that will assert for all ECC peripherals, see the section below for more details.

<a name="handling-ecc-interrupts-from-mss-peripherals"></a>

#### Handling ECC Interrupts from MSS Peripherals

<a name="correctable-ecc-errors-on-mss-peripherals"></a>

##### Correctable ECC Errors on MSS Peripherals

When the correable error bit for a peripheral is set in the "EDAC_INTEN_CR" register and the "peripheral_ecc_correct" interrupt is enabled on the PLIC, if a correctable ECC error is detected in the peripheral the "peripheral_ecc_correct" interrupt will assert.
When this happens the "EDAC_SR" register should be read to determine which peripheral has triggered the ECC event.
The ECC correctable interrupt can be cleared by writing 1 to any bits set in the "EDAC_SR" register.
Each peripheral also has an individual count register to highlight the number of correctable ECC errors that have occured on that peripheral.

<a name="un-correctable-ecc-errors-on-mss-peripherals"></a>

##### Un-correctable ECC Errors on MSS Peripherals

When the un-correctable error bit for a peripheral is set in the "EDAC_INTEN_CR" register and the "peripheral_ecc_error" interrupt is enabled on the PLIC, if an un-correctable ECC error is detected in the peripheral the "peripheral_ecc_error" interrupt will assert.
When this happens the "EDAC_SR" register should be read to determine which peripheral has triggered the ECC event.
The ECC error interrupt can be cleared by writing 1 to any bits set in the "EDAC_SR" register.

It is up to the user to determine what should be done in the case of an un-correctable error being detected in a peripheral.
