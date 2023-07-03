<html>
 
 ------------------------------------ 

# Mi-V uDMA Bare Metal Driver.
 
-----------------------------------------
## Table of Contents 
  - [Introduction](#introduction)

  - [Hardware Flow Dependency](#hardware-flow-dependency)

  - [Theory of Operation](#theory-of-operation)

- [Types](#types)
  - [miv_udma_instance_t](#mivudmainstancet)

- [Constants](#constants)
  - [MIV_uDMA_CTRL_IRQ_CONFIG](#mivudmactrlirqconfig)
  - [MIV_uDMA_STATUS_BUSY](#mivudmastatusbusy)
  - [MIV_uDMA_STATUS_ERROR](#mivudmastatuserror)

- [Functions](#functions)
  - [MIV_uDMA_init](#mivudmainit)
  - [MIV_uDMA_config](#mivudmaconfig)
  - [MIV_uDMA_start](#mivudmastart)
  - [MIV_uDMA_reset](#mivudmareset)
  - [MIV_uDMA_read_status](#mivudmareadstatus)

<div id="TitlePage" data-type="text">

# Introduction
The Mi-V uDMA driver provides a set of functions to control the Mi-V uDMA module
in the Mi-V Extended Subsystem (MIV_ESS) soft-IP. The Mi-V uDMA module allows
peripherals with AHB interfaces to transfer data independently of the MIV_RV32
RISC-V processor.

Following are the major features provided by the Mi-V uDMA driver:

  - Initialization and configuration
  - Start and reset the transaction

This driver can be used as part of a bare metal system where no operating system
is available. The driver can be adapted for use as part of an operating system,
but the implementation of the adaptation layer between the driver and the
operating system's driver model is outside the scope of this driver.

# Hardware Flow Dependency
The application software should initialize and configure the Mi-V uDMA through
calling the MIV_uDMA_init() and MIV_uDMA_config() functions for each Mi-V uDMA
instance in the design.

The uDMA can operate in two possible transfer configurations:

  - AHBL Read -> AHBL Write:  
In this configuration, the uDMA reads data from 
the source memory over an AHBL (mirrored main/initiator) read interface and 
writes data to the destination memory over an AHBL (mirrored main/initiator) 
write interface.
  - AHBL Read -> TAS Write:  
In this configuration, the uDMA reads data from 
the source memory over an AHBL (mirrored main/initiator) read interface and 
writes data to the destination memory over the TAS (mirrored main/initiator) 
write interface.

Note: The AHBL Read -> TAS Write configuration is out of scope for this driver.

# Theory of Operation
The uDMA module in the Mi-V Extended Sub System (MIV_ESS) is a single-channel
uDMA module that allows peripherals to perform read-write operations between
source and destination memory. The Mi-V uDMA driver is used in interrupt-driven
mode and uses the Mi-V uDMA IRQ signal to drive the interrupt service routine
(ISR), which signifies a transfer has completed. The status is checked in the
ISR to ensure the transfer is completed successfully.  
The reset operation in
the ISR resets the Mi-V uDMA controller. Once the Mi-V uDMA transfer is
complete, Mi-V uDMA retires. To initiate another transaction, Mi-V uDMA needs to
be configured again.

The operation of the Mi-V uDMA driver is divided into the following categories:

  - Initialization
  - Configuration
  - Start and reset the transfer

Initialization and configuration:  
Mi-V uDMA is first initialized by calling
MIV_uDMA_init() function. This function initializes the instance of Mi-V uDMA
with the base address. The MIV_uDMA_init() function must be called before
calling any other Mi-V uDMA driver functions.

The Mi-V uDMA is configured by calling MIV_uDMA_config() function. This function
configures the source_addr and dest_addr registers of the Mi-V uDMA with source
and destination addresses for Mi-V uDMA transfers. This function also configures
the transfer size and interrupt preference for successful transfers using Mi-V
uDMA.

Start and reset the transfer:  
Once the Mi-V uDMA is configured, initiate the
transfers by calling the MIV_uDMA_start() function. Once the Mi-V uDMA transfer
is started, it cannot be aborted, and the status of the transfer should be read
from the ISR by calling the MIV_uDMA_read_status() function.

Reset the Mi-V uDMA to the default state by calling the MIV_uDMA_reset()
function. After performing the reset operation, reconfigure the Mi-V uDMA to
perform transfers as MIV_uDMA_reset() resets the Mi-V uDMA controller.

</div>


# Types

 ---------------- 
<a name="mivudmainstancet"></a>
## miv_udma_instance_t
### Prototype 

<div id="Types$miv_udma_instance_t$prototype" data-type="code">

``` 
    typedef struct miv_udma_instance {
        addr_t base_address; 
    } miv_udma_instance_t ;
  ``` 


</div>

### Description 

<div id="Types$miv_udma_instance_t$description" data-type="text">

This structure holds the base of the Mi-V uDMA module, which is used in the
other functions of the driver to access the uDMA registers.


</div>


 --------------------------- 

# Constants

 ---------------- 
<div id="Constants$MIV_uDMA_CTRL_IRQ_CONFIG$description" data-type="text">

<a name="mivudmactrlirqconfig"></a>
## MIV_uDMA_CTRL_IRQ_CONFIG
The MIV_uDMA_CTRL_IRQ_CONFIG macro is used to assert the uDMA IRQ when an error
occurs during a uDMA transfer or on the completion of a uDMA transfer.

</div>

<div id="Constants$MIV_uDMA_STATUS_BUSY$description" data-type="text">

<a name="mivudmastatusbusy"></a>
## MIV_uDMA_STATUS_BUSY
The MIV_uDMA_STATUS_BUSY macro is used to indicate that the uDMA transfer is in
progress.

</div>

<div id="Constants$MIV_uDMA_STATUS_ERROR$description" data-type="text">

<a name="mivudmastatuserror"></a>
## MIV_uDMA_STATUS_ERROR
The MIV_uDMA_STATUS_ERROR macro is used to indicate that the last uDMA transfer
has caused an error.

</div>


# Functions

 ---------------- 
<a name="mivudmainit"></a>
## MIV_uDMA_init
### Prototype 

<div id="Functions$MIV_uDMA_init$prototype" data-type="code">

    void
    MIV_uDMA_init
    (
        miv_udma_instance_t * this_udma,
        addr_t base_addr
    );


</div>

### Description

<div id="Functions$MIV_uDMA_init$description" data-type="text">

The MIV_uDMA_init() function assigns the base address of the Mi-V uDMA module to
the uDMA instance structure.  
This address is used in a later part of the
driver to access the uDMA registers.

</div>


 --------------------------- 

### Parameters
#### this_udma
<div id="Functions$MIV_uDMA_init$description$parameters$this_udma" data-type="text" data-name="this_udma">

This parameter is a pointer to the miv_udma_instance_t structure.


</div>

#### base_addr
<div id="Functions$MIV_uDMA_init$description$parameters$base_addr" data-type="text" data-name="base_addr">

Base address of the Mi-V uDMA module.


</div>

### Return
<div id="Functions$MIV_uDMA_init$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="mivudmaconfig"></a>
## MIV_uDMA_config
### Prototype 

<div id="Functions$MIV_uDMA_config$prototype" data-type="code">

    void
    MIV_uDMA_config
    (
        miv_udma_instance_t * this_udma,
        addr_t src_addr,
        addr_t dest_addr,
        uint32_t transfer_size,
        uint32_t irq_config
    );


</div>

### Description

<div id="Functions$MIV_uDMA_config$description" data-type="text">

The MIV_uDMA_config() function is used to configure the Mi-V uDMA controller.
This function will set the source address, destination address, block size, and
IRQ configuration register.

</div>


 --------------------------- 

### Parameters
#### this_udma
<div id="Functions$MIV_uDMA_config$description$parameters$this_udma" data-type="text" data-name="this_udma">

This parameter is a pointer to the miv_udma_instance_t structure, which holds
the base address of the Mi-V uDMA module.


</div>

#### base_addr
<div id="Functions$MIV_uDMA_config$description$parameters$base_addr" data-type="text" data-name="base_addr">

Base address of the Mi-V uDMA.


</div>

#### src_addr
<div id="Functions$MIV_uDMA_config$description$parameters$src_addr" data-type="text" data-name="src_addr">

Source address of memory from where the uDMA reads the data.


</div>

#### dest_addr
<div id="Functions$MIV_uDMA_config$description$parameters$dest_addr" data-type="text" data-name="dest_addr">

Destination address where the data is written from src_addr.


</div>

#### transfer_size
<div id="Functions$MIV_uDMA_config$description$parameters$transfer_size" data-type="text" data-name="transfer_size">

Number of 32-bit words to transfer.


</div>

#### irq_config
<div id="Functions$MIV_uDMA_config$description$parameters$irq_config" data-type="text" data-name="irq_config">

uDMA IRQ configuration

  - When set, the IRQ is asserted when an error occurs during a uDMA transfer or
 on the completion of the uDMA transfer.  


  - When clear, the IRQ is only asserted when an error occurs during a uDMA 
transfer.


</div>

### Return
<div id="Functions$MIV_uDMA_config$description$return" data-type="text">

This function does not return any value.


</div>


 -------------------------------- 
<a name="mivudmastart"></a>
## MIV_uDMA_start
### Prototype 

<div id="Functions$MIV_uDMA_start$prototype" data-type="code">

    void
    MIV_uDMA_start
    (
        miv_udma_instance_t * this_udma
    );


</div>

### Description

<div id="Functions$MIV_uDMA_start$description" data-type="text">

The MIV_uDMA_start() function is used to start the uDMA transfer.

</div>


 --------------------------- 

### Parameters
#### this_udma
<div id="Functions$MIV_uDMA_start$description$parameters$this_udma" data-type="text" data-name="this_udma">

This parameter is a pointer to the miv_udma_instance_t structure, which holds
the base address of the Mi-V uDMA module.


</div>

### Return
<div id="Functions$MIV_uDMA_start$description$return" data-type="text">

This function does not return any value.


</div>


 -------------------------------- 
<a name="mivudmareset"></a>
## MIV_uDMA_reset
### Prototype 

<div id="Functions$MIV_uDMA_reset$prototype" data-type="code">

    void
    MIV_uDMA_reset
    (
        miv_udma_instance_t * this_udma
    );


</div>

### Description

<div id="Functions$MIV_uDMA_reset$description" data-type="text">

The MIV_uDMA_reset() function is used to clear the uDMA interrupt and reset the
uDMA transfer.

This function should be called from the interrupt handler to reset the values
set during MIV_uDMA_config().

</div>


 --------------------------- 

### Parameters
#### this_udma
<div id="Functions$MIV_uDMA_reset$description$parameters$this_udma" data-type="text" data-name="this_udma">

This parameter is a pointer to the miv_udma_instance_t structure, which holds
the base address of the Mi-V uDMA module.


</div>

### Return
<div id="Functions$MIV_uDMA_reset$description$return" data-type="text">

This function does not return any value.


</div>


 -------------------------------- 
<a name="mivudmareadstatus"></a>
## MIV_uDMA_read_status
### Prototype 

<div id="Functions$MIV_uDMA_read_status$prototype" data-type="code">

    uint32_t
    MIV_uDMA_read_status
    (
        miv_udma_instance_t * this_pdma
    );


</div>

### Description

<div id="Functions$MIV_uDMA_read_status$description" data-type="text">

The MIV_uDMA_read_status() function is used to read the status of the uDMA
transfer. When interrupt is enabled, this function can be called from the
interrupt handler to know the reason for a uDMA interrupt.

| Bit Number    | Name    | Description     | 
| -----|-----|-----|
| 0    | Busy    | When set indicates that uDMA transfer is in progress     | 
| 1    | Error    | When set indicates that last uDMA transfer caused an     | 
|   |   | error.    | 

</div>


 --------------------------- 

### Parameters
#### this_udma
<div id="Functions$MIV_uDMA_read_status$description$parameters$this_udma" data-type="text" data-name="this_udma">

This parameter is a pointer to the miv_udma_instance_t structure, which holds
the base address of the Mi-V uDMA module.


</div>

### Return
<div id="Functions$MIV_uDMA_read_status$description$return" data-type="text">

The return value indicates an error due to the busy status of the uDMA channel.


</div>


 -------------------------------- 

</html>
