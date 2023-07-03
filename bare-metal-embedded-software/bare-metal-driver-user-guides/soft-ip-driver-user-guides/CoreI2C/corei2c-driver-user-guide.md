<html>
 
 ------------------------------------ 

# CoreI2C Bare Metal Driver.
 
-----------------------------------------
## Table of Contents 
  - [Introduction](#introduction)

  - [Hardware Flow Dependencies](#hardware-flow-dependencies)

     - [Interrupt Control](#interrupt-control)

     - [SMBus Logic Options](#smbus-logic-options)

     - [Fixed Baud Rate Values](#fixed-baud-rate-values)

     - [Fixed Slave Address Options Values](#fixed-slave-address-options-values)

  - [Theory of Operation](#theory-of-operation)

     - [Initialization and Configuration](#initialization-and-configuration)

     - [Interrupt Control](#interrupt-control)

     - [I2C Slave Addressing Functions](#i2c-slave-addressing-functions)

     - [Transaction Types](#transaction-types)

     - [Write Transaction](#write-transaction)

     - [Read Transaction](#read-transaction)

     - [Write-Read Transaction](#write-read-transaction)

     - [Master Operations](#master-operations)

     - [Slave Operations](#slave-operations)

     - [Responding to Read Transactions](#responding-to-read-transactions)

     - [Responding to Write Transactions](#responding-to-write-transactions)

     - [Responding to Write-Read Transactions](#responding-to-write-read-transactions)

     - [Mixed Master-Slave Operations](#mixed-master-slave-operations)

     - [SMBus Control](#smbus-control)

- [Types](#types)
  - [i2c_channel_number_t](#i2cchannelnumbert)
  - [i2c_clock_divider_t](#i2cclockdividert)
  - [i2c_status_t](#i2cstatust)
  - [i2c_slave_handler_ret_t](#i2cslavehandlerrett)
  - [i2c_instance_t](#i2cinstancet)
  - [i2c_slave_wr_handler_t](#i2cslavewrhandlert)

- [Constants](#constants)
  - [I2C_RELEASE_BUS](#i2creleasebus)
  - [I2C_HOLD_BUS](#i2choldbus)
  - [Interrupt Identifier Number](#interrupt-identifier-number)
  - [I2C_NO_TIMEOUT](#i2cnotimeout)

- [Functions](#functions)
  - [I2C_init](#i2cinit)
  - [I2C_channel_init](#i2cchannelinit)
  - [I2C_isr](#i2cisr)
  - [I2C_write](#i2cwrite)
  - [I2C_read](#i2cread)
  - [I2C_write_read](#i2cwriteread)
  - [I2C_get_status](#i2cgetstatus)
  - [I2C_wait_complete](#i2cwaitcomplete)
  - [I2C_system_tick](#i2csystemtick)
  - [I2C_set_slave_tx_buffer](#i2csetslavetxbuffer)
  - [I2C_set_slave_rx_buffer](#i2csetslaverxbuffer)
  - [I2C_set_slave_mem_offset_length](#i2csetslavememoffsetlength)
  - [I2C_register_write_handler](#i2cregisterwritehandler)
  - [I2C_enable_slave](#i2cenableslave)
  - [I2C_disable_slave](#i2cdisableslave)
  - [I2C_set_slave_second_addr](#i2csetslavesecondaddr)
  - [I2C_disable_slave_second_addr](#i2cdisableslavesecondaddr)
  - [I2C_set_gca](#i2csetgca)
  - [I2C_clear_gca](#i2ccleargca)
  - [I2C_smbus_init](#i2csmbusinit)
  - [I2C_enable_smbus_irq](#i2cenablesmbusirq)
  - [I2C_disable_smbus_irq](#i2cdisablesmbusirq)
  - [I2C_suspend_smbus_slave](#i2csuspendsmbusslave)
  - [I2C_resume_smbus_slave](#i2cresumesmbusslave)
  - [I2C_reset_smbus](#i2cresetsmbus)
  - [I2C_set_smbus_alert](#i2csetsmbusalert)
  - [I2C_clear_smbus_alert](#i2cclearsmbusalert)
  - [I2C_get_irq_status](#i2cgetirqstatus)
  - [I2C_set_user_data](#i2csetuserdata)
  - [I2C_get_user_data](#i2cgetuserdata)

<div id="TitlePage" data-type="text">

# Introduction
The CoreI2C driver provides a set of functions for controlling the Microchip
CoreI2C hardware IP. The driver supports up to 16 separate I2C channels per
CoreI2C instance, with common slave address settings shared between channels on
a device.

Optional features of the CoreI2C allow it to operate with I2C based protocols
such as System Management Bus (SMBus), Power Management Bus (PMBus), and
Intelligent Platform Management Interface (IPMI). This driver provides support
for these features when enabled in the CoreI2C IP.

The major features provided by CoreI2C driver:

  - Provides support to configuring the I2C channels of each CoreI2C peripheral 
device.  


  - I2C master operations.  


  - I2C slave operations.  


  - SMBus related operations.  



This driver is used as part of a bare metal system where no operating system is
available. The driver gets adapted as a part of an operating system, but the
implementation of the adaptation layer between the driver and the operating
system's driver model is outside the scope of this driver.

# Hardware Flow Dependencies
Your application software should configure the CoreI2C driver through calls to
the I2C_init() function for each CoreI2C instance in the hardware design. The
configuration parameters include the CoreI2C hardware instance base address and
other runtime parameters, such as the I2C serial clock frequency and the I2C
device address.

Once channel 0 of a CoreI2C peripheral has been initialized by I2C_init(), any
additional channels present should be configured by calling I2C_channel_init()
for each of the remaining channels.

Apart from the CoreI2C hardware instance base address, no CoreI2C hardware
configuration parameters are used by the driver. Hence, no additional
configuration files are required to use the driver.

## Interrupt Control
The CoreI2C driver has to enable and disable the generation of interrupts by
CoreI2C at various times when it is operating. This enabling and disabling of
interrupts must be done through the system’s interrupt controller. For that
reason, the method of controlling the CoreI2C interrupt is system specific and
it is necessary to customize the I2C_enable_irq() and I2C_disable_irq()
functions. These functions are available in the i2c_interrupt.c file. The
default implementation calls HAL_ASSERT(0) to indicate to the application
developer that a suitable implementations for these functions must be provided.

The implementation of the I2C_enable_irq() function should permit interrupts
generated by a CoreI2C instance to interrupt the processor. The implementation
of the I2C_disable_irq() function should prevent interrupts generated by a
CoreI2C instance from interrupting the processor. See the provided example
projects for a working implementation of these functions.

The I2C_register_write_handler() function registers a write handler function
with the CoreI2C driver that calls on completion of an I2C write transaction by
the CoreI2C slave. It is your responsibility to create and register the
implementation of this handler function that processes or trigger the processing
of the received data.

The SMBSUS and SMBALERT interrupts are related to the SMBus interface and are
enabled and disabled through I2C_enable_smbus_irq() and I2C_disable_smbus_irq()
respectively. It is your responsibility to create interrupt handler functions in
your application to get the desired response for the SMBus interrupts.

Note: You must include the path to any application header files that are
included in the i2c_interrupt.c file, as an include path in your project's
compiler settings. The details of how to do this will depend on your development
software.

## SMBus Logic Options
SMBus related APIs does not have any effect if the "Generate SMBus Logic" is not
enabled in the CoreI2C hardware configuration. Following are API's that does not
give the desired results if SMBus Logic is disabled.

  - I2C_smbus_init()
  - I2C_reset_smbus()
  - I2C_enable_smbus_irq()
  - I2C_disable_smbus_irq()
  - I2C_suspend_smbus_slave()
  - I2C_resume_smbus_slave()
  - I2C_set_smsbus_alert()
  - I2C_clear_smsbus_alert()
  - I2C_get_irq_status()

## Fixed Baud Rate Values
The serial clock frequency parameter passed to the I2C_init() and
I2C_channel_init() functions may not have any effect if fixed values were
selected for Baud rate in the hardware configuration of CoreI2C. When fixed
values are selected for these baud rates, the driver cannot overwrite the fixed
values.

## Fixed Slave Address Options Values
The primary slave address parameter passed to the I2C_init() function and
secondary address value passed to the I2C_set_slave_second_addr() function, may
not have the desired effect if fixed values were selected for the slave 0
address and slave 1 address respectively. Proper operation of this version of
the driver requires the slave addresses to be programmable.

# Theory of Operation
The CoreI2C software driver is designed to allow the control of multiple
instances of CoreI2C with one or more I2C channels. Each channel in an instance
of CoreI2C in the hardware design is associated with a single instance of the
i2c_instance_t structure in the software. You must allocate memory for one
unique i2c_instance_t structure instance for each channel of each CoreI2C
hardware instance. The contents of these data structures are initialised by
calling I2C_init() and if necessary I2C_channel_init(). A pointer to the
structure is passed to the subsequent driver functions in order to identify the
CoreI2C hardware instance and channel to perform the requested operation.

Note: Do not attempt to directly manipulate the contents of i2c_instance_t
structures. These structures are only intended to be modified by the driver
functions.

The CoreI2C driver functions are grouped into the following categories:

  - Initialization and configuration functions  


  - Interrupt control  


  - I2C slave addressing functions  


  - I2C master operations functions to handle write, read, and write-read 
transactions  


  - I2C slave operations functions to handle write, read, and write-read 
transactions  


  - Mixed master-slave operations  


  - SMBus interface configuration and control  



## Initialization and Configuration
The CoreI2C device is first initialized by calling the I2C_init() function.
Since each CoreI2C peripheral supports up to 16 channels, an additional
function, I2C_channel_init(), is required to initialize the remaining channels
with their own data structures.

I2C_init() function initializes channel 0 of a CoreI2C and the i2c_instance_t
for channel 0 acts as the basis for further channel initialization as the
hardware base address and I2C serial address are same across all the channels.
Ensure to call I2C_init() function before calling any other I2C driver function
calls. The I2C_init() call for each CoreI2C takes the I2C serial address
assigned to the I2C and the serial clock divider to generate its I2C clock as
configuration parameters.

I2C_channel_init() function takes as input parameters a pointer to the CoreI2C
i2c_instance_t which has been initialized by calling the I2C_init() and a
pointer to a separate i2c_instance_t which represents this new channel. Another
input parameter which is required by this function is serial clock divider which
generates its I2C clock.

## Interrupt Control
The CoreI2C driver is interrupt driven and it uses each channels INT interrupt
to drive the state machine which is at the heart of the driver. The application
is responsible for providing the link between the interrupt generating hardware
and the CoreI2C interrupt handler and must ensure that the I2C_isr() function is
called with the correct i2c_instance_t structure pointer for the CoreI2C channel
initiating the interrupt.

The driver enables and disables the generation of INT interrupts by CoreI2C at
various times when it is operating through the user supplied I2C_enable_irq()
and I2C_disable_irq() functions.

The I2C_register_write_handler() function is used to register a write handler
function with the CoreI2C driver which is called on completion of an I2C write
transaction by the CoreI2C slave. It is the user applications responsibility to
create and register the implementation of this handler function that processes
or triggers the processing of the received data.

The other two interrupt sources in the CoreI2C are related to SMBus operation
and are enabled and disabled through I2C_enable_smbus_irq() and
I2C_disable_smbus_irq() respectively. Due to the application specific nature of
the response to SMBus interrupts, you must design interrupt handler functions in
the application to get the desired behaviour for SMBus related interrupts.

If enabled, the SMBA_INT signal from the CoreI2C is asserted if an SMBALERT
condition is signalled on the SMBALERT_NI input for the channel.

If enabled, the SMBS_INT signal from the CoreI2C is asserted if an SMBSUSPEND
condition is signalled on the SMBSUS_NI input for the channel.

## I2C Slave Addressing Functions
A CoreI2C peripheral responds to the following three slave addresses:

  - Slave address 0 - This is the primary slave address that accesses a CoreI2C 
channel when it acts as a slave in I2C transactions. You must configure the 
primary slave address using I2C_init().
  - Slave address 1 - This is the secondary slave address which might be 
required in certain application specific scenarios. The secondary slave address 
is configured by I2C_set_slave_second_addr() and is disabled by 
I2C_disable_slave_second_addr().
  - General call address - A CoreI2C slave can be configured to respond to a 
broadcast command by a master transmitting the general call address of 0x00. Use
 the I2C_set_gca() function to enable the slave to respond to the general call 
address. If the CoreI2C slave is not required to respond to the general call 
address, disable this address by calling I2C_clear_gca().

Note: All channels on a CoreI2C instance share the same slave address logic.
This means that they cannot have separate slave addresses and rely on the
separate physical I2C bus connections to distinguish them.

## Transaction Types
The I2C driver is designed to handle three types of I2C transaction: 
- Write
transactions 
- Read transactions 
- Write-read transactions

## Write Transaction
The master I2C device initiates a write transaction by sending a START bit as
soon as the bus becomes free. The START bit is followed by the 7-bit serial
address of the target slave device followed by the read/write bit indicating the
direction of the transaction. The slave acknowledges the receipt of it's address
with an acknowledge bit. The master sends data one byte at a time to the slave,
which must acknowledge the receipt of each byte for the next byte to be sent.
The master sends a STOP bit to complete the transaction. The slave can abort the
transaction by replying with a non-acknowledge bit instead of an acknowledge
bit.

The application programmer can choose not to send a STOP bit at the end of the
transaction causing the next transaction to begin with a repeated START bit.

## Read Transaction
The master I2C device initiates a read transaction by sending a START bit as
soon as the bus becomes free. The START bit is followed by the 7-bit serial
address of the target slave device followed by the read/write bit indicating the
direction of the transaction. The slave acknowledges the receipt of it's slave
address with an acknowledge bit. The slave sends data one byte at a time to the
master, which must acknowledge the receipt of each byte for the next byte to be
sent. The master sends a non-acknowledge bit following the last byte it wishes
to read followed by a STOP bit.

The application programmer can choose not to send a STOP bit at the end of the
transaction causing the next transaction to begin with a repeated START bit.

## Write-Read Transaction
The write-read transaction is a combination of a write transaction immediately
followed by a read transaction. There is no STOP bit in between the write and
read phases of a write-read transaction. A repeated START bit is sent between
the write and read phases.

Whilst the write handler is being executed, the slave holds the clock line low
to stretch the clock until the response is ready.

The write-read transaction is typically used to send a command or offset in the
write transaction specifying the logical data to be transferred during the read
phase.

The application programmer can choose not to send a STOP bit at the end of the
transaction causing the next transaction to begin with a repeated START bit.

## Master Operations
The application can use the I2C_write(), I2C_read(), and I2C_write_read()
functions to initiate an I2C bus transaction. The application can then wait for
the transaction to complete using the I2C_wait_complete() function or poll the
status of the I2C transaction using the I2C_get_status() function until it
returns a value different from I2C_IN_PROGRESS. The I2C_system_tick() function
is used to set a time base for the I2C_wait_complete() function's time out
delay.

## Slave Operations
To configure the I2C driver to operate as an I2C slave requires the use of the
following functions:

  - I2C_set_slave_tx_buffer()
  - I2C_set_slave_rx_buffer()
  - I2C_set_slave_mem_offset_length()  


  - I2C_register_write_handler()  


  - I2C_enable_slave()

Use of all functions is not required if the slave I2C does not need to support
all types of I2C read transactions. The subsequent sections list the functions
that must be used to support each transaction type.

## Responding to Read Transactions
The following functions are used to configure the CoreI2C driver to respond to
I2C read transactions: 
- I2C_set_slave_tx_buffer() 
- I2C_enable_slave()

The I2C_set_slave_tx_buffer() function specifies the data buffer that is
transmitted when the I2C slave is the target of an I2C read transaction. It is
then up to the application to manage the content of that buffer to control the
data that will be transmitted to the I2C master as a result of the read
transaction.

The I2C_enable_slave() function enables the I2C hardware instance to respond to
the I2C transactions. It must be called after the I2C driver has been configured
to respond to the required transaction types.

## Responding to Write Transactions
The following functions are used to configure the I2C driver to respond to I2C
write transactions: 
- I2C_set_slave_rx_buffer() 
- I2C_register_write_handler()
- I2C_enable_slave()

The I2C_set_slave_rx_buffer() function specifies the data buffer that stored the
data received by the I2C slave when it targets an I2C write transaction.

The I2C_register_write_handler() function specifies the handler function that
must be called on completion of the I2C write transaction. It is this handler
function that processes or triggers the processing of the received data.

The I2C_enable_slave() function enables the I2C hardware instance to respond to
I2C transactions. It must be called after the I2C driver has been configured to
respond to the required transaction types.

## Responding to Write-Read Transactions
The following functions are used to configure the CoreI2C driver to respond to
write-read transactions:

  - I2C_set_slave_mem_offset_length()
  - I2C_set_slave_tx_buffer()
  - I2C_set_slave_rx_buffer()
  - I2C_register_write_handler()
  - I2C_enable_slave()

The I2C_set_slave_mem_offset_length() function specifies the number of bytes
expected by the I2C slave during the write phase of the write-read transaction.

The I2C_set_slave_tx_buffer() function specifies the data that is transmitted to
the I2C master during the read phase of the write-read transaction. The value
received by the I2C slave during the write phase of the transaction will be used
as an index into the transmit buffer specified by this function. It decides
which part of the transmit buffer will be transmitted to the I2C master as part
of the read phase of the write-read transaction.

The I2C_set_slave_rx_buffer() function specifies the data buffer that stores the
data received by the I2C slave during the write phase of the write-read
transaction. This buffer must be large enough to accommodate the number of bytes
specified through the I2C_set_slave_mem_offset_length() function.

The I2C_register_write_handler() function can optionally be used to specify a
handler function that is called on completion of the write phase of the I2C
write-read transaction. If a handler function is registered, it is responsible
for processing the received data in the slave receive buffer and populating the
slave transmit buffer with the data that will be transmitted to the I2C master
as part of the read phase of the write-read transaction.

The I2C_enable_slave() function enables the CoreI2C hardware instance to respond
to the I2C transactions. It must be called after configuring the CoreI2C driver
to respond to the required transaction types.

## Mixed Master-Slave Operations
The CoreI2C device supports mixed master and slave operations. If the CoreI2C
slave has a transaction in progress and your application attempts to begin a
master mode transaction, the CoreI2C driver queues the master mode transaction
until the bus is released and the CoreI2C can switch to master mode and acquire
the bus. The CoreI2C master then starts the previously queued master
transaction.

## SMBus Control
The CoreI2C driver enables the CoreI2C peripheral’s SMBus functionality using
the I2C_smbus_init() function.

The I2C_suspend_smbus_slave() function is used with a master mode CoreI2C to
force slave devices on the SMBus to enter their Power-Down/Suspend mode. The
I2C_resume_smbus_slave() function is used to end the suspend operation on the
SMBus.

The I2C_reset_smbus() function is used with a master mode CoreI2C to force all
devices on the SMBus to reset their SMBUs interface.

The I2C_set_smsbus_alert() function is used by a slave mode CoreI2C to force
communication with the SMBus master. Once communications with the master is
initiated, the I2C_clear_smsbus_alert() function clears the alert condition.

The I2C_enable_smbus_irq() and I2C_disable_smbus_irq() functions are used to
enable and disable the SMBSUS and SMBALERT SMBus interrupts.

</div>


# Types

 ---------------- 
<a name="i2cchannelnumbert"></a>
## i2c_channel_number_t
<a name="prototype"></a>
### Prototype 

<div id="Types$i2c_channel_number_t$prototype" data-type="code">

 ``` 
    typedef enum {
        I2C_CHANNEL_0 = 0u,
        I2C_CHANNEL_1,
        I2C_CHANNEL_2,
        I2C_CHANNEL_3,
        I2C_CHANNEL_4,
        I2C_CHANNEL_5,
        I2C_CHANNEL_6,
        I2C_CHANNEL_7,
        I2C_CHANNEL_8,
        I2C_CHANNEL_9,
        I2C_CHANNEL_10,
        I2C_CHANNEL_11,
        I2C_CHANNEL_12,
        I2C_CHANNEL_13,
        I2C_CHANNEL_14,
        I2C_CHANNEL_15,
        I2C_MAX_CHANNELS = 16u
    } i2c_channel_number_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$i2c_channel_number_t$description" data-type="text">

The i2c_channel_number_t type is used to specify the channel number of a CoreI2C
instance.


</div>


 --------------------------- 
<a name="i2cclockdividert"></a>
## i2c_clock_divider_t
<a name="prototype"></a>
### Prototype 

<div id="Types$i2c_clock_divider_t$prototype" data-type="code">

 ``` 
    typedef enum {
        I2C_PCLK_DIV_256 = 0u,
        I2C_PCLK_DIV_224,
        I2C_PCLK_DIV_192,
        I2C_PCLK_DIV_160,
        I2C_PCLK_DIV_960,
        I2C_PCLK_DIV_120,
        I2C_PCLK_DIV_60,
        I2C_BCLK_DIV_8
    } i2c_clock_divider_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$i2c_clock_divider_t$description" data-type="text">

The i2c_clock_divider_t type specifies the divider to be applied to the I2C PCLK
or BCLK signal in order to generate the I2C clock. The I2C_BCLK_DIV_8 value
selects a clock frequency based on division of BCLK, all other values select a
clock frequency based on division of PCLK.


</div>


 --------------------------- 
<a name="i2cstatust"></a>
## i2c_status_t
<a name="prototype"></a>
### Prototype 

<div id="Types$i2c_status_t$prototype" data-type="code">

 ``` 
    typedef enum {
        I2C_SUCCESS = 0u,
        I2C_IN_PROGRESS,
        I2C_FAILED,
        I2C_TIMED_OUT
    } i2c_status_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$i2c_status_t$description" data-type="text">

The i2c_status_t type is used to report the status of I2C transactions.


</div>


 --------------------------- 
<a name="i2cslavehandlerrett"></a>
## i2c_slave_handler_ret_t
<a name="prototype"></a>
### Prototype 

<div id="Types$i2c_slave_handler_ret_t$prototype" data-type="code">

 ``` 
    typedef enum {
        I2C_REENABLE_SLAVE_RX = 0u,
        I2C_PAUSE_SLAVE_RX = 1u
    } i2c_slave_handler_ret_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$i2c_slave_handler_ret_t$description" data-type="text">

The i2c_slave_handler_ret_t type is used by slave write handler functions to
indicate whether or not the received data buffer should be released.


</div>


 --------------------------- 
<a name="i2cinstancet"></a>
## i2c_instance_t
<a name="prototype"></a>
### Prototype 

<div id="Types$i2c_instance_t$prototype" data-type="code">

``` 
    typedef struct i2c_instance {
        addr_t base_address; 
        uint_fast8_t ser_address; 
        uint_fast8_t target_addr; 
        uint8_t transaction; 
        uint_fast16_t random_read_addr; 
        uint8_t options; 
        const uint8_t *master_tx_buffer; 
        uint_fast16_t master_tx_size; 
        uint_fast16_t master_tx_idx; 
        uint_fast8_t dir; 
        uint8_t *master_rx_buffer; 
        uint_fast16_t master_rx_size; 
        uint_fast16_t master_rx_idx; 
        volatile i2c_status_t master_status; 
        uint32_t master_timeout_ms; 
        const uint8_t *slave_tx_buffer; 
        uint_fast16_t slave_tx_size; 
        uint_fast16_t slave_tx_idx; 
        uint8_t *slave_rx_buffer; 
        uint_fast16_t slave_rx_size; 
        uint_fast16_t slave_rx_idx; 
        volatile i2c_status_t slave_status; 
        uint_fast8_t slave_mem_offset_length; 
        i2c_slave_wr_handler_t slave_write_handler; 
        uint8_t is_slave_enabled; 
        void *p_user_data; 
        uint8_t bus_status; 
        uint8_t is_transaction_pending; 
        uint8_t pending_transaction; 
    } i2c_instance_t ;
  ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$i2c_instance_t$description" data-type="text">

This structure identifies various CoreI2C hardware instances in the system and
the I2C channels within them. The application software should declare one
instance of this structure for each channel of each instance of CoreI2C in your
system. I2C_init() and I2C_channel_init() functions initialize this structure
depending on whether it is channel 0 or one of the additional channels,
respectively. A pointer to an initialized instance of the structure should be
passed as the first parameter to the CoreI2C driver functions, to identify which
CoreI2C hardware instance and channel should perform the requested operation.

The contents of this data structure should not be modified or used outside of
the CoreI2C driver. Software using the CoreI2C driver should only need to create
one single instance of this data structure for each channel of each CoreI2C
hardware instance in the system then pass a pointer to these data structures
with each call to the CoreI2C driver in order to identify which CoreI2C hardware
instance to use.


</div>


 --------------------------- 
<a name="i2cslavewrhandlert"></a>
## i2c_slave_wr_handler_t
<a name="prototype"></a>
### Prototype 

<div id="Types$i2c_slave_wr_handler_t$prototype" data-type="code">

`typedef i2c_slave_handler_ret_t(* i2c_slave_wr_handler_t) (i2c_instance_t *instance, uint8_t *, uint16_t)`

</div>

<a name="description"></a>
### Description 

<div id="Types$i2c_slave_wr_handler_t$description" data-type="text">

This defines the function prototype that must be followed by I2C slave write
handler functions. These functions are registered with the CoreI2C driver
through the I2C_register_write_handler() function.

Declaring and Implementing Slave Write Handler Functions:  
Slave write handler
functions should follow the following prototype:


```
    i2c_slave_handler_ret_t write_handler
    ( 
        i2c_instance_t *instance, uint8_t * data, uint16_t size 
    );
```
The instance parameter is a pointer to the i2c_instance_t for which this slave
write handler has been declared.

The data parameter is a pointer to a buffer (received data buffer) holding the
data written to the I2C slave.

Define the INCLUDE_SLA_IN_RX_PAYLOAD macro for the driver to insert the actual
address used to access the slave as the first byte in the buffer. This allows
the applications to tailor their response based on the actual address used to
access the slave (primary address, secondary address, or GCA).

The size parameter is the number of bytes held in the received data buffer.
Handler functions must return one of the following values:

  - I2C_REENABLE_SLAVE_RX
  - I2C_PAUSE_SLAVE_RX

If the handler function returns I2C_REENABLE_SLAVE_RX, the driver releases the
received data buffer and allows further I2C write transactions to the I2C slave.

If the handler function returns I2C_PAUSE_SLAVE_RX, the I2C slave responds to
subsequent write requests with a non-acknowledge bit (NACK), until the received
data buffer content gets processed by some other part of the software
application.

Call the I2C_enable_slave() after returning the I2C_PAUSE_SLAVE_RX to release
the received data buffer in order to store the data received by the subsequent
I2C write transactions.


</div>


 --------------------------- 

# Constants

 ---------------- 
<div id="Constants$I2C_RELEASE_BUS$description" data-type="text">

<a name="i2creleasebus"></a>
## I2C_RELEASE_BUS
The I2C_RELEASE_BUS constant is used to specify the options parameter to
functions I2C_read(), I2C_write() and I2C_write_read() to indicate that a STOP
bit must be generated at the end of the I2C transaction to release the bus.

</div>

<div id="Constants$I2C_HOLD_BUS$description" data-type="text">

<a name="i2choldbus"></a>
## I2C_HOLD_BUS
The I2C_HOLD_BUS constant specify the options parameter to functions I2C_read(),
I2C_write(), and I2C_write_read() to indicate that a STOP bit must not be
generated at the end of the I2C transaction in order to retain the bus
ownership. This causes the next transaction to begin with a repeated START bit
and no STOP bit between the transactions.

</div>

<div id="Constants$I2C_NO_IRQ$description" data-type="text">

<a name="i2cnoirq"></a>
## Interrupt Identifier Number
The following constants specify the interrupt identifier number which is solely
used by the driver API. This has nothing to do with hardware interrupt line.
I2C_INTR_IRQ is the primary interrupt signal which drives the state machine of
the CoreI2C driver. The I2C_SMBALERT_IRQ and I2C_SMBSUS_IRQ are used by SMBus
interrupt enable and disable functions. These IRQ numbers are also used by
I2C_get_irq_status().

| Constant  | Description   | 
| -----|-----|
| I2C_NO_IRQ  | No interrupt   | 
| I2C_SMBALERT_IRQ  | Used by SMBus interrupt enable functions   | 
| I2C_SMBSUS_IRQ  | Used by SMBus interrupt disable functions   | 
| I2C_INTR_IRQ  | Primary interrupt signal which drives the state machine   | 

</div>

<div id="Constants$I2C_NO_TIMEOUT$description" data-type="text">

<a name="i2cnotimeout"></a>
## I2C_NO_TIMEOUT
The I2C_wait_complete() function uses I2C_NO_TIMEOUT constant as a parameter to
indicate that the wait for completion of the transaction should not time out.

</div>


# Functions

 ---------------- 
<a name="i2cinit"></a>
## I2C_init
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_init$prototype" data-type="code">

    void
    I2C_init
    (
        i2c_instance_t * this_i2c,
        addr_t base_address,
        uint8_t ser_address,
        i2c_clock_divider_t ser_clock_speed
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_init$description" data-type="text">

The I2C_init() function configures channel 0 of a CoreI2C instance. It sets the
base hardware address which is used to locate the CoreI2C instance in memory and
also used internally by I2C_channel_init() to calculate the register addresses
for any additional channels. The slave serial address set is shared by all
channels on a CoreI2C instance.  
If only one channel is configured in a
CoreI2C, the address of the i2c_instance_t used in I2C_Init() will also be used
in subsequent calls to the CoreI2C driver functions. If more than one channel is
configured in the CoreI2C, I2C_channel_init() will be called after I2C_init(),
which initializes the i2c_instance_t data structure for a specific channel.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_init$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

Pointer to the i2c_instance_t data structure that holds all the data related to
channel 0 of the CoreI2C instance is initialized. A pointer to this structure is
used in all subsequent calls to the CoreI2C driver functions which operates on
channel 0 of this CoreI2C instance.


</div>

#### base_address
<div id="Functions$I2C_init$description$parameters$base_address" data-type="text" data-name="base_address">

Base address in the processor's memory map of the registers of the CoreI2C
instance being initialized.


</div>

#### ser_address
<div id="Functions$I2C_init$description$parameters$ser_address" data-type="text" data-name="ser_address">

This parameter sets the primary I2C serial address (SLAVE0 address) for the
CoreI2C to initialize. It is the principal I2C bus address to which the CoreI2C
instance will respond. CoreI2C can operate in master mode or slave mode and the
serial address is significant only in the case of I2C slave mode. In master
mode, CoreI2C does not require a serial address and the value of this parameter
is not important. If you do not intend to use the CoreI2C device in slave mode,
then provide any dummy slave address value to this parameter. However, in
systems where the CoreI2C is expected to switch from master mode to slave mode,
it is advisable to initialize the CoreI2C device with a valid serial slave
address. Call the I2C_init() function whenever it is required to change the
primary slave address as there is no separate function to set the primary slave
address of the I2C device. The serial address initialized through this function
is basically the primary slave address or slave address0.
I2C_set_slave_second_addr() is used to set the secondary slave address or slave
address 1.  
Note : ser_address parameter does not have any affect if fixed
slave address is enabled in CoreI2C hardware design. CoreI2C will be always
addressed with the hardware configured fixed slave address.  

Note : ser_address parameter will not have any affect if the CoreI2C instance is only used in 
master mode.


</div>

#### ser_clock_speed
<div id="Functions$I2C_init$description$parameters$ser_clock_speed" data-type="text" data-name="ser_clock_speed">

This parameter sets the I2C serial clock frequency. It selects the divider that
generates the serial clock from the APB PCLK or from the BCLK. It can be one of
the following:

  - I2C_PCLK_DIV_256
  - I2C_PCLK_DIV_224
  - I2C_PCLK_DIV_192
  - I2C_PCLK_DIV_160
  - I2C_PCLK_DIV_960
  - I2C_PCLK_DIV_120
  - I2C_PCLK_DIV_60
  - I2C_BCLK_DIV_8  
Note: serial_clock_speed value does not have any affect if 
the fixed baud rate is enabled in CoreI2C hardware instance configuration 
dialogue window. The fixed baud rate divider value overrides the value passed as
 parameter in this function.  
Note: serial_clock_speed value is not critical 
for devices that only operate as slaves and can be set to any of the above 
values.  




</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_init$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_init$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_init$description$example$Example1" data-type="code" data-name="Example">


```
    #define COREI2C_BASE_ADDR  0xC0000000u
    #define COREI2C_SER_ADDR   0x10u

    i2c_instance_t g_i2c_inst;

    void system_init( void )
    {
        I2C_init( &g_i2c_inst, COREI2C_BASE_ADDR, COREI2C_SER_ADDR,
                  I2C_PCLK_DIV_256 );
    }
```

</div>


 -------------------------------- 
<a name="i2cchannelinit"></a>
## I2C_channel_init
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_channel_init$prototype" data-type="code">

    void
    I2C_channel_init
    (
        i2c_instance_t * this_i2c_channel,
        i2c_instance_t * this_i2c,
        i2c_channel_number_t channel_number,
        i2c_clock_divider_t ser_clock_speed
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_channel_init$description" data-type="text">

The I2C_channel_init() function initializes and configures hardware and data
structures of one of the additional channels of a CoreI2C instance. I2C_init()
must be called before calling this function to set the CoreI2C instance hardware
base address and I2C serial address. I2C_channel_init() also initializes I2C
serial clock divider to set the serial clock baud rate. The pointer to data
structure i2c_instance_t used for a particular channel is used as an input
parameter to subsequent CoreI2C driver functions which operate on this channel.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c_channel
<div id="Functions$I2C_channel_init$description$parameters$this_i2c_channel" data-type="text" data-name="this_i2c_channel">

Pointer to the i2c_instance_t data structure that holds all data related to the
CoreI2C channel gets initialized. A pointer to the same data structure is used
in subsequent calls to the CoreI2C driver functions in order to identify the
CoreI2C channel instance that should perform the operation implemented by the
called driver function.


</div>

#### this_i2c
<div id="Functions$I2C_channel_init$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

This is a pointer to an i2c_instance_t structure, previously initialized by
I2C_init(). It holds information regarding the hardware base address and I2C
serial address for the CoreI2C containing the channel to be initialized. This
information is required by I2C_channel_init() to initialize the i2c_instance_t
structure pointed by this_i2c_channel as all channels in a CoreI2C instance
share the same base address and serial address. It is very important that the
i2c_instance_t structure pointed by this_i2c must be previously initialized by
calling I2C_init().


</div>

#### channel_number
<div id="Functions$I2C_channel_init$description$parameters$channel_number" data-type="text" data-name="channel_number">

This parameter of type i2c_channel_number_t identifies the channel to be
initialized.


</div>

#### ser_clock_speed
<div id="Functions$I2C_channel_init$description$parameters$ser_clock_speed" data-type="text" data-name="ser_clock_speed">

This parameter sets the I2C serial clock frequency. It selects the divider that
is used to generate the serial clock from the APB PCLK or from the BCLK. It can
be one of the following:

  - I2C_PCLK_DIV_256
  - I2C_PCLK_DIV_224
  - I2C_PCLK_DIV_192
  - I2C_PCLK_DIV_160
  - I2C_PCLK_DIV_960
  - I2C_PCLK_DIV_120
  - I2C_PCLK_DIV_60
  - I2C_BCLK_DIV_8  
Note: serial_clock_speed value does not have any affect if 
the fixed baud rate is enabled in CoreI2C hardware instance configuration 
dialogue window. The fixed baud rate divider value will supersede the value 
passed as parameter in this function.  
Note: ser_clock_speed value is not 
critical for devices that only operate as slaves and can be set to any of the 
above values.  



</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_channel_init$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_channel_init$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_channel_init$description$example$Example1" data-type="code" data-name="Example">


```
    #define COREI2C_BASE_ADDR  0xC0000000u
    #define COREI2C_SER_ADDR   0x10u
    #define DATA_LENGTH        16u

    i2c_instance_t g_i2c_inst;
    i2c_instance_t g_i2c_channel_1_inst;

    uint8_t  tx_buffer[DATA_LENGTH];
    uint8_t  write_length = DATA_LENGTH;

    void system_init( void )
    {
        uint8_t  target_slave_addr = 0x12;
        
        // Initialize base CoreI2C instance
        I2C_init( &g_i2c_inst, COREI2C_BASE_ADDR, COREI2C_SER_ADDR,
                  I2C_PCLK_DIV_256 );
        
        // Initialize CoreI2C channel 1 with different clock speed
        I2C_channel_init( &g_i2c_channel_1_inst, &g_i2c_inst, I2C_CHANNEL_1,
                          I2C_PCLK_DIV_224 );
        
        // Write data to Channel 1 of CoreI2C instance.
        I2C_write( &g_i2c_channel_1_inst, target_slave_addr, tx_buffer,
                   write_length, I2C_RELEASE_BUS );
    }
```

</div>


 -------------------------------- 
<a name="i2cisr"></a>
## I2C_isr
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_isr$prototype" data-type="code">

    void
    I2C_isr
    (
        i2c_instance_t * this_i2c
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_isr$description" data-type="text">

The I2C_isr function is the CoreI2C interrupt service routine. User must call
this function from their application level CoreI2C interrupt handler function.
This function runs the I2C state machine based on previous and current status.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_isr$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_isr$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_isr$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_isr$description$example$Example1" data-type="code" data-name="Example">


```
     #define COREI2C_BASE_ADDR  0xC0000000u
     #define COREINTERRUPT_BASE_ADDR  0xCC000000u
     #define COREI2C_SER_ADDR   0x10u
     #define I2C_IRQ_NB           2u
     
     i2c_instance_t g_i2c_inst;

     void core_i2c_isr( void )
     {
        I2C_isr( &g_i2c_inst );
     }
     
    void main( void )
     {
         CIC_init( COREINTERRUPT_BASE_ADDR );
         NVIC_init();
         CIC_set_irq_handler( I2C_IRQ_NB, core_i2c_isr );
         I2C_init( &g_i2c_inst, COREI2C_BASE_ADDR, COREI2C_SER_ADDR,
                   I2C_PCLK_DIV_256 );
         NVIC_enable_interrupt( NVIC_IRQ_0 );
     }
```

</div>


 -------------------------------- 
<a name="i2cwrite"></a>
## I2C_write
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_write$prototype" data-type="code">

    void
    I2C_write
    (
        i2c_instance_t * this_i2c,
        uint8_t serial_addr,
        const uint8_t * write_buffer,
        uint16_t write_size,
        uint8_t options
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_write$description" data-type="text">

This function initiates an I2C master write transaction. This function returns
immediately after initiating the transaction. The content of the write buffer
passed as parameter should not be modified until the write transaction
completes. It also means that the memory allocated for the write buffer should
not be freed or should not go out of scope before the write completes. You can
check for the write transaction completion using the I2C_status() function.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_write$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

#### serial_addr
<div id="Functions$I2C_write$description$parameters$serial_addr" data-type="text" data-name="serial_addr">

This parameter specifies the serial address of the target I2C device.


</div>

#### write_buffer
<div id="Functions$I2C_write$description$parameters$write_buffer" data-type="text" data-name="write_buffer">

This parameter is a pointer to a buffer holding the data to be written to the
target I2C device. Do not to release the memory used by this buffer before the
write transaction completes. For example, it is not appropriate to return from a
function allocating this buffer as an auto array variable before the write
transaction completes as this would result in the buffer's memory being de-
allocated from the stack when the function returns. This memory could then be
subsequently reused and modified causing unexpected data to be written to the
target I2C device.


</div>

#### write_size
<div id="Functions$I2C_write$description$parameters$write_size" data-type="text" data-name="write_size">

Number of bytes held in the write_buffer to be written to the target I2C device.


</div>

#### options
<div id="Functions$I2C_write$description$parameters$options" data-type="text" data-name="options">

The options parameter is used to indicate if the I2C bus should be released on
completion of the write transaction. Using the I2C_RELEASE_BUS constant for the
options parameter causes a STOP bit to be generated at the end of the write
transaction causing the bus to be released for other I2C devices to use. Using
the I2C_HOLD_BUS constant as options parameter prevents a STOP bit from being
generated at the end of the write transaction, preventing other I2C devices from
initiating a bus transaction.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_write$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_write$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_write$description$example$Example1" data-type="code" data-name="Example">


```
    #define COREI2C_BASE_ADDR  0xC0000000u
    #define COREI2C_DUMMY_ADDR   0x10u
    #define DATA_LENGTH           16u

    i2c_instance_t g_i2c_inst;

    uint8_t  tx_buffer[DATA_LENGTH];
    uint8_t  write_length = DATA_LENGTH;     

    void main( void )
    {
        uint8_t  target_slave_addr = 0x12;
        i2c_status_t status;
        
        // Initialize base CoreI2C instance
        I2C_init( &g_i2c_inst, COREI2C_BASE_ADDR, COREI2C_DUMMY_ADDR,
                  I2C_PCLK_DIV_256 );
        
        // Write data to Channel 0 of CoreI2C instance.
        I2C_write( &g_i2c_inst, target_slave_addr, tx_buffer, write_length,
                   I2C_RELEASE_BUS );
                   
        // Wait for completion and record the outcome
        status = I2C_wait_complete( &g_i2c_inst, I2C_NO_TIMEOUT );
    }
```

</div>


 -------------------------------- 
<a name="i2cread"></a>
## I2C_read
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_read$prototype" data-type="code">

    void
    I2C_read
    (
        i2c_instance_t * this_i2c,
        uint8_t serial_addr,
        uint8_t * read_buffer,
        uint16_t read_size,
        uint8_t options
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_read$description" data-type="text">

This function initiates an I2C master read transaction. This function returns
immediately after initiating the transaction. The contents of the read buffer
passed as parameter should not be modified until the read transaction completes.
It also means that the memory allocated for the read buffer should not be freed
or should not go out of scope before the read completes. You can check for the
read transaction completion using the I2C_status() function.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_read$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

#### serial_addr
<div id="Functions$I2C_read$description$parameters$serial_addr" data-type="text" data-name="serial_addr">

This parameter specifies the serial address of the target I2C device.


</div>

#### read_buffer
<div id="Functions$I2C_read$description$parameters$read_buffer" data-type="text" data-name="read_buffer">

This is a pointer to a buffer where the data received from the target device
gets stored. Do not to release the memory used by this buffer before the read
transaction completes. For example, it is not appropriate to return from a
function allocating this buffer as an auto array variable before the read
transaction completes as this would result in the buffer's memory being de-
allocated from the stack when the function returns. This memory could then be
subsequently reallocated resulting in the read transaction corrupting the newly
allocated memory.


</div>

#### read_size
<div id="Functions$I2C_read$description$parameters$read_size" data-type="text" data-name="read_size">

This parameter specifies the number of bytes to read from the target device.
This size must not exceed the size of the read_buffer buffer.


</div>

#### options
<div id="Functions$I2C_read$description$parameters$options" data-type="text" data-name="options">

The options parameter is used to indicate if the I2C bus should be released on
completion of the read transaction. Using the I2C_RELEASE_BUS constant for the
options parameter causes a STOP bit to be generated at the end of the read
transaction causing the bus to be released for other I2C devices to use. Using
the I2C_HOLD_BUS constant as options parameter prevents a STOP bit from being
generated at the end of the read transaction, preventing other I2C devices from
initiating a bus transaction.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_read$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_read$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_read$description$example$Example1" data-type="code" data-name="Example">


```
    #define COREI2C_BASE_ADDR  0xC0000000u
    #define COREI2C_DUMMY_ADDR   0x10u
    #define DATA_LENGTH           16u

    i2c_instance_t g_i2c_inst;

    uint8_t  rx_buffer[DATA_LENGTH];
    uint8_t  read_length = DATA_LENGTH;     

    void main( void )
    {
        uint8_t  target_slave_addr = 0x12;
        i2c_status_t status;
        
        // Initialize base CoreI2C instance
        I2C_init( &g_i2c_inst, COREI2C_BASE_ADDR, COREI2C_DUMMY_ADDR,
                  I2C_PCLK_DIV_256 );
        
        // Read data from target slave Channel 0 of CoreI2C instance.
        I2C_read( &g_i2c_inst, target_slave_addr, rx_buffer, read_length,
                  I2C_RELEASE_BUS );
        
        status = I2C_wait_complete( &g_i2c_inst, I2C_NO_TIMEOUT );
    }
```

</div>


 -------------------------------- 
<a name="i2cwriteread"></a>
## I2C_write_read
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_write_read$prototype" data-type="code">

    void
    I2C_write_read
    (
        i2c_instance_t * this_i2c,
        uint8_t serial_addr,
        const uint8_t * addr_offset,
        uint16_t offset_size,
        uint8_t * read_buffer,
        uint16_t read_size,
        uint8_t options
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_write_read$description" data-type="text">

This function initiates an I2C write-read transaction where data is first
written to the target device before issuing a restart condition and changing the
direction of the I2C transaction in order to read from the target device.

The same warnings about buffer allocation in I2C_write() and I2C_read() applies
to this function.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_write_read$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

#### serial_addr
<div id="Functions$I2C_write_read$description$parameters$serial_addr" data-type="text" data-name="serial_addr">

This parameter specifies the serial address of the target I2C device.


</div>

#### addr_offset
<div id="Functions$I2C_write_read$description$parameters$addr_offset" data-type="text" data-name="addr_offset">

This parameter is a pointer to the buffer containing the data that is sent to
the slave during the write phase of the write-read transaction. This data is
typically used to specify an address offset specifying to the I2C slave device
what data it must return during the read phase of the write-read transaction.


</div>

#### offset_size
<div id="Functions$I2C_write_read$description$parameters$offset_size" data-type="text" data-name="offset_size">

This parameter specifies the number of offset bytes to be written during the
write phase of the write-read transaction. This is typically the size of the
buffer pointed by the addr_offset parameter.


</div>

#### read_buffer
<div id="Functions$I2C_write_read$description$parameters$read_buffer" data-type="text" data-name="read_buffer">

This parameter is a pointer to the buffer where the data read from the I2C slave
will be stored.


</div>

#### read_size
<div id="Functions$I2C_write_read$description$parameters$read_size" data-type="text" data-name="read_size">

This parameter specifies the number of bytes to read from the target I2C slave
device. This size must not exceed the size of the buffer pointed by the
read_buffer parameter.


</div>

#### options
<div id="Functions$I2C_write_read$description$parameters$options" data-type="text" data-name="options">

The options parameter is used to indicate if the I2C bus should be released on
completion of the write-read transaction. Using the I2C_RELEASE_BUS constant for
the options parameter causes a STOP bit to be generated at the end of the write-
read transaction causing the bus to be released for other I2C devices to use.
Using the I2C_HOLD_BUS constant as options parameter prevents a STOP bit from
being generated at the end of the write-read transaction, preventing other I2C
devices from initiating a bus transaction.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_write_read$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_write_read$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_write_read$description$example$Example1" data-type="code" data-name="Example">


```
    #define COREI2C_BASE_ADDR    0xC0000000u
    #define COREI2C_DUMMY_ADDR   0x10u
    #define TX_LENGTH            16u
    #define RX_LENGTH            8u

    i2c_instance_t g_i2c_inst;
    uint8_t  rx_buffer[RX_LENGTH];
    uint8_t  read_length = RX_LENGTH;     
    uint8_t  tx_buffer[TX_LENGTH];
    uint8_t  write_length = TX_LENGTH;  

    void main( void )
    {
        uint8_t  target_slave_addr = 0x12;
        i2c_status_t status;
        // Initialize base CoreI2C instance
        I2C_init( &g_i2c_inst, COREI2C_BASE_ADDR, COREI2C_DUMMY_ADDR,
                  I2C_PCLK_DIV_256 );
                  
        I2C_write_read( &g_i2c_inst, target_slave_addr, tx_buffer, write_length,
                        rx_buffer, read_length, I2C_RELEASE_BUS );
                        
        status = I2C_wait_complete( &g_i2c_inst, I2C_NO_TIMEOUT );
    }
```

</div>


 -------------------------------- 
<a name="i2cgetstatus"></a>
## I2C_get_status
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_get_status$prototype" data-type="code">

    i2c_status_t
    I2C_get_status
    (
        i2c_instance_t * this_i2c
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_get_status$description" data-type="text">

This function indicates the current state of a CoreI2C channel.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_get_status$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_get_status$description$return" data-type="text">

The return value indicates the current state of a CoreI2C channel or the outcome
of the previous transaction if no transaction is in progress. Following are the
return values are:

  - I2C_SUCCESS  
The last I2C transaction has completed successfully.  


  - I2C_IN_PROGRESS  
There is an I2C transaction in progress.
  - I2C_FAILED  
The last I2C transaction failed.
  - I2C_TIMED_OUT  
The request has failed to complete in the allotted time.  




</div>

##### Example
<div id="Functions$I2C_get_status$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_get_status$description$example$Example1" data-type="code" data-name="Example">


```
    i2c_instance_t g_i2c_inst;

    while( I2C_IN_PROGRESS == I2C_get_status( &g_i2c_inst ) )
    {
        // Do something useful while waiting for I2C operation to complete
        our_i2c_busy_task();
    }

    if( I2C_SUCCESS != I2C_get_status( &g_i2c_inst ) )
    {
        // Something went wrong... 
        our_i2c_error_recovery( &g_i2c_inst );
    }
```

</div>


 -------------------------------- 
<a name="i2cwaitcomplete"></a>
## I2C_wait_complete
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_wait_complete$prototype" data-type="code">

    i2c_status_t
    I2C_wait_complete
    (
        i2c_instance_t * this_i2c,
        uint32_t timeout_ms
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_wait_complete$description" data-type="text">

This function waits for the current I2C transaction to complete. The return
value indicates whether the last I2C transaction was successful or not.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_wait_complete$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

#### timeout_ms
<div id="Functions$I2C_wait_complete$description$parameters$timeout_ms" data-type="text" data-name="timeout_ms">

The timeout_ms parameter specifies the delay within which the current I2C
transaction should complete. The time out delay is given in milliseconds.
I2C_wait_complete() will return I2C_TIMED_OUT if the current transaction has not
completed after the time out delay has expired. This parameter can be set to
I2C_NO_TIMEOUT to indicate that I2C_wait_complete() must not time out.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_wait_complete$description$return" data-type="text">

The return value indicates the outcome of the last I2C transaction. It can be
one of the following:

  - I2C_SUCCESS  
The last I2C transaction has completed successfully.
  - I2C_FAILED  
The last I2C transaction failed.
  - I2C_TIMED_OUT  
The last transaction failed to complete within the time out 
delay given as second parameter.


</div>

##### Example
<div id="Functions$I2C_wait_complete$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_wait_complete$description$example$Example1" data-type="code" data-name="Example">


```
    #define COREI2C_BASE_ADDR  0xC0000000u
    #define COREI2C_DUMMY_ADDR   0x10u
    #define DATA_LENGTH           16u

    i2c_instance_t g_i2c_inst;

    uint8_t  rx_buffer[DATA_LENGTH];
    uint8_t  read_length = DATA_LENGTH;     

    void main( void )
    {
        uint8_t  target_slave_addr = 0x12;
        i2c_status_t status;
        
        // Initialize base CoreI2C instance
        I2C_init( &g_i2c_inst, COREI2C_BASE_ADDR, COREI2C_DUMMY_ADDR,
                  I2C_PCLK_DIV_256 );
        
        // Read data from Channel 0 of CoreI2C instance.
        I2C_read( &g_i2c_inst, target_slave_addr, rx_buffer, read_length,
                  I2C_RELEASE_BUS );
        
        // Wait for completion and record the outcome
        status = I2C_wait_complete( &g_i2c_inst, I2C_NO_TIMEOUT );
    }
```

</div>


 -------------------------------- 
<a name="i2csystemtick"></a>
## I2C_system_tick
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_system_tick$prototype" data-type="code">

    void
    I2C_system_tick
    (
        i2c_instance_t * this_i2c,
        uint32_t ms_since_last_tick
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_system_tick$description" data-type="text">

This function is used to control the expiration of the time out delay specified
as a parameter to the I2C_wait_complete() function. It must be called from the
interrupt service routine of a periodic interrupt source such as the SysTick
timer interrupt. It takes the period of the interrupt source as its
ms_since_last_tick parameter and uses it as the time base for the
I2C_wait_complete() function's time out delay.

Note: This function does not need to be called if the I2C_wait_complete()
function is called with a timeout_ms value of I2C_NO_TIMEOUT.  
Note: If this
function is not called then the I2C_wait_complete() function will behave as if
its timeout_ms was specified as I2C_NO_TIMEOUT and it will not time out.  
Note:
If this function is being called from an interrupt handler (for example,
SysTick) it is important that the calling interrupt have a lower priority than
the CoreI2C interrupt(s) to ensure any updates to the shared data are protected.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_system_tick$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

#### ms_since_last_tick
<div id="Functions$I2C_system_tick$description$parameters$ms_since_last_tick" data-type="text" data-name="ms_since_last_tick">

The ms_since_last_tick parameter specifies the number of milliseconds that
elapsed since the last call to I2C_system_tick(). This parameter would typically
be a constant specifying the interrupt rate of a timer used to generate system
ticks.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_system_tick$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_system_tick$description$example$Example1" data-type="text" data-name="Example">
The following example shows how the I2C_system_tick() function.
I2C_system_tick() is called for each I2C channel from the SysTick timer
interrupt service routine. The following example shows how the SysTick is
configured to generate an interrupt in every 10 milliseconds.


</div>

<div id="Functions$I2C_system_tick$description$example$Example1" data-type="code" data-name="Example">


```
    #define SYSTICK_INTERVAL_MS 10

    void SysTick_Handler(void)
    {
        I2C_system_tick(&g_core_i2c0, SYSTICK_INTERVAL_MS);
        I2C_system_tick(&g_core_i2c2, SYSTICK_INTERVAL_MS);
    }
```

</div>


 -------------------------------- 
<a name="i2csetslavetxbuffer"></a>
## I2C_set_slave_tx_buffer
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_set_slave_tx_buffer$prototype" data-type="code">

    void
    I2C_set_slave_tx_buffer
    (
        i2c_instance_t * this_i2c,
        const uint8_t * tx_buffer,
        uint16_t tx_size
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_set_slave_tx_buffer$description" data-type="text">

This function specifies the memory buffer holding the data that will be sent to
the I2C master when this CoreI2C channel is the target of an I2C read or write-
read transaction.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_set_slave_tx_buffer$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

#### tx_buffer
<div id="Functions$I2C_set_slave_tx_buffer$description$parameters$tx_buffer" data-type="text" data-name="tx_buffer">

This parameter is a pointer to the memory buffer holding the data to be returned
to the I2C master when this CoreI2C channel is the target of an I2C read or
write-read transaction.


</div>

#### tx_size
<div id="Functions$I2C_set_slave_tx_buffer$description$parameters$tx_size" data-type="text" data-name="tx_size">

Size of the transmit buffer pointed by the tx_buffer parameter.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_set_slave_tx_buffer$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_set_slave_tx_buffer$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_set_slave_tx_buffer$description$example$Example1" data-type="code" data-name="Example">


```
    #define COREI2C_BASE_ADDR    0xC0000000u
    #define SLAVE_SER_ADDR       0x10u
    #define SLAVE_TX_BUFFER_SIZE 10u

    i2c_instance_t g_i2c_inst;

    uint8_t g_slave_tx_buffer[SLAVE_TX_BUFFER_SIZE] = { 1, 2, 3, 4, 5,
                                                        6, 7, 8, 9, 10 };

    void main( void )
    {
        // Initialize the CoreI2C driver with its base address, I2C serial
        // address and serial clock divider.
        I2C_init( &g_i2c_inst, COREI2C_BASE_ADDR, SLAVE_SER_ADDR,
                  I2C_PCLK_DIV_256 );
       
        // Specify the transmit buffer containing the data that will be
        // returned to the master during read and write-read transactions.
        I2C_set_slave_tx_buffer( &g_i2c_inst, g_slave_tx_buffer, 
                                 sizeof(g_slave_tx_buffer) );
    }
```

</div>


 -------------------------------- 
<a name="i2csetslaverxbuffer"></a>
## I2C_set_slave_rx_buffer
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_set_slave_rx_buffer$prototype" data-type="code">

    void
    I2C_set_slave_rx_buffer
    (
        i2c_instance_t * this_i2c,
        uint8_t * rx_buffer,
        uint16_t rx_size
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_set_slave_rx_buffer$description" data-type="text">

This function specifies the memory buffer that is used by the CoreI2C channel to
receive data when it is a slave. This buffer is the memory where data gets
stored when the CoreI2C channel is the target of an I2C master write transaction
(that is, when it is the slave).

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_set_slave_rx_buffer$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

#### rx_buffer
<div id="Functions$I2C_set_slave_rx_buffer$description$parameters$rx_buffer" data-type="text" data-name="rx_buffer">

This parameter is a pointer to the memory buffer allocated by the caller
software to be used as a slave receive buffer.


</div>

#### rx_size
<div id="Functions$I2C_set_slave_rx_buffer$description$parameters$rx_size" data-type="text" data-name="rx_size">

Size of the slave receive buffer. This is the amount of memory allocated to the
buffer pointed by rx_buffer.  
Note: Indirectly, this buffer size specifies the
maximum I2C write transaction length this CoreI2C channel targets. This is
because this CoreI2C channel responds to further received bytes with a non-
acknowledge bit (NACK) as soon as its receive buffer is full. This causes the
write transaction to fail.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_set_slave_rx_buffer$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_set_slave_rx_buffer$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_set_slave_rx_buffer$description$example$Example1" data-type="code" data-name="Example">


```
    #define COREI2C_BASE_ADDR    0xC0000000u
    #define SLAVE_SER_ADDR       0x10u
    #define SLAVE_RX_BUFFER_SIZE 10u

    i2c_instance_t g_i2c_inst;

    uint8_t g_slave_rx_buffer[SLAVE_RX_BUFFER_SIZE];

    void main( void )
    {
        // Initialize the CoreI2C driver with its base address, I2C serial
        // address and serial clock divider.
        I2C_init( &g_i2c_inst, COREI2C_BASE_ADDR, SLAVE_SER_ADDR,
                  I2C_PCLK_DIV_256 );
       
        // Specify the buffer used to store the data written by the I2C master.
        I2C_set_slave_rx_buffer( &g_i2c_inst, g_slave_rx_buffer, 
                                 sizeof(g_slave_rx_buffer) );
    }
```

</div>


 -------------------------------- 
<a name="i2csetslavememoffsetlength"></a>
## I2C_set_slave_mem_offset_length
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_set_slave_mem_offset_length$prototype" data-type="code">

    void
    I2C_set_slave_mem_offset_length
    (
        i2c_instance_t * this_i2c,
        uint8_t offset_length
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_set_slave_mem_offset_length$description" data-type="text">

This function is used as part of the configuration of a CoreI2C channel to
operate as a slave supporting write-read transactions. It specifies the number
of bytes expected as part of the write phase of a write-read transaction. The
bytes received during the write phase of a write-read transaction will be
interpreted as an offset into the slave's transmit buffer. This allows random
access into the I2C slave transmit buffer from a remote I2C master.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_set_slave_mem_offset_length$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

#### offset_length
<div id="Functions$I2C_set_slave_mem_offset_length$description$parameters$offset_length" data-type="text" data-name="offset_length">

The offset_length parameter configures the number of bytes to be interpreted by
the CoreI2C slave as a memory offset value during the write phase of write-read
transactions. The maximum value for the offset_length parameter is two. The
value of offset_length has the following effect on the interpretation of the
received data. 
- If offset_length is 0, the offset into the transmit buffer is
fixed at 0.  

- If offset_length is 1, a single byte of received data is
interpreted as an unsigned 8-bit offset value in the range 0 to 255.  

- If
offset_length is 2, 2 bytes of received data are interpreted as an unsigned
16-bit offset value in the range 0 to 65535. The first byte received in this
case provides the high order bits of the offset and the second byte provides the
low order bits.
  
If the number of bytes received does not match the non 0 value
of offset_length, the transmit buffer offset is set to 0.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_set_slave_mem_offset_length$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_set_slave_mem_offset_length$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_set_slave_mem_offset_length$description$example$Example1" data-type="code" data-name="Example">


```
    #define COREI2C_BASE_ADDR    0xC0000000u
    #define SLAVE_SER_ADDR       0x10u
    #define SLAVE_TX_BUFFER_SIZE 10u

    i2c_instance_t g_i2c_inst;

    uint8_t g_slave_tx_buffer[SLAVE_TX_BUFFER_SIZE] = { 1, 2, 3, 4, 5,
                                                        6, 7, 8, 9, 10 };

    void main( void )
    {
        // Initialize the CoreI2C driver with its base address, I2C serial
        // address and serial clock divider.
        I2C_init( &g_i2c_inst, COREI2C_BASE_ADDR, SLAVE_SER_ADDR,
                  I2C_PCLK_DIV_256 );
        I2C_set_slave_tx_buffer( &g_i2c_inst, g_slave_tx_buffer, 
                                 sizeof(g_slave_tx_buffer) );
        I2C_set_slave_mem_offset_length( &g_i2c_inst, 1 );
    }
```

</div>


 -------------------------------- 
<a name="i2cregisterwritehandler"></a>
## I2C_register_write_handler
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_register_write_handler$prototype" data-type="code">

    void
    I2C_register_write_handler
    (
        i2c_instance_t * this_i2c,
        i2c_slave_wr_handler_t handler
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_register_write_handler$description" data-type="text">

Register the function that is called to process the data written to this CoreI2C
channel when it is the slave in an I2C write transaction.

Note: If a write handler is registered, it is called on completion of the write
phase of a write-read transaction and responsible for processing the received
data in the slave receive buffer and populating the slave transmit buffer with
the data that is transmitted to the I2C master as part of the read phase of the
write-read transaction. If a write handler is not registered, the write data of
a write-read transaction is interpreted as an offset into the slave’s transmit
buffer and handled by the driver.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_register_write_handler$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

#### handler
<div id="Functions$I2C_register_write_handler$description$parameters$handler" data-type="text" data-name="handler">

Pointer to the function that processes the I2C write request.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_register_write_handler$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_register_write_handler$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_register_write_handler$description$example$Example1" data-type="code" data-name="Example">


```
    #define COREI2C_BASE_ADDR    0xC0000000u
    #define SLAVE_SER_ADDR       0x10u
    #define SLAVE_TX_BUFFER_SIZE 10u

    i2c_instance_t g_i2c_inst;

    uint8_t g_slave_tx_buffer[SLAVE_TX_BUFFER_SIZE] = { 1, 2, 3, 4, 5,
                                                       6, 7, 8, 9, 10 };

    // local function prototype
    void slave_write_handler
    (
        i2c_instance_t * this_i2c,
        uint8_t * p_rx_data,
        uint16_t rx_size
    );

    void main( void )
    {
        // Initialize the CoreI2C driver with its base address, I2C serial
        // address and serial clock divider.
        I2C_init( &g_i2c_inst, COREI2C_BASE_ADDR, SLAVE_SER_ADDR, 
                  I2C_PCLK_DIV_256 );
        I2C_set_slave_tx_buffer( &g_i2c_inst, g_slave_tx_buffer, 
                                 sizeof(g_slave_tx_buffer) );
        I2C_set_slave_mem_offset_length( &g_i2c_inst, 1 );
        I2C_register_write_handler( &g_i2c_inst, slave_write_handler );
    }
```

</div>


 -------------------------------- 
<a name="i2cenableslave"></a>
## I2C_enable_slave
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_enable_slave$prototype" data-type="code">

    void
    I2C_enable_slave
    (
        i2c_instance_t * this_i2c
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_enable_slave$description" data-type="text">

This function enables slave mode operation for a CoreI2C channel. It enables the
CoreI2C slave to receive data when it is the target of an I2C read, write, or
write-read transaction.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_enable_slave$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_enable_slave$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_enable_slave$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_enable_slave$description$example$Example1" data-type="code" data-name="Example">


```
    // Enable I2C slave
    I2C_enable_slave( &g_i2c_inst );
```

</div>


 -------------------------------- 
<a name="i2cdisableslave"></a>
## I2C_disable_slave
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_disable_slave$prototype" data-type="code">

    void
    I2C_disable_slave
    (
        i2c_instance_t * this_i2c
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_disable_slave$description" data-type="text">

This function disables the slave mode operation for a CoreI2C channel. It stops
the CoreI2C slave that acknowledges the I2C read, write, or write-read
transactions targeted at it.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_disable_slave$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_disable_slave$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_disable_slave$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_disable_slave$description$example$Example1" data-type="code" data-name="Example">


```
    // Disable I2C slave
    I2C_disable_slave( &g_i2c_inst );
```

</div>


 -------------------------------- 
<a name="i2csetslavesecondaddr"></a>
## I2C_set_slave_second_addr
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_set_slave_second_addr$prototype" data-type="code">

    void
    I2C_set_slave_second_addr
    (
        i2c_instance_t * this_i2c,
        uint8_t second_slave_addr
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_set_slave_second_addr$description" data-type="text">

The I2C_set_slave_second_addr() function sets the secondary slave address for a
CoreI2C slave device. This is an additional slave address required in certain
applications, for example, to enable fail-safe operation in a system. As the
CoreI2C device supports 7-bit addressing, the highest value assigned to second
slave address is 127 (0x7F).

Note: This function does not support CoreI2C hardware configured with a fixed
second slave address. The current implementation of the ADDR1[0] register bit
makes it difficult for the driver to support both programmable and fixed second
slave address, so we choose to support programmable only.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_set_slave_second_addr$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

#### second_slave_addr
<div id="Functions$I2C_set_slave_second_addr$description$parameters$second_slave_addr" data-type="text" data-name="second_slave_addr">

The second_slave_addr parameter is the secondary slave address of the I2C
device.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_set_slave_second_addr$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_set_slave_second_addr$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_set_slave_second_addr$description$example$Example1" data-type="code" data-name="Example">


```
    #define COREI2C_BASE_ADDR  0xC0000000u
    #define SLAVE_SER_ADDR     0x10u
    #define SECOND_SLAVE_ADDR  0x20u

    i2c_instance_t g_i2c_inst;
    void main( void )
    {
        // Initialize the CoreI2C driver with its base address, primary I2C
        // serial address and serial clock divider.
        I2C_init( &g_i2c_inst, COREI2C_BASE_ADDR, SLAVE_SER_ADDR, 
                  I2C_PCLK_DIV_256 );
        I2C_set_slave_second_addr( &g_i2c_inst, SECOND_SLAVE_ADDR );
    }
```

</div>


 -------------------------------- 
<a name="i2cdisableslavesecondaddr"></a>
## I2C_disable_slave_second_addr
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_disable_slave_second_addr$prototype" data-type="code">

    void
    I2C_disable_slave_second_addr
    (
        i2c_instance_t * this_i2c
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_disable_slave_second_addr$description" data-type="text">

The I2C_disable_slave_second_addr() function disables the secondary slave
address of the CoreI2C slave device.  
Note: This version of the driver only
supports CoreI2C hardware configured with a programmable second slave address.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_disable_slave_second_addr$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_disable_slave_second_addr$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_disable_slave_second_addr$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_disable_slave_second_addr$description$example$Example1" data-type="code" data-name="Example">


```
    i2c_instance_t g_i2c_inst;
    I2C_disable_slave_second_addr( &g_i2c_inst);
```

</div>


 -------------------------------- 
<a name="i2csetgca"></a>
## I2C_set_gca
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_set_gca$prototype" data-type="code">

    void
    I2C_set_gca
    (
        i2c_instance_t * this_i2c
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_set_gca$description" data-type="text">

The I2C_set_gca() function is used to set the general call acknowledgement bit
of a CoreI2C slave device. This allows all channels of the CoreI2C slave device
to respond to a general call or broadcast message from an I2C master.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_set_gca$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_set_gca$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_set_gca$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_set_gca$description$example$Example1" data-type="code" data-name="Example">


```
    i2c_instance_t g_i2c_inst;

    // Enable recognition of the General Call Address
    I2C_set_gca( &g_i2c_inst ); 
```

</div>


 -------------------------------- 
<a name="i2ccleargca"></a>
## I2C_clear_gca
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_clear_gca$prototype" data-type="code">

    void
    I2C_clear_gca
    (
        i2c_instance_t * this_i2c
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_clear_gca$description" data-type="text">

The I2C_clear_gca() function is used to clear the general call acknowledgement
bit of a CoreI2C slave device. This will stop all channels of the I2C slave
device responding to any general call or broadcast message from the master.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_clear_gca$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_clear_gca$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_clear_gca$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_clear_gca$description$example$Example1" data-type="code" data-name="Example">


```
    i2c_instance_t g_i2c_inst;

    // Disable recognition of the General Call Address
    I2C_clear_gca( &g_i2c_inst );
```

</div>


 -------------------------------- 
<a name="i2csmbusinit"></a>
## I2C_smbus_init
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_smbus_init$prototype" data-type="code">

    void
    I2C_smbus_init
    (
        i2c_instance_t * this_i2c
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_smbus_init$description" data-type="text">

The I2C_smbus_init() function enables SMBus timeouts and status logic for a
CoreI2C channel.  
Note: This and any of the other SMBus related functionality
will only have an effect if the CoreI2C was instantiated with the Generate SMBus
Logic option checked.  
Note: If the CoreI2C was instantiated with the Generate
IPMI Logic option checked this function then enables the IPMI 3mS SCL low
timeout but none of the other SMBus functions will have any effect.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_smbus_init$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_smbus_init$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_smbus_init$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_smbus_init$description$example$Example1" data-type="code" data-name="Example">


```
    #define COREI2C_BASE_ADDR  0xC0000000u
    #define SLAVE_SER_ADDR     0x10u

    i2c_instance_t g_i2c_inst;

    void system_init( void )
    {
         I2C_init( &g_i2c_inst, COREI2C_BASE_ADDR, SLAVE_SER_ADDR,
                   I2C_PCLK_DIV_256 );
         
         // Initialize SMBus feature
         I2C_smbus_init( &g_i2c_inst);
    }
```

</div>


 -------------------------------- 
<a name="i2cenablesmbusirq"></a>
## I2C_enable_smbus_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_enable_smbus_irq$prototype" data-type="code">

    void
    I2C_enable_smbus_irq
    (
        i2c_instance_t * this_i2c,
        uint8_t irq_type
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_enable_smbus_irq$description" data-type="text">

The I2C_enable_smbus_irq() function is used to enable the CoreI2C channel’s
SMBSUS and SMBALERT SMBus interrupts.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_enable_smbus_irq$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

#### irq_type
<div id="Functions$I2C_enable_smbus_irq$description$parameters$irq_type" data-type="text" data-name="irq_type">

The irq_type specify the SMBUS interrupt(s) which will be enabled. The two
possible interrupts are:

  - I2C_SMBALERT_IRQ  


  - I2C_SMBSUS_IRQ  

To enable both interrupts in one call, use I2C_SMBALERT_IRQ | 
I2C_SMBSUS_IRQ.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_enable_smbus_irq$description$return" data-type="text">

None


</div>

##### Example
<div id="Functions$I2C_enable_smbus_irq$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_enable_smbus_irq$description$example$Example1" data-type="code" data-name="Example">


```
     #define COREI2C_BASE_ADDR  0xC0000000u
     #define SLAVE_SER_ADDR     0x10u

     i2c_instance_t g_i2c_inst;

     void main( void )
     {
         I2C_init( &g_i2c_inst, COREI2C_BASE_ADDR, SLAVE_SER_ADDR,
                   I2C_PCLK_DIV_256 );
         
         // Initialize SMBus feature
         I2C_smbus_init( &g_i2c_inst );
         
         // Enable both I2C_SMBALERT_IRQ & I2C_SMBSUS_IRQ interrupts
         I2C_enable_smbus_irq( &g_i2c_inst,
                               (uint8_t)(I2C_SMBALERT_IRQ | I2C_SMBSUS_IRQ) );
    }
```

</div>


 -------------------------------- 
<a name="i2cdisablesmbusirq"></a>
## I2C_disable_smbus_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_disable_smbus_irq$prototype" data-type="code">

    void
    I2C_disable_smbus_irq
    (
        i2c_instance_t * this_i2c,
        uint8_t irq_type
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_disable_smbus_irq$description" data-type="text">

The I2C_disable_smbus_irq() function disable the CoreI2C channel’s SMBSUS and
SMBALERT SMBus interrupts.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_disable_smbus_irq$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

#### irq_type
<div id="Functions$I2C_disable_smbus_irq$description$parameters$irq_type" data-type="text" data-name="irq_type">

The irq_type specifies the SMBUS interrupt(s) which are disabled. The two
possible interrupts are: 
- I2C_SMBALERT_IRQ 
- I2C_SMBSUS_IRQ  

To disable both
ints in one call, use I2C_SMBALERT_IRQ | I2C_SMBSUS_IRQ.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_disable_smbus_irq$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_disable_smbus_irq$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_disable_smbus_irq$description$example$Example1" data-type="code" data-name="Example">


```
    #define COREI2C_BASE_ADDR  0xC0000000u
    #define SLAVE_SER_ADDR     0x10u

    i2c_instance_t g_i2c_inst;

    void main( void )
    {
        I2C_init( &g_i2c_inst, COREI2C_BASE_ADDR, SLAVE_SER_ADDR,
                  I2C_PCLK_DIV_256 );
        
        // Initialize SMBus feature
        I2C_smbus_init( &g_i2c_inst );
        
        // Enable both SMBALERT & SMBSUS interrupts
        I2C_enable_smbus_irq( &g_i2c_inst,
                              (uint8_t)(I2C_SMBALERT_IRQ | I2C_SMBSUS_IRQ));
        
        ...        

        // Disable the SMBALERT interrupt
        I2C_disable_smbus_irq( &g_i2c_inst, I2C_SMBALERT_IRQ );
    }
```

</div>


 -------------------------------- 
<a name="i2csuspendsmbusslave"></a>
## I2C_suspend_smbus_slave
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_suspend_smbus_slave$prototype" data-type="code">

    void
    I2C_suspend_smbus_slave
    (
        i2c_instance_t * this_i2c
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_suspend_smbus_slave$description" data-type="text">

The I2C_suspend_smbus_slave() function forces any SMBUS slave devices connected
to a CoreI2C channel into Power-Down or Suspend mode by asserting the channel's
SMBSUS signal. The CoreI2C channel is the SMBus master in this case.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_suspend_smbus_slave$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_suspend_smbus_slave$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_suspend_smbus_slave$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_suspend_smbus_slave$description$example$Example1" data-type="code" data-name="Example">


```
    #define COREI2C_BASE_ADDR  0xC0000000u
    #define SLAVE_SER_ADDR     0x10u

    i2c_instance_t g_i2c_inst;

    void main( void )
    {
        I2C_init( &g_i2c_inst, COREI2C_BASE_ADDR, SLAVE_SER_ADDR, 
                  I2C_PCLK_DIV_256 );

        // Initialize SMBus feature
        I2C_smbus_init( &g_i2c_inst );

        // suspend SMBus slaves
        I2C_suspend_smbus_slave( &g_i2c_inst );

        ...

        // Re-enable SMBus slaves
        I2C_resume_smbus_slave( &g_i2c_inst );
    }
```

</div>


 -------------------------------- 
<a name="i2cresumesmbusslave"></a>
## I2C_resume_smbus_slave
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_resume_smbus_slave$prototype" data-type="code">

    void
    I2C_resume_smbus_slave
    (
        i2c_instance_t * this_i2c
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_resume_smbus_slave$description" data-type="text">

The I2C_resume_smbus_slave() function de-asserts the CoreI2C channel's SMBSUS
signal to take any connected slave devices out of the Suspend mode. The CoreI2C
channel is the SMBus master in this case.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_resume_smbus_slave$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_resume_smbus_slave$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_resume_smbus_slave$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_resume_smbus_slave$description$example$Example1" data-type="code" data-name="Example">


```
    #define COREI2C_BASE_ADDR  0xC0000000u
    #define SLAVE_SER_ADDR     0x10u

    i2c_instance_t g_i2c_inst;

    void main( void )
    {
        I2C_init( &g_i2c_inst, COREI2C_BASE_ADDR, SLAVE_SER_ADDR, 
                  I2C_PCLK_DIV_256 );

        // Initialize SMBus feature
        I2C_smbus_init( &g_i2c_inst );

        // suspend SMBus slaves
        I2C_suspend_smbus_slave( &g_i2c_inst );

        ...

        // Re-enable SMBus slaves
        I2C_resume_smbus_slave( &g_i2c_inst );
    }
```

</div>


 -------------------------------- 
<a name="i2cresetsmbus"></a>
## I2C_reset_smbus
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_reset_smbus$prototype" data-type="code">

    void
    I2C_reset_smbus
    (
        i2c_instance_t * this_i2c
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_reset_smbus$description" data-type="text">

The I2C_reset_smbus() function resets the CoreI2C channel's SMBus connection by
forcing SCLK low for 35 mS. The reset that automatically cleares after 35 ms
gets elapsed. The CoreI2C channel is the SMBus master in this case.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_reset_smbus$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_reset_smbus$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_reset_smbus$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_reset_smbus$description$example$Example1" data-type="code" data-name="Example">


```
    #define COREI2C_BASE_ADDR  0xC0000000u
    #define SLAVE_SER_ADDR     0x10u

    i2c_instance_t g_i2c_inst;

    void main( void )
    {
        I2C_init( &g_i2c_inst, COREI2C_BASE_ADDR, SLAVE_SER_ADDR, 
                  I2C_PCLK_DIV_256 );

        // Initialize SMBus feature
        I2C_smbus_init( &g_i2c_inst );

        // Make sure the SMBus channel is in a known state by resetting it
        I2C_reset_smbus( &g_i2c_inst ); 
    }
```

</div>


 -------------------------------- 
<a name="i2csetsmbusalert"></a>
## I2C_set_smbus_alert
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_set_smbus_alert$prototype" data-type="code">

    void
    I2C_set_smbus_alert
    (
        i2c_instance_t * this_i2c
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_set_smbus_alert$description" data-type="text">

The I2C_set_smbus_alert() function is used to force master communication with an
I2C slave device by asserting the CoreI2C channel's SMBALERT signal. The CoreI2C
channel is the SMBus slave in this case.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_set_smbus_alert$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_set_smbus_alert$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_set_smbus_alert$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_set_smbus_alert$description$example$Example1" data-type="code" data-name="Example">


```
    #define COREI2C_BASE_ADDR  0xC0000000u
    #define SLAVE_SER_ADDR     0x10u

    i2c_instance_t g_i2c_inst;

    void main( void )
    {
        I2C_init( &g_i2c_inst, COREI2C_BASE_ADDR, SLAVE_SER_ADDR, 
                  I2C_PCLK_DIV_256 );

        // Initialize SMBus feature
        I2C_smbus_init( &g_i2c_inst );

        // Get the SMBus masters attention
        I2C_set_smbus_alert( &g_i2c_inst );

        ...

        // Once we are happy, drop the alert
        I2C_clear_smbus_alert( &g_i2c_inst );
    }
```

</div>


 -------------------------------- 
<a name="i2cclearsmbusalert"></a>
## I2C_clear_smbus_alert
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_clear_smbus_alert$prototype" data-type="code">

    void
    I2C_clear_smbus_alert
    (
        i2c_instance_t * this_i2c
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_clear_smbus_alert$description" data-type="text">

The I2C_clear_smbus_alert() function is used to de-assert the CoreI2C channel's
SMBALERT signal once a slave device gets a response from the master. The CoreI2C
channel is the SMBus slave in this case.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_clear_smbus_alert$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_clear_smbus_alert$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_clear_smbus_alert$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_clear_smbus_alert$description$example$Example1" data-type="code" data-name="Example">


```
    #define COREI2C_BASE_ADDR  0xC0000000u
    #define SLAVE_SER_ADDR     0x10u

    i2c_instance_t g_i2c_inst;

    void main( void )
    {
        I2C_init( &g_i2c_inst, COREI2C_BASE_ADDR, SLAVE_SER_ADDR, 
                  I2C_PCLK_DIV_256 );

        // Initialize SMBus feature
        I2C_smbus_init( &g_i2c_inst );

        // Get the SMBus masters attention
        I2C_set_smbus_alert( &g_i2c_inst );

        ...

        // Once we are happy, drop the alert
        I2C_clear_smbus_alert( &g_i2c_inst );
    }
```

</div>


 -------------------------------- 
<a name="i2cgetirqstatus"></a>
## I2C_get_irq_status
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_get_irq_status$prototype" data-type="code">

    uint8_t
    I2C_get_irq_status
    (
        i2c_instance_t * this_i2c
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_get_irq_status$description" data-type="text">

The I2C_get_irq_status function returns information about which interrupts are
currently pending in a CoreI2C channel. The interrupts supported by CoreI2C are:
- SMBUSALERT 
- SMBSUS 
- INTR  

The macros I2C_NO_IRQ, I2C_SMBALERT_IRQ,
I2C_SMBSUS_IRQ, and I2C_INTR_IRQ are provided to use with this function.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_get_irq_status$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_get_irq_status$description$return" data-type="text">

This function returns the status of the CoreI2C channel's interrupts as a single
byte bitmap where a bit is set to indicate a pending interrupt. The following
are the bit positions associated with each interrupt type:  
Bit 0 -
SMBUS_ALERT_IRQ  
Bit 1 - SMBSUS_IRQ  
Bit 2 - INTR_IRQ  
It returns 0, if there
are no pending interrupts.


</div>

##### Example
<div id="Functions$I2C_get_irq_status$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_get_irq_status$description$example$Example1" data-type="code" data-name="Example">


```
    #define COREI2C_BASE_ADDR  0xC0000000u
    #define SLAVE_SER_ADDR     0x10u

    i2c_instance_t g_i2c_inst;

    void main( void )
    {
         uint8_t irq_to_enable = I2C_SMBALERT_IRQ | I2C_SMBSUS_IRQ;
         uint8_t pending_irq = 0u;
         
         I2C_init( &g_i2c_inst, COREI2C_BASE_ADDR, SLAVE_SER_ADDR,
                   I2C_PCLK_DIV_256 );

         // Initialize SMBus feature
         I2C_smbus_init( &g_i2c_inst );

         // Enable both I2C_SMBALERT_IRQ & I2C_SMBSUS_IRQ irq
         I2C_enable_smbus_irq( &g_i2c_inst, irq_to_enable );

         // Get I2C IRQ type
         pending_irq = I2C_get_irq_status( &g_i2c_inst );

         // Let's assume, in system, INTR and SMBALERT IRQ is pending.
         // So pending_irq will return status of both the IRQs

         if( pending_irq & I2C_SMBALERT_IRQ )
         {
            // if true, it means SMBALERT_IRQ is there in pending IRQ list
         }
         if( pending_irq & I2C_INTR_IRQ )
         {
            // if true, it means I2C_INTR_IRQ is there in pending IRQ list
         }
    }
```

</div>


 -------------------------------- 
<a name="i2csetuserdata"></a>
## I2C_set_user_data
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_set_user_data$prototype" data-type="code">

    void
    I2C_set_user_data
    (
        i2c_instance_t * this_i2c,
        void * p_user_data
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_set_user_data$description" data-type="text">

The I2C_set_user_data() function allows the association of a block of
application specific data with a CoreI2C channel. The composition of the data
block is an application matter and the driver simply provides the means for the
application to set and retrieve the pointer. For example, this is used to
provide additional channel specific information to the slave write handler.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_set_user_data$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

#### p_user_data
<div id="Functions$I2C_set_user_data$description$parameters$p_user_data" data-type="text" data-name="p_user_data">

The p_user_data parameter is a pointer to the user specific data block for this
channel. It is defined as void * as the driver does not know the actual type of
data being pointed to and simply stores the pointer for later retrieval by the
application.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_set_user_data$description$return" data-type="text">

None.


</div>

##### Example
<div id="Functions$I2C_set_user_data$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_set_user_data$description$example$Example1" data-type="code" data-name="Example">


```
    #define COREI2C_BASE_ADDR  0xC0000000u
    #define SLAVE_SER_ADDR     0x10u

    i2c_instance_t g_i2c_inst;
    app_data_t channel_xdata;

    void main( void )
    {
        app_data_t *p_xdata;

        I2C_init( &g_i2c_inst, COREI2C_BASE_ADDR, SLAVE_SER_ADDR, 
                  I2C_PCLK_DIV_256 );

        // Store location of user data in instance structure
        I2C_set_user_data( &g_i2c_inst, (void *)&channel_xdata );

        ...

        // Retrieve location of user data and do some work on it
        p_xdata = (app_data_t *)I2C_get_user_data( &g_i2c_inst );
        if( NULL != p_xdata )
        {
            p_xdata->foo = 123;
        }
    }
```

</div>


 -------------------------------- 
<a name="i2cgetuserdata"></a>
## I2C_get_user_data
<a name="prototype"></a>
### Prototype 

<div id="Functions$I2C_get_user_data$prototype" data-type="code">

    void *
    I2C_get_user_data
    (
        i2c_instance_t * this_i2c
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$I2C_get_user_data$description" data-type="text">

The I2C_get_user_data() function is used to allows the retrieval of the address
of a block of application specific data associated with a CoreI2C channel. The
composition of the data block is an application matter and the driver simply
provides the means for the application to set and retrieve the pointer. For
example, this is used to provide additional channel specific information to the
slave write handler.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_i2c
<div id="Functions$I2C_get_user_data$description$parameters$this_i2c" data-type="text" data-name="this_i2c">

The this_i2c parameter is a pointer to the i2c_instance_t data structure holding
all data related to a specific CoreI2C channel. For example, if only one channel
is initialized, this data structure holds the information of channel 0 of the
instantiated CoreI2C hardware.


</div>

<a name="return"></a>
### Return
<div id="Functions$I2C_get_user_data$description$return" data-type="text">

This function returns a pointer to the user specific data block for this
channel. It is defined as void * as the driver does not know the actual type of
data being pointed. If no user data has been registered for this channel a NULL
pointer is returned.


</div>

##### Example
<div id="Functions$I2C_get_user_data$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$I2C_get_user_data$description$example$Example1" data-type="code" data-name="Example">


```
    #define COREI2C_BASE_ADDR  0xC0000000u
    #define SLAVE_SER_ADDR     0x10u

    i2c_instance_t g_i2c_inst;
    app_data_t channel_xdata;

    void main( void )
    {
        app_data_t *p_xdata;

        I2C_init( &g_i2c_inst, COREI2C_BASE_ADDR, SLAVE_SER_ADDR,
                  I2C_PCLK_DIV_256 );

        // Store location of user data in instance structure
        I2C_set_user_data( &g_i2c_inst, (void *)&channel_xdata );

        ...
        
        // Retrieve location of user data and do some work on it
        p_xdata = (app_data_t *)I2C_get_user_data( &g_i2c_inst );
        if( NULL != p_xdata )
        {
            p_xdata->foo = 123;
        }
    }
```

</div>


 -------------------------------- 
