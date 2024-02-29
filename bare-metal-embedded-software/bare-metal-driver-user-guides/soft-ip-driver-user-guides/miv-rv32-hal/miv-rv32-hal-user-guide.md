<html>
 
 ------------------------------------ 

# MIV_RV32 Hardware Abstraction Layer
 
-----------------------------------------
## Table of Contents 
  - [Introduction](#introduction)

     - [MIV_RV32 V3.1](#miv_rv32-v3.1)

  - [Customizing MIV_RV32 HAL](#customizing-miv_rv32-hal)

     - [Interrupt Handling](#interrupt-handling)

     - [MIE Register Map for MIV_RV32 V3.0](#mie-register-map-for-miv_rv32-v3.0)

     - [Floating Point Interrupt Support](#floating-point-interrupt-support)

     - [SUBSYS - SubSystem for RISC-V](#subsys---subsystem-for-risc-v)

- [Types](#types)

- [Constants](#constants)
  - [SUBSYS Backwards Compatibility](#subsys-backwards-compatibility)
  - [MTIME Timer Interrupt Constants](#mtime-timer-interrupt-constants)
      - [MTIME and MTIMECMP](#mtime-and-mtimecmp)
      - [MIV_RV32_EXT_TIMER](#mivrv32exttimer)
      - [MIV_RV32_EXT_TIMECMP](#mivrv32exttimecmp)
  - [External IRQ](#external-irq)
  - [RISC-V Specification Interrupts](#risc-v-specification-interrupts)
  - [BootROM](#bootrom)
  - [SUBSYS Register Configuration](#subsys-register-configuration)
  - [SUBSYS Interrupt Request Masks](#subsys-interrupt-request-masks)

- [Functions](#functions)
  - [MRV32_clear_gpr_ecc_errors](#mrv32cleargpreccerrors)
  - [MRV32_mgeui_clear_irq](#mrv32mgeuiclearirq)
  - [MRV32_mgeci_clear_irq](#mrv32mgeciclearirq)
  - [MRV_enable_local_irq](#mrvenablelocalirq)
  - [MRV_disable_local_irq](#mrvdisablelocalirq)
  - [MRV_enable_interrupts](#mrvenableinterrupts)
  - [MRV_disable_interrupts](#mrvdisableinterrupts)
  - [MRV_read_mtvec_base](#mrvreadmtvecbase)
  - [MRV_set_mtvec_base](#mrvsetmtvecbase)
  - [MRV_read_mtime](#mrvreadmtime)
  - [MRV_raise_soft_irq](#mrvraisesoftirq)
  - [MRV_clear_soft_irq](#mrvclearsoftirq)
  - [SysTick_Handler](#systickhandler)
  - [MRV_systick_config](#mrvsystickconfig)
  - [MRV_BootROM_reconfigure](#mrvbootromreconfigure)
  - [MRV32_subsys_enable_irq](#mrv32subsysenableirq)
  - [MRV32_subsys_disable_irq](#mrv32subsysdisableirq)
  - [MRV32_subsys_clear_irq](#mrv32subsysclearirq)
  - [MRV32_subsys_irq_cause](#mrv32subsysirqcause)
  - [MRV32_is_gpr_ded](#mrv32isgprded)
  - [MRV32_clear_gpr_ded](#mrv32cleargprded)
  - [MRV32_enable_parity_check](#mrv32enableparitycheck)
  - [MRV32_disable_parity_check](#mrv32disableparitycheck)
  - [MRV32_cpu_soft_reset](#mrv32cpusoftreset)

<div id="TitlePage" data-type="text">

# Introduction
This document describes the Hardware Abstraction Layer (HAL) for the MIV_RV32
Soft IP Core. This release of the HAL corresponds to the Soft IP core MIV_RV32
v3.1 release. It also supports earlier versions of the MIV_RV32 as well as the
legacy RV32 IP cores. The preprocessor macros provided with the MIV_RV32 HAL are
used to customize it to target the Soft Processor IP version being used in your
project.

The term "MIV_RV32" represents following two cores:

  - MIV_RV32 v3.0 and later (the latest and greatest Mi-V soft processor)  


  - MIV_RV32IMC v2.1 (MIV_RV32 v3.0 is a drop in replacement for this core) It 
is highly recommended to migrate your design to MIV_RV32 v3.1

The term, Legacy RV32 IP cores, represents following IP cores:

  - MIV_RV32IMA_L1_AHB  


  - MIV_RV32IMA_L1_AXI  


  - MIV_RV32IMAF_L1_AHB

These legacy RV32 IP cores are deprecated. It is highly recommended to migrate
your designs to MIV_RV32 v3.1 (and subsequent IP releases) for the latest
enhancements, bug fixes, and support.

## MIV_RV32 V3.1
This is the latest release of the MIV_RV32 Soft IP core. For more details, see the MIV_RV32 User 
<a href="https://www.microchip.com/en-us/products/fpgas-
and-plds/ip-core-tools/miv-rv32">Guide</a>

The MIV_RV32 Core and this document use the following terms:

  - SUBSYS - Processor Subsystem for RISC-V
  - OPSRV - Offload Processor Subsystem for RISC-V
  - GPR - General Purpose Registers
  - MGECIE - Machine GPR ECC Correctable Interrupt Enable
  - MGEUIE - Machine GPR ECC Uncorrectable Interrupt Enable
  - MTIE - Machine Timer Interrupt Enable
  - MEIE - Machine External Interrupt Enable
  - MSIE - Machine Software Interrupt Enable
  - ISR - Interrupt Service Routine

# Customizing MIV_RV32 HAL
To use the HAL with older releases of MIV_RV32 preprocessor, macros have been
provided. Using these macros, any of the IP version is targeted. The HAL is used
to target the mentioned platforms by adding the following macros in Project 
Properties > C/C++ Build > Settings > Preprocessor  available in the Assembler 
and Compiler settings. The following table shows the macros corresponding to the
MIV Core being used in your libero project. By default, the HAL targets v3.1 of 
the IP core and no macros need to be set for this configutation.

| Libero MI-V Soft IP Version    | SoftConsole Macro     | 
| -----|-----|
| MIV_RV32 v3.1    | no macro required     | 
| MIV_RV32 v3.0    | MIV_CORE_V3_0     | 
| Legacy RV32 Cores    | MIV_LEGACY_RV32    | 

## Interrupt Handling
The MIE Register is defined as a enum in the HAL, and the table below is used as
a reference when the vectored interrupts are enabled in the GUI core
configurator.

The MIE register is a RISC-V Control and Status Register (CSR), which stands for
the Machine Interrupt Enable. This is used to enable the machine mode interrupts
in the MIV_RV32 hart. Refer to the RISC-V Priv spec for more details.

The following table shows the trap entry addresses when an interrupt occurs and
the vectored interrupts are enabled in the GUI configurator.

| MIE Register Bit    | Interrupt Enable    | Vector Address     | 
| -----|-----|-----|
| 31    | MSYS_IE7    | mtvec.BASE + 0x7C     | 
| 30    | MSYS_IE6    | mtvec.BASE + 0x78     | 
| 29    | MSYS_IE5    | mtvec.BASE + 0x74     | 
| 28    | MSYS_IE4    | mtvec.BASE + 0x70     | 
| 27    | MSYS_IE3    | mtvec.BASE + 0x6C     | 
| 26    | MSYS_IE2    | mtvec.BASE + 0x68     | 
| 25    | MSYS_IE1    | mtvec.BASE + 0x64     | 
| 24    | MSYS_IE0    | mtvec.BASE + 0x60     | 
| 23    | SUBSYS_EI    | mtvec.BASE + 0x5C     | 
| 22    | SUBSYSR    | mtvec.BASE + 0x58     | 
| 17    | MGECIE    | mtvec.BASE + 0x44     | 
| 16    | MGEUIE    | mtvec.BASE + 0x40     | 
| 11    | MEIE    | mtvec.BASE + 0x2C     | 
| 7    | MTIE    | mtvec.BASE + 0x1C     | 
| 3    | MSIE    | mtvec.BASE + 0x0C    | 

For changes in MIE register map, see the [MIE Register Map for MIV_RV32 v3.0](#mie-register-map-for-miv_rv32-v3.0) section.

SUBSYSR is currently not being used by the core and is Reserved for future use.

The mtvec.BASE field corresponds to the bits [31:2], where mtvec stands for
Machine Trap Vector, and all traps set the PC to the value stored in the
mtvec.BASE field when in Non-Vectored mode. In this case, a generic trap handler
is as an interrupt service routine.

When Vectored interrupts are enabled, use this formula to calculate the trap
address: (mtvec.BASE + 4*cause), where cause comes from the mcause CSR. The
mcause register is written with a code indicating the event that caused the
trap. For more details, see the RISC-V priv specification.

The MIV_RV32 Soft IP core does not contain a Platfrom Level Interrup Controller
(PLIC). It is advised to use the PLIC contained within the MIV_ESS sub-system.
Connect the PLIC interrupt output of the MIV_ESS to the EXT_IRQ pin on the
MIV_RV32.

The following table is the MIE register map for the MIV_RV32 Core V3.0. It only
highlights the differences between the V3.0 and V3.1 of the core.

## MIE Register Map for MIV_RV32 V3.0
| MIE Register Bit    | Target Interrupt    | Vector Address     | 
| -----|-----|-----|
| 31    | Not in use    | top table     | 
| 30    | SUBSYS_EI    | addr + 0x78     | 
| 23    | Not in use    | Not in use     | 
| 22    | Not in use    | Not in use    | 

Other interrupt bit postions like the MGEUIE and MSYS_IE5 to MSYS_IE0 remain
unchanged.

## Floating Point Interrupt Support
When an interrupt is taken and Floating Point instructions are used in the ISR,
the floating point register context must be saved to resume the application
correctly. To use this feature, enable the provided macro in the Softconsole
build settings. This feature is turned off by default as it adds overhead which
is not required when the ISR does not use FP insturctions and saving the
general purpose register context is sufficient.

| Macro Name    | Definition     | 
| -----|-----|
| MIV_FP_CONTEXT_SAVE    | Define to save the FP register file    | 

## SUBSYS - SubSystem for RISC-V
SUBSYS stands for SubSystem for RISC-V. Refer to the MIV_RV32 v3.1 Handbook for 
more details.  
NOTE: This was previously (MIV_RV32 v3.0) known as OPSRV, which stands for 
"Offload Processor Subsystem for RISC-V". See the earlier versions of the 
handbook for more details. The MIV_RV32 HAL now uses SUBSYS instead of OPSRV.

</div>


# Types

 ---------------- 

# Constants

 ---------------- 
<div id="Constants$OPSRV_TCM_ECC_CE_IRQ$description" data-type="text">

<a name="opsrvtcmeccceirq"></a>
## SUBSYS Backwards Compatibility
For application code using the older macro names and API functions, these macros
act as a compatibility layer and applications which use OPSRV API features work
due to these macro definitions. However, it is adviced to update your
application code to use the SUBSYS macros and API functions.

| Macro Name    | Now Called     | 
| -----|-----|
| OPSRV_TCM_ECC_CE_IRQ    | SUBSYS_TCM_ECC_CE_IRQ     | 
| OPSRV_TCM_ECC_UCE_IRQ    | SUBSYS_TCM_ECC_UCE_IRQ     | 
| OPSRV_AXI_WR_RESP_IRQ    | SUBSYS_AXI_WR_RESP_IRQ     | 
| MRV32_MSYS_OPSRV_IRQn    | MRV32_SUBSYS_IRQn     | 
| MRV32_opsrv_enable_irq    | MRV32_subsys_enable_irq     | 
| MRV32_opsrv_disable_irq    | MRV32_subsys_disable_irq     | 
| MRV32_opsrv_clear_irq    | MRV32_subsys_clear_irq     | 
| OPSRV_IRQHandler    | SUBSYS_IRQHandler    | 

## MTIME Timer Interrupt Constants
These values contain the register addresses for the registers used by the
machine timer interrupt

MTIME_PRESCALER is not defined on the MIV_RV32IMC v2.0 and v2.1. By using this
definition the system crashes. For those core, use the following definition:

#define MTIME_PRESCALER 100u

<a name="mtime-and-mtimecmp"></a>
### MTIME and MTIMECMP
MIV_RV32 core offers flexibility in terms of generating MTIME and MTIMECMP
registers internal to the core or using external time reference. There four
possible combinations:

  - Internal MTIME and Internal MTIME IRQ enabled Generate the MTIME and 
MTIMECMP registers internally. (The only combination available on legacy RV32 
cores)
  - Internal MTIME enabled and Internal MTIME IRQ disabled Generate the MTIME 
internally and have a timer interrupt input to the core as external pin. In this
 case, 1 pin port will be available on MIV_RV32 for timer interrupt.
  - When the internal MTIME is disabled, and the Internal MTIME IRQ is enabled, 
the system generates the time value externally and generates the mtimecmp and 
interrupt internally (for example, a multiprocessor system with a shared time 
between all cores). In this case, a 64-bit port is available on the MIV_RV32 
core as input.
  - Internal MTIME and Internal MTIME IRQ disabled Generate both the time and 
timer interrupts externally. In this case a 64 bit port will be available on the
 MIV_RV32 core as input, and a 1 pin port will be available for timer interrupt.

To handle all these combinations in the firmware, the following constants must
be defined in accordance with the configuration that you have made on your
MIV_RV32 core design.

<a name="mivrv32exttimer"></a>
### MIV_RV32_EXT_TIMER
When defined, it means that the MTIME register is not available internal to the
core. In this case, a 64 bit port will be available on the MIV_RV32 core as
input. When this macro is not defined, it means that the MTIME register is
available internally to the core.

<a name="mivrv32exttimecmp"></a>
### MIV_RV32_EXT_TIMECMP
When defined, it means the MTIMECMP register is not available internally to the
core and the Timer interrupt input to the core comes as an external pin. When
this macro is not defined it means the that MTIMECMP register exists internal to
the core and that the timer interrupt is generated internally.

NOTE: All these macros must not be defined if you are using a MIV_RV32 core.

</div>

<div id="Constants$EXT_IRQ_KEEP_ENABLED$description" data-type="text">

<a name="extirqkeepenabled"></a>
## External IRQ
Return value from External IRQ handler. This is used to disable the External
Interrupt.

| Macro Name    | Value    | Description     | 
| -----|-----|-----|
| EXT_IRQ_KEEP_ENABLED    | 0    | Keep external interrupts enabled     | 
| EXT_IRQ_DISABLE    | 1    | Disable external interrupts    | 

</div>

<div id="Constants$MRV32_SOFT_IRQn$description" data-type="text">

<a name="mrv32softirqn"></a>
## RISC-V Specification Interrupts
These definitions are provided for easy identification of the interrupt in the
MIE/MIP registers. Apart from the standard software, timer, and external
interrupts, the names of the additional interrupts correspond to the names as
used in the MIV_RV32 handbook. Please refer the MIV_RV32 handbook for more
details.

All the interrups, provided by the MIV_RV32 core, following table shows the 
interrupt priority order and register description as mentioned in the RISC-V spec.

| Macro Name    | Value    | Description     | 
| -----|-----|-----|
| MRV32_SOFT_IRQn    | MIE_3_IRQn    | Software interrupt enable     | 
| MRV32_TIMER_IRQn    | MIE_7_IRQn    | Timer interrupt enable     | 
| MRV32_EXT_IRQn    | MIE_11_IRQn    | External interrupt enable    | 

</div>

<div id="Constants$BOOTROM_START$description" data-type="text">

<a name="bootromstart"></a>
## BootROM
When BootROM is enabled, on reset, the core copies data from a memory mapped
source memory into a destination memory location and then the core boots from 
the destination memory location. The source start or end addresses and the
destination start address can be provided through GUI inputs. If the
Reconfigurable option is enabled, then the addresses become software
reconfigurable, which can be used with a soft reset to reboot and run alternative
code.The source and destination memory must be a memory mapped location
accessible by the core across the full transfer size.

MTVEC address - By default, the mtvec.BASE is set at Reset Vector Address + 0x04.
When the BootROM is enabled, the mtvec.BASE is set at destination address + 0x04.
When using Reconfigurable BootROM, the MTVEC register needs to be defined
and programmed through software.

Reset Behaviour - With the BootROM feature enabled, upon reset, the PC takes on
the value of the BootROM dest_addr. When the BootROM is enabled, ensure that the
boot code linker script matches the dest_addr, since booting starts from the
destination_addr.

BootROM Register Map:

| Name    | Address    | Description     | 
| -----|-----|-----|
| src_start_addr    | 0xA100    | Core copies data beginning here     | 
| src_end_addr    | 0xA104    | Last address copied by BootROM     | 
| destination_addr    | 0xA108    | Destination memory beginning from here    | 

</div>

<div id="Constants$SUBSYS_SOFT_REG_GRP_DED$description" data-type="text">

<a name="subsyssoftreggrpded"></a>
## SUBSYS Register Configuration
For the SUBSYS registers configutation, the following definitions are used in
the SUBSYS API functions. For example, to raise soft interrupts, enable parity
checks, soft reset, and so on.

| Configuration    | Value    | Description     | 
| -----|-----|-----|
| SUBSYS_SOFT_REG_GRP_DED    | 0x04    | Mask for the Core GPR DED Reset Register     | 
| SUBSYS_CFG_PARITY_CHECK    | 0x01    | Use to set or clear the parity check on the TCM     | 
| SUBSYS_SOFT_RESET    | 0x01    | Use the SUBSYS soft reset the MIV_RV32 IP core     | 
| SUBSYS_SOFT_IRQ    | 0x02    | Use to raise a software interrupt through SUBSYS    | 

</div>

<div id="Constants$SUBSYS_TCM_ECC_CE_IRQ$description" data-type="text">

<a name="subsystcmeccceirq"></a>
## SUBSYS Interrupt Request Masks
The following values correspond to the bit value of the SUBSYS interrupt enable
and interrupt pending register.

| Interrupt Mask    | Value    | Description     | 
| -----|-----|-----|
| SUBSYS_TCM_ECC_CE_IRQ    | 0x01u    | TCM ECC controllable error IRQ enable     | 
| SUBSYS_TCM_ECC_UCE_IRQ    | 0x02u    | TCM ECC uncontrollable error IRQ enable     | 
| SUBSYS_AXI_WR_RESP_IRQ    | 0x10u    | AXI write response error IRQ enable     | 
| SUBSYS_ICACHE_ECC_CE_IRQ    | 0x40u    | Icache ECC Correctable error IRQ     | 
| SUBSYS_ICACHE_ECC_UCE_IRQ    | 0x80u    | Icache ECC Uncorrectable error IRQ     | 
| SUBSYS_BASE_ADDR    | 0x6000u    | Base address of the SUBSYS    | 

</div>


# Functions

 ---------------- 
<a name="mrv32cleargpreccerrors"></a>
## MRV32_clear_gpr_ecc_errors
<a name="prototype"></a>
### Prototype 

<div id="Functions$MRV32_clear_gpr_ecc_errors$prototype" data-type="code">

    void
    MRV32_clear_gpr_ecc_errors
    (
        void 
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MRV32_clear_gpr_ecc_errors$description" data-type="text">

The MRV32_clear_gpr_ecc_errors() function clears single bit ECC errors on the
GPRs. The ECC block does not write back corrected data to memory. Hence, when
ECC is enabled for the GPRs and if that data has a single bit error then the
data coming out of the ECC block is corrected and will not have the error, but
the data source will still have the error. Therefore, if data has a single bit
error, then the corrected data must be written back to prevent the single bit
error from becoming a double bit error. Clear the pending interrupt bit after
this using MRV32_mgeci_clear_irq() function to complete the ECC error handling.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
<div id="Functions$MRV32_clear_gpr_ecc_errors$description$parameters$This" data-type="text" data-name="None">

This function takes no parameters.


</div>

<a name="return"></a>
### Return
<div id="Functions$MRV32_clear_gpr_ecc_errors$description$return" data-type="text">

This functions returns the CORE_GPR_DED_RESET_REG bit value.


</div>


 -------------------------------- 
<a name="mrv32mgeuiclearirq"></a>
## MRV32_mgeui_clear_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$MRV32_mgeui_clear_irq$prototype" data-type="code">

    void
    MRV32_mgeui_clear_irq
    (
        void 
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MRV32_mgeui_clear_irq$description" data-type="text">

The MRV32_mgeui_clear_irq() function clears the GPR ECC Uncorrectable Interrupt.
MGEUI interrupt is available only when ECC is enabled in the MIV_RV32 IP
configurator.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
<div id="" data-type="text" data-name="None">

This function takes no parameters.


</div>

<a name="return"></a>
### Return
<div id="Functions$MRV32_mgeui_clear_irq$description$return" data-type="text">

This function does not return any value.


</div>


 -------------------------------- 
<a name="mrv32mgeciclearirq"></a>
## MRV32_mgeci_clear_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$MRV32_mgeci_clear_irq$prototype" data-type="code">

    void
    MRV32_mgeci_clear_irq
    (
        void 
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MRV32_mgeci_clear_irq$description" data-type="text">

The MRV32_mgeci_clear_irq() function clears the GPR ECC Correctable Interrupt
MGECI interrupt is available only when ECC is enabled in the MIV_RV32 IP
configurator.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
<div id="" data-type="text" data-name="None">

This function takes no parameters.


</div>

<a name="return"></a>
### Return
<div id="Functions$MRV32_mgeci_clear_irq$description$return" data-type="text">

This function does not return any value.


</div>


 -------------------------------- 
<a name="mrvenablelocalirq"></a>
## MRV_enable_local_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$MRV_enable_local_irq$prototype" data-type="code">

    void
    MRV_enable_local_irq
    (
        uint32_t mask
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MRV_enable_local_irq$description" data-type="text">

The MRV_enable_local_irq() function enables the local interrupts. It takes a
mask value as input. For each set bit in the mask value, the corresponding
interrupt bit in the MIE register is enabled.

MRV_enable_local_irq( MRV32_SOFT_IRQn | MRV32_TIMER_IRQn | MRV32_EXT_IRQn |
MRV32_MSYS_EIE0_IRQn | MRV32_MSYS_SUBSYS_IRQn);

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### 
<div id="" data-type="text" data-name="">


</div>

<a name="return"></a>
### Return
<div id="" data-type="text">


</div>


 -------------------------------- 
<a name="mrvdisablelocalirq"></a>
## MRV_disable_local_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$MRV_disable_local_irq$prototype" data-type="code">

    void
    MRV_disable_local_irq
    (
        uint32_t mask
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MRV_disable_local_irq$description" data-type="text">

The MRV_disable_local_irq() function disables the local interrupts. It takes a
mask value as input. For each set bit in the mask value, the corresponding
interrupt bit in the MIE register is disabled.

MRV_disable_local_irq( MRV32_SOFT_IRQn | MRV32_TIMER_IRQn | MRV32_EXT_IRQn |
MRV32_MSYS_EIE0_IRQn | MRV32_MSYS_SUBSYS_IRQn);

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### 
<div id="" data-type="text" data-name="">


</div>

<a name="return"></a>
### Return
<div id="" data-type="text">


</div>


 -------------------------------- 
<a name="mrvenableinterrupts"></a>
## MRV_enable_interrupts
<a name="prototype"></a>
### Prototype 

<div id="Functions$MRV_enable_interrupts$prototype" data-type="code">

    void
    MRV_enable_interrupts
    (
        void 
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MRV_enable_interrupts$description" data-type="text">

The MRV_enable_interrupts() function enables all interrupts by setting the
machine mode interrupt enable bit in MSTATUS register.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
<div id="Functions$MRV_enable_interrupts$description$parameters$This" data-type="text" data-name="None">

This function takes no parameters.


</div>

<a name="return"></a>
### Return
<div id="Functions$MRV_enable_interrupts$description$return" data-type="text">

This functions returns the CORE_GPR_DED_RESET_REG bit value.


</div>


 -------------------------------- 
<a name="mrvdisableinterrupts"></a>
## MRV_disable_interrupts
<a name="prototype"></a>
### Prototype 

<div id="Functions$MRV_disable_interrupts$prototype" data-type="code">

    void
    MRV_disable_interrupts
    (
        void 
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MRV_disable_interrupts$description" data-type="text">

The MRV_disable_interrupts() function disables all interrupts by clearing the
machine mode interrupt enable bit in MSTATUS register.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
<div id="Functions$MRV_disable_interrupts$description$parameters$This" data-type="text" data-name="None">

This function takes no parameters.


</div>

<a name="return"></a>
### Return
<div id="Functions$MRV_disable_interrupts$description$return" data-type="text">

This functions returns the CORE_GPR_DED_RESET_REG bit value.


</div>


 -------------------------------- 
<a name="mrvreadmtvecbase"></a>
## MRV_read_mtvec_base
<a name="prototype"></a>
### Prototype 

<div id="Functions$MRV_read_mtvec_base$prototype" data-type="code">

    uint32_t
    MRV_read_mtvec_base
    (
        void 
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MRV_read_mtvec_base$description" data-type="text">

The MRV_read_mtvec_base() function reads the mtvec base value, which is the address
used when an interrupt/trap occurs. In the mtvec register, [31:2] is the BASE
address. NOTE: The BASE address must be aligned on a 4B boundary.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
<div id="Functions$MRV_read_mtvec_base$description$parameters$The" data-type="text" data-name="None">

This function takes no parameters.


</div>

<a name="return"></a>
### Return
<div id="Functions$MRV_read_mtvec_base$description$return" data-type="text">

The function returns the value of the BASE field [31:2] as an unsigned 32-bit
value.


</div>


 -------------------------------- 
<a name="mrvsetmtvecbase"></a>
## MRV_set_mtvec_base
<a name="prototype"></a>
### Prototype 

<div id="Functions$MRV_set_mtvec_base$prototype" data-type="code">

    void
    MRV_set_mtvec_base
    (
        uint32_t mtvec_base
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MRV_set_mtvec_base$description" data-type="text">

The MRV_set_mtvec_base() function takes the mtvec_base address as a unsigned int
and writes the value into the BASE field [31:2] in the mtvec CSR, MODE[1:0] is
Read-only. BASE is 4B aligned, so the lowest 2 bits of mtvec_base are ignored.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### mtvec_base
<div id="Functions$MRV_set_mtvec_base$description$parameters$mtvec_base" data-type="text" data-name="mtvec_base">

Any legal value is passed into the function, and it is used as the trap_entry
for interrupts. The PC jumps to this address provided when an interrupt occurs.
In case of vectored interrupts, the address value mentioned in the vector table
under the MIE Register Map is updated to the value passed to this function
parameter.


</div>

<a name="return"></a>
### Return
<div id="Functions$MRV_set_mtvec_base$description$return" data-type="text">

This function does not return any value.


</div>


 -------------------------------- 
<a name="mrvreadmtime"></a>
## MRV_read_mtime
<a name="prototype"></a>
### Prototype 

<div id="Functions$MRV_read_mtime$prototype" data-type="code">

    uint64_t
    MRV_read_mtime
    (
        void 
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MRV_read_mtime$description" data-type="text">

The MRV_read_mtime() function returns the current MTIME register value.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
<div id="" data-type="text" data-name="None">

This function takes no parameters.


</div>

<a name="return"></a>
### Return
<div id="" data-type="text">


</div>


 -------------------------------- 
<a name="mrvraisesoftirq"></a>
## MRV_raise_soft_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$MRV_raise_soft_irq$prototype" data-type="code">

    void
    MRV_raise_soft_irq
    (
        void 
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MRV_raise_soft_irq$description" data-type="text">

The MRV_raise_soft_irq() function raises a synchronous software interrupt by
writing into the MSIP register.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
<div id="Functions$MRV_raise_soft_irq$description$parameters$This" data-type="text" data-name="None">

This function takes no parameters.


</div>

<a name="return"></a>
### Return
<div id="Functions$MRV_raise_soft_irq$description$return" data-type="text">

This function does not return any value.


</div>


 -------------------------------- 
<a name="mrvclearsoftirq"></a>
## MRV_clear_soft_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$MRV_clear_soft_irq$prototype" data-type="code">

    void
    MRV_clear_soft_irq
    (
        void 
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MRV_clear_soft_irq$description" data-type="text">

The MRV_clear_soft_irq() function clears a synchronous software interrupt by
clearing the MSIP register.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
<div id="Functions$MRV_clear_soft_irq$description$parameters$This" data-type="text" data-name="None">

This function takes no parameters.


</div>

<a name="return"></a>
### Return
<div id="Functions$MRV_clear_soft_irq$description$return" data-type="text">

This function does not return any value.


</div>


 -------------------------------- 
<a name="systickhandler"></a>
## SysTick_Handler
<a name="prototype"></a>
### Prototype 

<div id="Functions$SysTick_Handler$prototype" data-type="code">

    void
    SysTick_Handler
    (
        void 
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$SysTick_Handler$description" data-type="text">

System tick handler. This handler function gets called when the Machine timer
interrupt asserts. An implementation of this function must be provided by the
application to implement the application specific machine timer interrupt
handling. If application does not provide such implementation, the weakly linked
handler stub function implemented in riscv_hal_stubs.c gets linked.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
<div id="" data-type="text" data-name="None">

This function takes no parameters.


</div>

<a name="return"></a>
### Return
<div id="" data-type="text">


</div>


 -------------------------------- 
<a name="mrvsystickconfig"></a>
## MRV_systick_config
<a name="prototype"></a>
### Prototype 

<div id="Functions$MRV_systick_config$prototype" data-type="code">

    uint32_t
    MRV_systick_config
    (
        uint64_t ticks
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MRV_systick_config$description" data-type="text">

System timer tick configuration. Configures the machine timer to generate a
system tick interrupt at regular intervals. Takes the number of system clock
ticks between interrupts.

Though this function can take any valid ticks value as parameter, we expect
that, for all practical purposes, a small tick value (to generate periodic
interrupts every few miliseconds) is passed. If you need to generate periodic
events in the range of seconds or more, you may use the SysTick_Handler() to
further count the number of interrupts and hence the larger time intervals.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### ticks
<div id="Functions$MRV_systick_config$description$parameters$ticks" data-type="text" data-name="ticks">

This is the number of ticks or clock cycles which are counted down from the
interrupt to be triggered.


</div>

<a name="return"></a>
### Return
<div id="Functions$MRV_systick_config$description$return" data-type="text">

Returns 0 if successful. Returns 1 if the interrupt interval is not achieved.


</div>


 -------------------------------- 
<a name="mrvbootromreconfigure"></a>
## MRV_BootROM_reconfigure
<a name="prototype"></a>
### Prototype 

<div id="Functions$MRV_BootROM_reconfigure$prototype" data-type="code">

    void
    MRV_BootROM_reconfigure
    (
        uint32_t start_addr,
        uint32_t end_addr,
        uint32_t destination_addr
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MRV_BootROM_reconfigure$description" data-type="text">

BootROM Address Reconfiguration

Configures the BootROM registers with the source start, end, and destination
addresses for the core when BootROM addresses are reconfigurable.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### start_addr
<div id="Functions$MRV_BootROM_reconfigure$description$parameters$start_addr" data-type="text" data-name="start_addr">

Starting address for the BootROM copy.


</div>

#### end_addr
<div id="Functions$MRV_BootROM_reconfigure$description$parameters$end_addr" data-type="text" data-name="end_addr">

End address for the BootROM copy.


</div>

#### destination_addr
<div id="Functions$MRV_BootROM_reconfigure$description$parameters$destination_addr" data-type="text" data-name="destination_addr">

On Reset, the core will start running code from this memory address.


</div>

<a name="return"></a>
### Return
<div id="" data-type="text">


</div>


 -------------------------------- 
<a name="mrv32subsysenableirq"></a>
## MRV32_subsys_enable_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$MRV32_subsys_enable_irq$prototype" data-type="code">

    void
    MRV32_subsys_enable_irq
    (
        uint32_t irq_mask
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MRV32_subsys_enable_irq$description" data-type="text">

The MRV32_subsys_enable_irq() function initializes the SUBSYS interrupts. It
takes the logical OR of the following defined IRQ masks as a parameter.

| irq_mask    | Value     | 
| -----|-----|
| SUBSYS_TCM_ECC_CE_IRQ    | 0x01u     | 
| SUBSYS_TCM_ECC_UCE_IRQ    | 0x02u     | 
| SUBSYS_AXI_WR_RESP_IRQ    | 0x10u     | 
| SUBSYS_ICACHE_ECC_CE_IRQ    | 0x40u     | 
| SUBSYS_ICACHE_ECC_UCE_IRQ    | 0x80u    | 

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### irq_mask
<div id="Functions$MRV32_subsys_enable_irq$description$parameters$irq_mask" data-type="text" data-name="irq_mask">

| irq_mask    | Value     | 
| -----|-----|
| SUBSYS_TCM_ECC_CE_IRQ    | 0x01u     | 
| SUBSYS_TCM_ECC_UCE_IRQ    | 0x02u     | 
| SUBSYS_AXI_WR_RESP_IRQ    | 0x10u     | 
| SUBSYS_ICACHE_ECC_CE_IRQ    | 0x40u     | 
| SUBSYS_ICACHE_ECC_UCE_IRQ    | 0x80u    | 


</div>

<a name="return"></a>
### Return
<div id="Functions$MRV32_subsys_enable_irq$description$return" data-type="text">

This function does not return any value.


</div>


 -------------------------------- 
<a name="mrv32subsysdisableirq"></a>
## MRV32_subsys_disable_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$MRV32_subsys_disable_irq$prototype" data-type="code">

    void
    MRV32_subsys_disable_irq
    (
        uint32_t irq_mask
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MRV32_subsys_disable_irq$description" data-type="text">

The MRV32_subsys_disable_irq() function disables the SUBSYS interrupts. It takes
the logical OR of the following defined IRQ masks as a parameter.

| irq_mask    | Value     | 
| -----|-----|
| SUBSYS_TCM_ECC_CE_IRQ    | 0x01u     | 
| SUBSYS_TCM_ECC_UCE_IRQ    | 0x02u     | 
| SUBSYS_AXI_WR_RESP_IRQ    | 0x10u     | 
| SUBSYS_ICACHE_ECC_CE_IRQ    | 0x40u     | 
| SUBSYS_ICACHE_ECC_UCE_IRQ    | 0x80u    | 

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### irq_mask
<div id="Functions$MRV32_subsys_disable_irq$description$parameters$irq_mask" data-type="text" data-name="irq_mask">

| irq_mask    | Value     | 
| -----|-----|
| SUBSYS_TCM_ECC_CE_IRQ    | 0x01u     | 
| SUBSYS_TCM_ECC_UCE_IRQ    | 0x02u     | 
| SUBSYS_AXI_WR_RESP_IRQ    | 0x10u     | 
| SUBSYS_ICACHE_ECC_CE_IRQ    | 0x40u     | 
| SUBSYS_ICACHE_ECC_UCE_IRQ    | 0x80u    | 


</div>

<a name="return"></a>
### Return
<div id="Functions$MRV32_subsys_disable_irq$description$return" data-type="text">

This function does not return any value.


</div>


 -------------------------------- 
<a name="mrv32subsysclearirq"></a>
## MRV32_subsys_clear_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$MRV32_subsys_clear_irq$prototype" data-type="code">

    void
    MRV32_subsys_clear_irq
    (
        uint32_t irq_mask
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MRV32_subsys_clear_irq$description" data-type="text">

The MRV32_subsys_clear_irq() function clears the SUBSYS interrupts, which was
triggered. It takes the logical OR of the following defined IRQ masks as a
parameter.

| irq_mask    | Value     | 
| -----|-----|
| SUBSYS_TCM_ECC_CE_IRQ    | 0x01u     | 
| SUBSYS_TCM_ECC_UCE_IRQ    | 0x02u     | 
| SUBSYS_AXI_WR_RESP_IRQ    | 0x10u     | 
| SUBSYS_ICACHE_ECC_CE_IRQ    | 0x40u     | 
| SUBSYS_ICACHE_ECC_UCE_IRQ    | 0x80u    | 

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### irq_mask
<div id="Functions$MRV32_subsys_clear_irq$description$parameters$irq_mask" data-type="text" data-name="irq_mask">

| irq_mask    | Value     | 
| -----|-----|
| SUBSYS_TCM_ECC_CE_IRQ    | 0x01u     | 
| SUBSYS_TCM_ECC_UCE_IRQ    | 0x02u     | 
| SUBSYS_AXI_WR_RESP_IRQ    | 0x10u     | 
| SUBSYS_ICACHE_ECC_CE_IRQ    | 0x40u     | 
| SUBSYS_ICACHE_ECC_UCE_IRQ    | 0x80u    | 


</div>

<a name="return"></a>
### Return
<div id="Functions$MRV32_subsys_clear_irq$description$return" data-type="text">

This function does not return any value.


</div>


 -------------------------------- 
<a name="mrv32subsysirqcause"></a>
## MRV32_subsys_irq_cause
<a name="prototype"></a>
### Prototype 

<div id="Functions$MRV32_subsys_irq_cause$prototype" data-type="code">

    uint32_t
    MRV32_subsys_irq_cause
    (
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MRV32_subsys_irq_cause$description" data-type="text">

The MRV32_subsys_irq_cause() function returns the irq_pend register value which
is present in the SUBSYS. This is be used to check which irq_mask value caused
the SUBSYS interrupt to occur.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### This
<div id="Functions$MRV32_subsys_irq_cause$description$parameters$This" data-type="text" data-name="This">

function does not take any parameters


</div>

<a name="return"></a>
### Return
<div id="Functions$MRV32_subsys_irq_cause$description$return" data-type="text">

This function returns the irq_pend regsiter value.


</div>


 -------------------------------- 
<a name="mrv32isgprded"></a>
## MRV32_is_gpr_ded
<a name="prototype"></a>
### Prototype 

<div id="Functions$MRV32_is_gpr_ded$prototype" data-type="code">

    uint32_t
    MRV32_is_gpr_ded
    (
        void 
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MRV32_is_gpr_ded$description" data-type="text">

The MRV32_is_gpr_ded() function returns the core_gpr_ded_reset_reg bit value.
When ECC is enabled, the core_gpr_ded_reset_reg is set when the core was reset
due to GPR DED error.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
<div id="Functions$MRV32_is_gpr_ded$description$parameters$This" data-type="text" data-name="None">

This function takes no parameters.


</div>

<a name="return"></a>
### Return
<div id="Functions$MRV32_is_gpr_ded$description$return" data-type="text">

This functions returns the CORE_GPR_DED_RESET_REG bit value.


</div>


 -------------------------------- 
<a name="mrv32cleargprded"></a>
## MRV32_clear_gpr_ded
<a name="prototype"></a>
### Prototype 

<div id="Functions$MRV32_clear_gpr_ded$prototype" data-type="code">

    void
    MRV32_clear_gpr_ded
    (
        void 
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MRV32_clear_gpr_ded$description" data-type="text">

The MRV32_clear_gpr_ded() function must be used to clear the
core_gpr_ded_reset_reg bit. When ECC is enabled, the core_gpr_ded_reset_reg is
set when the core was previously reset due to GPR DED error.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
<div id="Functions$MRV32_clear_gpr_ded$description$parameters$This" data-type="text" data-name="None">

This function takes no parameters.


</div>

<a name="return"></a>
### Return
<div id="Functions$MRV32_clear_gpr_ded$description$return" data-type="text">

This function does not return any value.


</div>


 -------------------------------- 
<a name="mrv32enableparitycheck"></a>
## MRV32_enable_parity_check
<a name="prototype"></a>
### Prototype 

<div id="Functions$MRV32_enable_parity_check$prototype" data-type="code">

    void
    MRV32_enable_parity_check
    (
        void 
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MRV32_enable_parity_check$description" data-type="text">

The MRV32_enable_parity_check() function is used to enable parity check on the
TCM and it's interface transactions. This feature is not available on MIV_RV32
v3.1 and MIV_RV32 v3.0.100

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
<div id="Functions$MRV32_enable_parity_check$description$parameters$This" data-type="text" data-name="None">

This function takes no parameters.


</div>

<a name="return"></a>
### Return
<div id="Functions$MRV32_enable_parity_check$description$return" data-type="text">

This function does not return any value.


</div>


 -------------------------------- 
<a name="mrv32disableparitycheck"></a>
## MRV32_disable_parity_check
<a name="prototype"></a>
### Prototype 

<div id="Functions$MRV32_disable_parity_check$prototype" data-type="code">

    void
    MRV32_disable_parity_check
    (
        void 
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MRV32_disable_parity_check$description" data-type="text">

The MRV32_disable_parity_check() function is used to disable parity check on the
TCM and it's interface transactions.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
<div id="Functions$MRV32_disable_parity_check$description$parameters$This" data-type="text" data-name="None">

This function takes no parameters.


</div>

<a name="return"></a>
### Return
<div id="Functions$MRV32_disable_parity_check$description$return" data-type="text">

This function does not return any value.


</div>


 -------------------------------- 
<a name="mrv32cpusoftreset"></a>
## MRV32_cpu_soft_reset
<a name="prototype"></a>
### Prototype 

<div id="Functions$MRV32_cpu_soft_reset$prototype" data-type="code">

    void
    MRV32_cpu_soft_reset
    (
        void 
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MRV32_cpu_soft_reset$description" data-type="text">

The MRV32_cpu_soft_reset() function is used to cause a soft cpu reset on the
MIV_RV32 soft processor core.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
<div id="Functions$MRV32_cpu_soft_reset$description$parameters$This" data-type="text" data-name="None">

This function takes no parameters.


</div>

<a name="return"></a>
### Return
<div id="Functions$MRV32_cpu_soft_reset$description$return" data-type="text">

This function does not return any value.


</div>


 -------------------------------- 
