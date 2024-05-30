<html>
 
 ------------------------------------ 

# CoreQSPI Bare Metal Driver
 
-----------------------------------------
## Table of Contents 
  - [Introduction](#introduction)

     - [Features](#features)

  - [Theory of Operation](#theory-of-operation)

  - [Driver Initialization and Configuration](#driver-initialization-and-configuration)

  - [SPI Master Block Transfer Control](#spi-master-block-transfer-control)

     - [Polled Block Transfer](#polled-block-transfer)

     - [Interrupt Driven Block Transfer](#interrupt-driven-block-transfer)

  - [QSPI Status](#qspi-status)

  - [Direct Access](#direct-access)

- [Types](#types)
  - [qspi_io_format](#qspiioformat)
  - [qspi_protocol_mode](#qspiprotocolmode)
  - [qspi_clk_div](#qspiclkdiv)
  - [qspi_status_handler_t](#qspistatushandlert)
  - [qspi_config_t](#qspiconfigt)
  - [qspi_instance_t](#qspiinstancet)

- [Constants](#constants)
  - [SDI Pin Sampling](#sdi-pin-sampling)
  - [Generic constant definitions](#generic-constant-definitions)

- [Functions](#functions)
  - [QSPI_init](#qspiinit)
  - [QSPI_disable](#qspidisable)
  - [QSPI_configure](#qspiconfigure)
  - [QSPI_get_config](#qspigetconfig)
  - [QSPI_polled_transfer_block](#qspipolledtransferblock)
  - [QSPI_irq_transfer_block](#qspiirqtransferblock)
  - [QSPI_set_status_handler](#qspisetstatushandler)
  - [QSPI_read_direct_access_reg](#qspireaddirectaccessreg)
  - [QSPI_write_direct_access_reg](#qspiwritedirectaccessreg)
  - [QSPI_read_status](#qspireadstatus)
  - [qspi_isr](#qspiisr)

<div id="TitlePage" data-type="text">

# Introduction
This CoreQSPI provides AHB system Interface and SPI interface to connect with
the SPI memory devices. The control register is used to configure the IP in
different modes and the FIFO is used to buffer the data across clock domains.

This driver provides a set of functions for configuring and controlling CoreQSPI
as part of a bare metal system, where no operating system is available. This
driver can be adapted as part of an operating system, but the implementation of
the adaptation layer between the driver and the operating system's driver model
is outside the scope of this user guide.

This driver is not compatible with the CoreQSPI versions below v2.1.

## Features
  - Master operation
  - Motorola SPI supported
  - Slave select operation in idle cycles configurable
  - Supports Extended SPI operation (1, 2, and 4-bit)
  - Supports QSPI operation (4-bit operation)
  - Supports BSPI operation (2-bit operation)
  - Support XIP (execute in place)
  - Supports three or four-byte SPI address

# Theory of Operation
The CoreQSPI driver functions are grouped into the following categories.

  - Initialization and configuration functions
  - Polled transmit and receive functions
  - Interrupt-driven transmit and receive functions

# Driver Initialization and Configuration
The CoreQSPI supports the following operations:

  - Normal SPI operations (1-bit)
  - Dual SPI operations (2-bit)
  - Quad SPI operations (4-bit)

The CoreQSPI driver provides the QSPI_init() function to initialize the CoreQSPI
hardware block. This initialization function must be called before calling any
other CoreQSPI driver functions.

The CoreQSPI driver provides the QSPI_config() function to configure QSPI with
the desired configuration values. It also provides the QSPI_get_config()
function to read back the current configuration of QSPI use QSPI_get_config()
function to retrieve the current configurations and then overwrite them with
application-specific values, such as SPI mode, SPI clock rate, SDI sampling,
QSPI operation, XIP mode, and XIP address bits. All these configuration options
are explained in detail in the API function description of the respective
function.

# SPI Master Block Transfer Control
Once the driver is initialized and configured, data transmission and reception
are achieved by configuring it to the desired value. CoreQSPI is designed to
specifically work with SPI flash memories. It supports a single, active-low
slave-select output. Block transfers are accomplished in the following ways:

  - Polled block transfer
  - Interrupt-driven block transfer

## Polled Block Transfer
The QSPI_polled_transfer_block() function accomplishes data transfers where no
interrupt is used. The QSPI_polled_transfer_block() function polls the status
register to know the current status of the on-going transfer. This is a blocking
function. A CoreQSPI block transfer always has some amount of data to be
transmitted (at least one command byte), but receiving useful data from the
target memory device is optional. So, if the scheduled block transfer is only
transferring data and not receiving any data, then QSPI_polled_transfer_block()
function exits after transmitting the required bytes. If data needs to be
received from the target memory device during a particular transfer, the
QSPI_polled_transfer_block() function terminates once the expected data is
received from the target memory device.

## Interrupt Driven Block Transfer
This block transfer is accomplished using interrupts instead of polling the
status register. The following functions are provided to support interrupt-
driven block transfers:

  - QSPI_irq_transfer_block()
  - QSPI_set_status_handler()

The QSPI_set_status_handler() function must be used to set a status handler
call-back function with the driver. The QSPI_set_status_handler() function is
called back by the driver at two different stages of the transfer. At the first
stage, it is called when the required number of bytes are transmitted. At the
second stage, if there is data to be received from the target memory device, it
is invoked again once the desired data is received. An appropriate status value
is passed by the driver as a parameter of this call-back function so that the
application infers that an event has occurred.

# QSPI Status
The QSPI_read_status() function reads the current status of CoreQSPI. The
QSPI_read_status() function is typically used to know the status of an ongoing
transfer. The QSPI_read_status() function returns the status register value and
is called at any time after CoreQSPI is initialized and configured.

# Direct Access
CoreQSPI allows direct access to the QSPI interface pins to support access to
non-standard SPI devices through direct CPU control.

This driver provides the following functions to read and write the direct access
register of CoreQSPI:

  - QSPI_read_direct_access_reg()
  - QSPI_write_direct_access_reg()

Using these functions, you generate any sequence of binary transitions on the
QSPI pins that might be needed to communicate with the non-standard target
devices.

</div>


# Types

 ---------------- 
<a name="qspiioformat"></a>
## qspi_io_format
### Prototype 

<div id="Types$qspi_io_format$prototype" data-type="code">

 ``` 
    typedef enum {
        QSPI_NORMAL = 0u,
        QSPI_DUAL_EX_RO = 2u,
        QSPI_QUAD_EX_RO = 3u,
        QSPI_DUAL_EX_RW = 4u,
        QSPI_QUAD_EX_RW = 5u,
        QSPI_DUAL_FULL = 6u,
        QSPI_QUAD_FULL = 7u
    } qspi_io_format;


 ``` 


</div>

### Description 

<div id="Types$qspi_io_format$description" data-type="text">

The following table shows that these values are used to program the io_format
parameter of the configuration structure of this driver.


```
    qspi_config.io_format = QSPI_QUAD_FULL;
    QSPI_configure(&qspi_config);
```
| Value    | Description     | 
| -----|-----|
| QSPI_NORMAL    | 1-bit Normal SPI operations     | 
|   | (single DQ0 TX and single DQ1 RX lines)     | 
|   |   | 
| QSPI_DUAL_EX_RO    | 2-bit SPI operations     | 
|   | (command and address bytes on DQ0 only. data on DQ[1:0])     | 
|   |   | 
| QSPI_QUAD_EX_RO    | 4-bit SPI operations     | 
|   | (command and address bytes on DQ0 only. data on DQ[3:0])     | 
|   |   | 
| QSPI_DUAL_EX_RW    | 2-bit SPI operations     | 
|   | (command byte on DQ0 only. Address and data on DQ[1:0])     | 
|   |   | 
| QSPI_QUAD_EX_RW    | 4-bit SPI operations     | 
|   | (command byte on DQ0 only. Address and data on DQ[3:0])     | 
|   |   | 
| QSPI_DUAL_FULL    | 2-bit SPI operations     | 
|   | (command, address, and data on DQ[1:0])     | 
|   |   | 
| QSPI_QUAD_FULL    | 4-bit SPI operations     | 
|   | (command, address, and data on DQ[3:0])     | 
|   |   | 


</div>


 --------------------------- 
<a name="qspiprotocolmode"></a>
## qspi_protocol_mode
### Prototype 

<div id="Types$qspi_protocol_mode$prototype" data-type="code">

 ``` 
    typedef enum {
        QSPI_MODE0 = 0x0u,
        QSPI_MODE3 = 0x1u
    } qspi_protocol_mode;


 ``` 


</div>

### Description 

<div id="Types$qspi_protocol_mode$description" data-type="text">

These values are used to program the spi_mode parameter of the configuration
structure of this driver. For example:


```
    qspi_config.spi_mode = QSPI_MODE0;
    QSPI_configure(&qspi_config);
```
| Value    | Description     | 
| -----|-----|
| QSPI_MODE0    | Set clock IDLE to low (0)     | 
| QSPI_MODE0    | Set clock IDLE to high (1)    | 


</div>


 --------------------------- 
<a name="qspiclkdiv"></a>
## qspi_clk_div
### Prototype 

<div id="Types$qspi_clk_div$prototype" data-type="code">

 ``` 
    typedef enum {
        QSPI_CLK_DIV_2 = 0x1u,
        QSPI_CLK_DIV_4 = 0x2u,
        QSPI_CLK_DIV_6 = 0x3u,
        QSPI_CLK_DIV_8 = 0x4u,
        QSPI_CLK_DIV_10 = 0x5u,
        QSPI_CLK_DIV_12 = 0x6u,
        QSPI_CLK_DIV_14 = 0x7u,
        QSPI_CLK_DIV_16 = 0x8u,
        QSPI_CLK_DIV_18 = 0x9u,
        QSPI_CLK_DIV_20 = 0xAu,
        QSPI_CLK_DIV_22 = 0xBu,
        QSPI_CLK_DIV_24 = 0xCu,
        QSPI_CLK_DIV_26 = 0xDu,
        QSPI_CLK_DIV_28 = 0xEu,
        QSPI_CLK_DIV_30 = 0xFu
    } qspi_clk_div;


 ``` 


</div>

### Description 

<div id="Types$qspi_clk_div$description" data-type="text">

These values are used to program the spi_mode parameter of the configuration
structure of this driver. For example:


```
    qspi_config.clk_div =  QSPI_CLK_DIV_2;
    QSPI_configure(&qspi_config);
```
| Value    | Description     | 
| -----|-----|
| QSPI_CLK_DIV_2    | Set SPI clock to HCLK/2     | 
| QSPI_CLK_DIV_4    | Set SPI clock to HCLK/4     | 
| QSPI_CLK_DIV_6    | Set SPI clock to HCLK/6     | 
| QSPI_CLK_DIV_8    | Set SPI clock to HCLK/8     | 
| QSPI_CLK_DIV_10    | Set SPI clock to HCLK/10     | 
| QSPI_CLK_DIV_12    | Set SPI clock to HCLK/12     | 
| QSPI_CLK_DIV_14    | Set SPI clock to HCLK/14     | 
| QSPI_CLK_DIV_16    | Set SPI clock to HCLK/16     | 
| QSPI_CLK_DIV_18    | Set SPI clock to HCLK/18     | 
| QSPI_CLK_DIV_20    | Set SPI clock to HCLK/20     | 
| QSPI_CLK_DIV_22    | Set SPI clock to HCLK/22     | 
| QSPI_CLK_DIV_24    | Set SPI clock to HCLK/24     | 
| QSPI_CLK_DIV_26    | Set SPI clock to HCLK/26     | 
| QSPI_CLK_DIV_28    | Set SPI clock to HCLK/28     | 
| QSPI_CLK_DIV_30    | Set SPI clock to HCLK/30    | 


</div>


 --------------------------- 
<a name="qspistatushandlert"></a>
## qspi_status_handler_t
### Prototype 

<div id="Types$qspi_status_handler_t$prototype" data-type="code">

`typedef void(* qspi_status_handler_t) (uint32_t status)`

</div>

### Description 

<div id="Types$qspi_status_handler_t$description" data-type="text">

This prototype defines the function prototype that must be followed by CoreQSPI
status handler functions. This function is registered with the CoreQSPI driver
by calling the QSPI_set_status_handler() function.

Declaring and Implementing Status Handler Function: Slave frame receive handler
functions should follow the following prototype: void
transfer_status_handler(uint32_t status);

The actual name of the status handler not important. You can use any name of
your choice. The status parameter contains a value indicating which of the TX-
DONE, or RX-DONE event has caused the interrupt.


</div>


 --------------------------- 
<a name="qspiconfigt"></a>
## qspi_config_t
### Prototype 

<div id="Types$qspi_config_t$prototype" data-type="code">

``` 
    typedef struct qspi_config {
        uint8_t xip; 
        uint8_t xip_addr; 
        qspi_protocol_mode spi_mode; 
        qspi_clk_div clk_div; 
        qspi_io_format io_format; 
        uint8_t sample; 
    } qspi_config_t ;
  ``` 


</div>

### Description 

<div id="Types$qspi_config_t$description" data-type="text">

This is the structure definition for the CoreQSPI configuration instance. It
defines the configuration data that the application exchanges with the driver.

The following parameters are the configuration options for CoreQSPI.

| Parameter    | Description     | 
| -----|-----|
| xip    | Enable or disable XIP mode     | 
| xip_addr    | Number of address bytes used in XIP mode     | 
| spi_mode    | Select either Motorola mode0 or mode3     | 
| clk_div    | HCLK Clock divider for generating SPI clock     | 
| io_format    | QSPI transfer format, extended, dual, quad and so on.     | 
| sample    | Select the event on which the QSPI samples the incoming data    | 


</div>


 --------------------------- 
<a name="qspiinstancet"></a>
## qspi_instance_t
### Prototype 

<div id="Types$qspi_instance_t$prototype" data-type="code">

``` 
    typedef struct __qspi_instance_t {
        addr_t base_address; 
    } qspi_instance_t ;
  ``` 


</div>

### Description 

<div id="Types$qspi_instance_t$description" data-type="text">

There should be one instance of this structure for each instance of CoreQSPI in
your system. The QSPI_init() function initializes this structure. It is used to
identify the various CoreQSPI hardware instances in your system. An initialized
timer instance structure should be passed as first parameter to CoreQSPI driver
functions to identify which CoreQSPI instance should perform the requested
operation. Software using this driver should only need to create one single
instance of this data structure for each hardware QSPI instance in the system.


</div>


 --------------------------- 

# Constants

 ---------------- 
<div id="Constants$QSPI_SAMPLE_POSAGE_SPICLK$description" data-type="text">

<a name="qspisampleposagespiclk"></a>
## SDI Pin Sampling
The following constants can be used as input parameter value to configure the
event on which the SDI pin is sampled.


```
    qspi_config.sample = QSPI_SAMPLE_POSAGE_SPICLK;
    QSPI_configure(&qspi_config);
```
| Constant    | Description     | 
| -----|-----|
| QSPI_SAMPLE_POSAGE_SPICLK    | The SDI pin is sampled at the rising edge off SPI CLOCK     | 
| QSPI_SAMPLE_ACTIVE_SPICLK    | The SDI pin is sampled on the last falling HCLK edge in the SPI clock period     | 
| QSPI_SAMPLE_NEGAGE_SPICLK    | The SDI pin is sampled at the rising HCLK edge as SPICLK falls    | 

</div>

<div id="Constants$QSPI_ENABLE$description" data-type="text">

<a name="qspienable"></a>
## Generic constant definitions
The following constants can be used to configure the CoreQSPI where a zero or
non-zero value such as enable or disable is to be provided as input parameter.


```
    qspi_config.xip = QSPI_DISABLE;
    QSPI_configure(&qspi_config);
```
| Constant    | Description     | 
| -----|-----|
| QSPI_ENABLE    | Enable     | 
| QSPI_DISABLE    | Disable    | 

</div>


# Functions

 ---------------- 
<a name="qspiinit"></a>
## QSPI_init
### Prototype 

<div id="Functions$QSPI_init$prototype" data-type="code">

    void
    QSPI_init
    (
        qspi_instance_t * this_qspi,
        addr_t addr
    );


</div>

### Description

<div id="Functions$QSPI_init$description" data-type="text">

The QSPI_init() function initializes and enables the CoreQSPI. It enables the
QSPI hardware block, and configures it with the default values.

</div>


 --------------------------- 

### Parameters
#### this_qspi
<div id="Functions$QSPI_init$description$parameters$this_qspi" data-type="text" data-name="this_qspi">

It is a pointer to the qspi_instance_t data structure that holds all the data
related to the CoreQSPI instance being initialized. A pointer to this data
structure is used in all subsequent calls to the CoreQSPI driver functions that
operate on this CoreQSPI instance.


</div>

### Return
<div id="Functions$QSPI_init$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$QSPI_init$description$example$Example1" data-type="text" data-name="Example">
This example shows how to initialize the CoreQSPI driver.


</div>

<div id="Functions$QSPI_init$description$example$Example1" data-type="code" data-name="Example">


```
    QSPI_init(QSPI_INSTANCE, base_addr);
```

</div>


 -------------------------------- 
<a name="qspidisable"></a>
## QSPI_disable
### Prototype 

<div id="Functions$QSPI_disable$prototype" data-type="code">

    void
    QSPI_disable
    (
        qspi_instance_t * this_qspi
    );


</div>

### Description

<div id="Functions$QSPI_disable$description" data-type="text">

The QSPI_disable() function disables the CoreQSPI hardware block.

</div>


 --------------------------- 

### Parameters
#### this_qspi
<div id="Functions$QSPI_disable$description$parameters$this_qspi" data-type="text" data-name="this_qspi">

It is a pointer to the qspi_instance_t data structure that holds all the data
related to the CoreQSPI instance being initialized. A pointer to this data
structure is used in all subsequent calls to the CoreQSPI driver functions that
operate on this CoreQSPI instance.


</div>

### Return
<div id="Functions$QSPI_disable$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$QSPI_disable$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$QSPI_disable$description$example$Example1" data-type="code" data-name="Example">


```
    QSPI_disable(QSPI_INSTANCE);
```

</div>


 -------------------------------- 
<a name="qspiconfigure"></a>
## QSPI_configure
### Prototype 

<div id="Functions$QSPI_configure$prototype" data-type="code">

    void
    QSPI_configure
    (
        qspi_instance_t * this_qspi,
        const qspi_config_t * config
    );


</div>

### Description

<div id="Functions$QSPI_configure$description" data-type="text">

The QSPI_configure() function configures the CoreQSPI to the desired
configuration values.

</div>


 --------------------------- 

### Parameters
#### this_qspi
<div id="Functions$QSPI_configure$description$parameters$this_qspi" data-type="text" data-name="this_qspi">

It is a pointer to the qspi_instance_t data structure that holds all the data
related to the CoreQSPI instance being initialized. A pointer to this data
structure is used in all subsequent calls to the CoreQSPI driver functions that
operate on this CoreQSPI instance.


</div>

#### config
<div id="Functions$QSPI_configure$description$parameters$config" data-type="text" data-name="config">

The config parameter is a pointer to the qspi_config_t structure, which provides
new configuration values. See the qspi_config_t section for details.


</div>

### Return
<div id="Functions$QSPI_configure$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$QSPI_configure$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$QSPI_configure$description$example$Example1" data-type="code" data-name="Example">


```
    qspi_config.clk_div =  QSPI_CLK_DIV_8;
    qspi_config.sample = QSPI_SAMPLE_POSAGE_SPICLK;
    qspi_config.spi_mode = QSPI_MODE3;
    qspi_config.xip = QSPI_DISABLE;
    qspi_config.io_format = t_io_format;

    QSPI_configure(QSPI_INSTANCE, &qspi_config);
```

</div>


 -------------------------------- 
<a name="qspigetconfig"></a>
## QSPI_get_config
### Prototype 

<div id="Functions$QSPI_get_config$prototype" data-type="code">

    void
    QSPI_get_config
    (
        qspi_instance_t * this_qspi,
        qspi_config_t * config
    );


</div>

### Description

<div id="Functions$QSPI_get_config$description" data-type="text">

The QSPI_get_config() function reads-back the current configurations of the
CoreQSPI. This function can be used when you want to read the current
configurations, modify the configuration values of your choice and reconfigure
the CoreQSPI hardware, using QSPI_configure() function.

</div>


 --------------------------- 

### Parameters
#### this_qspi
<div id="Functions$QSPI_get_config$description$parameters$this_qspi" data-type="text" data-name="this_qspi">

It is a pointer to the qspi_instance_t data structure that holds all the data
related to the CoreQSPI instance being initialized. A pointer to this data
structure is used in all subsequent calls to the CoreQSPI driver functions that
operate on this CoreQSPI instance.


</div>

#### config
<div id="Functions$QSPI_get_config$description$parameters$config" data-type="text" data-name="config">

The config parameter is a pointer to an qspi_config_t structure, where the
current configuration values of the CoreQSPI are returned.


</div>

### Return
<div id="Functions$QSPI_get_config$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$QSPI_get_config$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$QSPI_get_config$description$example$Example1" data-type="code" data-name="Example">


```
    qspi_config_t beforexip_qspi_config = { 0 };
    QSPI_get_config(QSPI_INSTANCE, &beforexip_qspi_config);
```

</div>


 -------------------------------- 
<a name="qspipolledtransferblock"></a>
## QSPI_polled_transfer_block
### Prototype 

<div id="Functions$QSPI_polled_transfer_block$prototype" data-type="code">

    void
    QSPI_polled_transfer_block
    (
        qspi_instance_t * this_qspi,
        uint8_t num_addr_bytes,
        const void *const tx_buffer,
        uint32_t tx_byte_size,
        const void *const rd_buffer,
        uint32_t rd_byte_size,
        uint8_t num_idle_cycles
    );


</div>

### Description

<div id="Functions$QSPI_polled_transfer_block$description" data-type="text">

The QSPI_polled_transfer_block() function carries out a QSPI transfer with the
target memory device using polling method of data transfer. The QSPI transfer
characteristics are configured every time a new transfer is initiated. This is a
blocking function.

</div>


 --------------------------- 

### Parameters
#### rd_buffer
<div id="Functions$QSPI_polled_transfer_block$description$parameters$rd_buffer" data-type="text" data-name="rd_buffer">

The rd_buffer parameter is the pointer to the buffer where the data returned by
the target memory device is stored.


</div>

#### rd_byte_size
<div id="Functions$QSPI_polled_transfer_block$description$parameters$rd_byte_size" data-type="text" data-name="rd_byte_size">

The rd_byte_size parameter is the exact number of bytes that needs to be
received from the target memory device.


</div>

#### num_idle_cycles
<div id="Functions$QSPI_polled_transfer_block$description$parameters$num_idle_cycles" data-type="text" data-name="num_idle_cycles">

The num_idle_cycles parameter indicates the number of Idle cycles/dummy clock
edges that must be generated after the address bytes are transmitted and before
target memory device starts sending data. This must be correctly set based on
the target memory device and the SPI command being used. This may also vary
based on SPI clock and the way the target memory device is configured.


</div>

### Return
<div id="Functions$QSPI_polled_transfer_block$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$QSPI_polled_transfer_block$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$QSPI_polled_transfer_block$description$example$Example1" data-type="code" data-name="Example">


```
    QSPI_polled_transfer_block(QSPI_INSTANCE, 0, command_buf, 0, rd_buf, 1,0);
```

</div>


 -------------------------------- 
<a name="qspiirqtransferblock"></a>
## QSPI_irq_transfer_block
### Prototype 

<div id="Functions$QSPI_irq_transfer_block$prototype" data-type="code">

    uint8_t
    QSPI_irq_transfer_block
    (
        qspi_instance_t * this_qspi,
        uint8_t num_addr_bytes,
        const void *const tx_buffer,
        uint32_t tx_byte_size,
        const void *const rd_buffer,
        uint32_t rd_byte_size,
        uint8_t num_idle_cycles
    );


</div>

### Description

<div id="Functions$QSPI_irq_transfer_block$description" data-type="text">

The QSPI_irq_transfer_block() function is used to carry out a QSPI transfer with
the target memory device using interrupt method of data transfers. The QSPI
transfer characteristics are configured every time a new transfer is initiated.
This is non-blocking function. You must configure the interrupt handler
function, before calling this function. It will enable the interrupts and start
transmitting as many bytes as requested. You will get an indication when the
actual SPI transmit operation is complete. When The transmit-done interrupt
event occurs and this driver calls-back the interrupt handler function that you
previously provided. If the transfer includes receiving data from the target
memory device, then the receive-available and receive-done interrupts are also
enabled by this function. The data will be received in the interrupt routine.
The interrupt handler provided by you will be called-back again when the
receive-done interrupt event occurs.

</div>


 --------------------------- 

### Parameters
#### rd_buffer
<div id="Functions$QSPI_irq_transfer_block$description$parameters$rd_buffer" data-type="text" data-name="rd_buffer">

The rd_buffer parameter is a pointer to the buffer where the data returned by
the target memory device is to be stored.


</div>

#### rd_byte_size
<div id="Functions$QSPI_irq_transfer_block$description$parameters$rd_byte_size" data-type="text" data-name="rd_byte_size">

The rd_byte_size parameter is the exact number of bytes that needs to be
received from the target memory device.


</div>

#### num_idle_cycles
<div id="Functions$QSPI_irq_transfer_block$description$parameters$num_idle_cycles" data-type="text" data-name="num_idle_cycles">

The num_idle_cycles parameter indicates the number of IDLE/dummy clock edges
that must be generated after the address bytes are transmitted and before target
memory device starts sending data. This must be correctly set based on the
target memory device and the SPI command being used. This may also vary based on
SPI clock and the way the target memory device is configured.


</div>

### Return
<div id="Functions$QSPI_irq_transfer_block$description$return" data-type="text">

This function returns a non-zero value if the CoreQSPI is busy completing the
previous transfer and can not accept a new transfer. A zero return value
indicates successful execution of this function.


</div>

##### Example
<div id="Functions$QSPI_irq_transfer_block$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$QSPI_irq_transfer_block$description$example$Example1" data-type="code" data-name="Example">


```
    QSPI_irq_transfer_block(QSPI_INSTANCE, 0, command_buf, 0, rd_buf, 1,0);
```

</div>


 -------------------------------- 
<a name="qspisetstatushandler"></a>
## QSPI_set_status_handler
### Prototype 

<div id="Functions$QSPI_set_status_handler$prototype" data-type="code">

    void
    QSPI_set_status_handler
    (
        qspi_status_handler_t handler
    );


</div>

### Description

<div id="Functions$QSPI_set_status_handler$description" data-type="text">

The QSPI_set_status_handler() function registers an interrupt handler function
with this driver which is used to indicate the interrupt status back to the
application. This status handler function is called by this driver in two
events. First, when the transmission of required bytes is completed (Transmit-
Done). Second, if data is to be received from the target memory device, then
this function is called again when required data is received (Receive-Done).

</div>


 --------------------------- 

### Parameters
#### this_qspi
<div id="Functions$QSPI_set_status_handler$description$parameters$this_qspi" data-type="text" data-name="this_qspi">

It is a pointer to the qspi_instance_t data structure that holds all the data
related to the CoreQSPI instance being initialized. A pointer to this data
structure is used in all subsequent calls to the CoreQSPI driver functions that
operate on this CoreQSPI instance.


</div>

#### handler
<div id="Functions$QSPI_set_status_handler$description$parameters$handler" data-type="text" data-name="handler">

The handler parameter is the interrupt handler function of the type
qspi_status_handler_t, which needs to be registered.


</div>

### Return
<div id="Functions$QSPI_set_status_handler$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$QSPI_set_status_handler$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$QSPI_set_status_handler$description$example$Example1" data-type="code" data-name="Example">


```
    QSPI_set_status_handler(QSPI_INSTANCE, transfer_status_handler);
```

</div>


 -------------------------------- 
<a name="qspireaddirectaccessreg"></a>
## QSPI_read_direct_access_reg
### Prototype 

<div id="Functions$QSPI_read_direct_access_reg$prototype" data-type="code">

    uint32_t
    QSPI_read_direct_access_reg
    (
        qspi_instance_t * this_qspi
    );


</div>

### Description

<div id="Functions$QSPI_read_direct_access_reg$description" data-type="text">

The QSPI_read_direct_access_reg() reads the current value of the direct access
register(DAR) of the CoreQSPI. DAR allows direct access to the QSPI interface
pins to support access to non-standard SPI devices through direct CPU control.

</div>


 --------------------------- 

### Parameters
#### this_qspi
<div id="Functions$QSPI_read_direct_access_reg$description$parameters$this_qspi" data-type="text" data-name="this_qspi">

It is a pointer to the qspi_instance_t data structure that holds all the data
related to the CoreQSPI instance being initialized. A pointer to this data
structure is used in all subsequent calls to the CoreQSPI driver functions that
operate on this CoreQSPI instance.


</div>

#### This
<div id="Functions$QSPI_read_direct_access_reg$description$parameters$This" data-type="text" data-name="This">

function takes no parameters.


</div>

### Return
<div id="Functions$QSPI_read_direct_access_reg$description$return" data-type="text">

This function returns the current value of the DAR of the CoreQSPI.


</div>


 -------------------------------- 
<a name="qspiwritedirectaccessreg"></a>
## QSPI_write_direct_access_reg
### Prototype 

<div id="Functions$QSPI_write_direct_access_reg$prototype" data-type="code">

    void
    QSPI_write_direct_access_reg
    (
        qspi_instance_t * this_qspi,
        uint32_t value
    );


</div>

### Description

<div id="Functions$QSPI_write_direct_access_reg$description" data-type="text">

The QSPI_write_direct_access_reg() writes value of the DAR of the CoreQSPI. DAR
allows direct access to the QSPI interface pins, to support access to the non-
standard SPI devices through direct CPU control.

</div>


 --------------------------- 

### Parameters
#### this_qspi
<div id="Functions$QSPI_write_direct_access_reg$description$parameters$this_qspi" data-type="text" data-name="this_qspi">

It is a pointer to the qspi_instance_t data structure that holds all the data
related to the CoreQSPI instance being initialized. A pointer to this data
structure is used in all subsequent calls to the CoreQSPI driver functions that
operate on this CoreQSPI instance.


</div>

#### value
<div id="Functions$QSPI_write_direct_access_reg$description$parameters$value" data-type="text" data-name="value">

The value parameter is the value that needs to be set into the direct access
register of the CoreQSPI.


</div>

### Return
<div id="Functions$QSPI_write_direct_access_reg$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="qspireadstatus"></a>
## QSPI_read_status
### Prototype 

<div id="Functions$QSPI_read_status$prototype" data-type="code">

    uint32_t
    QSPI_read_status
    (
        qspi_instance_t * this_qspi
    );


</div>

### Description

<div id="Functions$QSPI_read_status$description" data-type="text">

The QSPI_read_status() function is used to read the status of the CoreQSPI. This
function returns the status register value and can be called any time after the
CoreQSPI is initialized and configured.

</div>


 --------------------------- 

### Parameters
#### this_qspi
<div id="Functions$QSPI_read_status$description$parameters$this_qspi" data-type="text" data-name="this_qspi">

It is a pointer to the qspi_instance_t data structure that holds all the data
related to the CoreQSPI instance being initialized. A pointer to this data
structure is used in all subsequent calls to the CoreQSPI driver functions that
operate on this CoreQSPI instance.


</div>

### Return
<div id="Functions$QSPI_read_status$description$return" data-type="text">

This function returns the the current value of the status register of the
CoreQSPI.


</div>


 -------------------------------- 
<a name="qspiisr"></a>
## qspi_isr
### Prototype 

<div id="Functions$qspi_isr$prototype" data-type="code">

    void
    qspi_isr
    (
        qspi_instance_t * this_qspi
    );


</div>

### Description

<div id="Functions$qspi_isr$description" data-type="text">

Interrupt service routine to handle the QSPI interrupt events. The CoreQSPI
interrupt will appear as an external interrupt on the processor. This function
must be called by the application from the external interrupt handler which
corresponds to the CoreQSPI interrupt.

</div>


 --------------------------- 

### Parameters
#### this_qspi
<div id="Functions$qspi_isr$description$parameters$this_qspi" data-type="text" data-name="this_qspi">

It is a pointer to the qspi_instance_t data structure that holds all the data
related to the CoreQSPI instance being initialized. A pointer to this data
structure is used in all subsequent calls to the CoreQSPI driver functions that
operate on this CoreQSPI instance.


</div>

### Return
<div id="Functions$qspi_isr$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 

</html>
