<html>
 
 ------------------------------------ 

# CoreUARTapb Bare Metal Driver.
 
-----------------------------------------
## Table of Contents 
  - [Introduction](#introduction)

  - [Driver Configuration](#driver-configuration)

  - [Theory of Operation](#theory-of-operation)

- [Types](#types)
  - [UART_instance_t](#uartinstancet)

- [Constants](#constants)
  - [Data Bits Length Defines](#data-bits-length-defines)
  - [Parity Defines](#parity-defines)
  - [Error Status Definitions](#error-status-definitions)

- [Functions](#functions)
  - [UART_init](#uartinit)
  - [UART_send](#uartsend)
  - [UART_fill_tx_fifo](#uartfilltxfifo)
  - [UART_get_rx](#uartgetrx)
  - [UART_polled_tx_string](#uartpolledtxstring)
  - [UART_get_rx_status](#uartgetrxstatus)

<div id="TitlePage" data-type="text">

# Introduction
CoreUARTapb is an implementation of the Universal Asynchronous
Receiver/Transmitter aimed at minimal FPGA tile usage within a Microchip FPGA.
The CoreUARTapb bare metal software driver is designed to be used in systems
with no operating system.

The CoreUARTapb driver provides functions for basic polled transmitting and
receiving operations. It also provide functions that allow the use of the
CoreUARTapb in interrupt-driven mode but leaves the management of interrupts to
the calling application, as interrupt enabling and disabling are not controlled
through the CoreUARTapb registers. The CoreUARTapb driver is provided as C
source code.

# Driver Configuration
Your application software should configure the CoreUARTapb driver by calling the
UART_init() function for each CoreUARTapb instance in the hardware design. The
configuration parameters include the CoreUARTapb hardware instance base address
and other runtime parameters, such as baud rate, bit width, and parity. No
CoreUARTapb hardware configuration parameters are needed by the driver, apart
from the CoreUARTapb hardware instance base address. Hence, no additional
configuration files are required to use the driver.

A CoreUARTapb hardware instance is generated with fixed baud rate, character
size, and parity configuration settings as part of the hardware flow. The
baud_value and line_config parameter values passed to the UART_init() function
have no effect if fixed values were selected for the baud rate, character size,
and parity in the hardware configuration of CoreUARTapb. When fixed values are
selected for these hardware configuration parameters, the driver is unable to
overwrite the fixed values in the CoreUARTapb control registers, CTRL1 and
CTRL2.

# Theory of Operation
The CoreUARTapb software driver is designed to allow the control of multiple
instances of CoreUARTapb. Each instance of CoreUARTapb in the hardware design is
associated with a single instance of the UART_instance_t structure in the
software. You need to allocate memory for one unique UART_instance_t structure
instance for each CoreUARTapb hardware instance. The contents of these data
structures are initialized while calling the UART_init() function. A pointer
to the structure is passed to the subsequent driver functions in order to
identify the CoreUARTapb hardware instance you wish to perform the requested
operation on.

Note: Do not attempt to directly manipulate the content of UART_instance_t
structures. This structure is only intended to be modified by the driver
function.

Once initialized, the driver transmits and receives data. Transmit is performed
using the UART_send() function. If this function blocks, then it returns only
when the data passed to it has been sent to the CoreUARTapb hardware. Data
received by the CoreUARTapb hardware is read by the user application using the
UART_get_rx() function.

The UART_fill_tx_fifo() function is also provided as a part of the interrupt-
driven transmit. This function fills the CoreUARTapb hardware transmit FIFO with
the content of a data buffer passed as a parameter before returning. The control
of the interrupts must be implemented outside the driver, as the CoreUARTapb
hardware does not provide the ability to enable or disable its interrupt
sources.

The UART_polled_tx_string() function is provided to transmit a NULL-terminated
string in polled mode. If this function blocks, then it returns only when the
data passed to it has been sent to the CoreUARTapb hardware.

The UART_get_rx_status() function returns the error status of the CoreUARTapb
receiver. This is used by applications to take appropriate action in case of
receiver errors.

</div>


# Types

 ---------------- 
<a name="uartinstancet"></a>
## UART_instance_t
<a name="prototype"></a>
### Prototype 

<div id="Types$UART_instance_t$prototype" data-type="code">

``` 
    typedef struct {
        addr_t base_address; 
        uint8_t status; 
    } UART_instance_t ;
  ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$UART_instance_t$description" data-type="text">

There should be one instance of this structure for each instance of CoreUARTapb
in your system. This structure instance identifies various UARTs in a system and
should be passed as first parameter to UART functions to identify which UART
performs the requested operation. The 'status' element in the structure is used
to provide sticky status information.


</div>


 --------------------------- 

# Constants

 ---------------- 
<div id="Constants$DATA_7_BITS$description" data-type="text">

<a name="data7bits"></a>
## Data Bits Length Defines
These constants define the data length in a UART packet.

| Constant    | Description     | 
| -----|-----|
| DATA_7_BITS    | Data length is 7-bits     | 
| DATA_8_BITS    | Data length is 8-bits    | 

</div>

<div id="Constants$NO_PARITY$description" data-type="text">

<a name="noparity"></a>
## Parity Defines
These constants define parity check options.

| Constant    | Description     | 
| -----|-----|
| NO_PARITY    | No Parity bit     | 
| EVEN_PARITY    | Even Parity bit     | 
| ODD_PARITY    | ODD Parity bit    | 

</div>

<div id="Constants$UART_APB_PARITY_ERROR$description" data-type="text">

<a name="uartapbparityerror"></a>
## Error Status Definitions
These constants define the different types of possible errors in UART
transmission of data.

| Constant    | Description     | 
| -----|-----|
| UART_APB_PARITY_ERROR    | Data parity error     | 
| UART_APB_OVERFLOW_ERROR    | Data overflow error     | 
| UART_APB_FRAMING_ERROR    | Data framing error     | 
| UART_APB_NO_ERROR    | No error     | 
| UART_APB_INVALID_PARAM    | Invalid parameter    | 

</div>


# Functions

 ---------------- 
<a name="uartinit"></a>
## UART_init
<a name="prototype"></a>
### Prototype 

<div id="Functions$UART_init$prototype" data-type="code">

    void
    UART_init
    (
        UART_instance_t * this_uart,
        addr_t base_addr,
        uint16_t baud_value,
        uint8_t line_config
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$UART_init$description" data-type="text">

The UART_init() function initializes the UART with the configuration passed as
parameters. The configuration parameters are the baud_value that generates the
baud rate and the line configuration (bit length and parity).

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_uart
<div id="Functions$UART_init$description$parameters$this_uart" data-type="text" data-name="this_uart">

The this_uart parameter is a pointer to the UART_instance_t structure, which
holds all data regarding this instance of the CoreUARTapb. This pointer is used
to identify the target CoreUARTapb hardware instance in subsequent calls to the
CoreUARTapb functions.


</div>

#### base_addr
<div id="Functions$UART_init$description$parameters$base_addr" data-type="text" data-name="base_addr">

The base_address parameter is the base address in the processor's memory map for
the registers of the CoreUARTapb instance being initialized.


</div>

#### baud_value
<div id="Functions$UART_init$description$parameters$baud_value" data-type="text" data-name="baud_value">

The baud_value parameter selects the baud rate for the UART. The baud value is
calculated from the frequency of the system clock in hertz and the desired baud
rate using the following equation:  
baud_value = (clock / (baud_rate * 16)) -
1.  
The baud_value parameter must be a value in the range 0 to 8191 (or 0x0000
to 0x1FFF).


</div>

#### line_config
<div id="Functions$UART_init$description$parameters$line_config" data-type="text" data-name="line_config">

This parameter is the line configuration, specifies the bit length and parity
settings. This is the logical OR of:

  - DATA_7_BITS
  - DATA_8_BITS
  - NO_PARITY
  - EVEN_PARITY
  - ODD_PARITY  
For example, 8 bits even parity would be specified as 
(DATA_8_BITS | EVEN_PARITY). 


</div>

<a name="return"></a>
### Return
<div id="Functions$UART_init$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$UART_init$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$UART_init$description$example$Example1" data-type="code" data-name="Example">


```
    #define BAUD_VALUE_57600    25

    #define COREUARTAPB0_BASE_ADDR      0xC3000000UL

    UART_instance_t g_uart;
    int main()
    {
       UART_init(&g_uart, COREUARTAPB0_BASE_ADDR, 
                 BAUD_VALUE_57600, (DATA_8_BITS | EVEN_PARITY));
    }
```

</div>


 -------------------------------- 
<a name="uartsend"></a>
## UART_send
<a name="prototype"></a>
### Prototype 

<div id="Functions$UART_send$prototype" data-type="code">

    void
    UART_send
    (
        UART_instance_t * this_uart,
        const uint8_t * tx_buffer,
        size_t tx_size
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$UART_send$description" data-type="text">

The UART_send() function is used to transmit data. It transfers the content of
the transmitter data buffer, passed as a function parameter, into the UART's
hardware transmitter FIFO. It returns when the full content of the transmitter
data buffer has been transferred to the UART's transmitter FIFO.

Note: You should not assume that the data you are sending using this function
has been received at the other end by the time this function returns. The actual
transmission over the serial connection is still be taking place at the time of
the function return. It is safe to release or reuse the memory used as the
transmit buffer once this function returns.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_uart
<div id="Functions$UART_send$description$parameters$this_uart" data-type="text" data-name="this_uart">

The this_uart parameter is a pointer to the UART_instance_t structure, which
holds all data regarding this instance of the CoreUARTapbUART.


</div>

#### tx_buffer
<div id="Functions$UART_send$description$parameters$tx_buffer" data-type="text" data-name="tx_buffer">

The tx_buffer parameter is a pointer to a buffer that contains the data to be
transmitted.


</div>

#### tx_size
<div id="Functions$UART_send$description$parameters$tx_size" data-type="text" data-name="tx_size">

The tx_size parameter is the size in bytes of the transmitted data.


</div>

<a name="return"></a>
### Return
<div id="Functions$UART_send$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$UART_send$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$UART_send$description$example$Example1" data-type="code" data-name="Example">


```
    uint8_t testmsg1[] = {"\n\r\n\r\n\rUART_send() test message 1"};
    UART_send(&g_uart,(const uint8_t *)&testmsg1,sizeof(testmsg1));
```

</div>


 -------------------------------- 
<a name="uartfilltxfifo"></a>
## UART_fill_tx_fifo
<a name="prototype"></a>
### Prototype 

<div id="Functions$UART_fill_tx_fifo$prototype" data-type="code">

    size_t
    UART_fill_tx_fifo
    (
        UART_instance_t * this_uart,
        const uint8_t * tx_buffer,
        size_t tx_size
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$UART_fill_tx_fifo$description" data-type="text">

The UART_fill_tx_fifo() function fills the UART's transmitter hardware FIFO with
the data found in the transmitter buffer that is passed in as a function
parameter. The function returns either when the FIFO is full or when the
complete contents of the transmitter buffer have been copied into the FIFO. It
returns the number of bytes copied into the UART's transmitter hardware FIFO.
This function is intended to be used as part of interrupt-driven transmission.

Note: You should not assume that the data you transmit using this function has
been received at the other end by the time this function returns. The actual
transmission over the serial connection is still be taking place at the time of
the function return.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_uart
<div id="Functions$UART_fill_tx_fifo$description$parameters$this_uart" data-type="text" data-name="this_uart">

The this_uart parameter is a pointer to the UART_instance_t structure, which
holds all data regarding this instance of the UART.


</div>

#### tx_buffer
<div id="Functions$UART_fill_tx_fifo$description$parameters$tx_buffer" data-type="text" data-name="tx_buffer">

The tx_buffer parameter is a pointer to a buffer that contains the data to be
transmitted.


</div>

#### tx_size
<div id="Functions$UART_fill_tx_fifo$description$parameters$tx_size" data-type="text" data-name="tx_size">

The tx_size parameter is the size in bytes of the transmitted data.


</div>

<a name="return"></a>
### Return
<div id="Functions$UART_fill_tx_fifo$description$return" data-type="text">

This function returns the number of bytes copied into the UART's transmitter
hardware FIFO.


</div>

##### Example
<div id="Functions$UART_fill_tx_fifo$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$UART_fill_tx_fifo$description$example$Example1" data-type="code" data-name="Example">


```
    void send_using_interrupt
    (
        uint8_t * pbuff,
        size_t tx_size
    )
    {
        size_t size_in_fifo;
        size_in_fifo = UART_fill_tx_fifo( &g_uart, pbuff, tx_size );
    }
```

</div>


 -------------------------------- 
<a name="uartgetrx"></a>
## UART_get_rx
<a name="prototype"></a>
### Prototype 

<div id="Functions$UART_get_rx$prototype" data-type="code">

    size_t
    UART_get_rx
    (
        UART_instance_t * this_uart,
        uint8_t * rx_buffer,
        size_t buff_size
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$UART_get_rx$description" data-type="text">

The UART_get_rx() function reads the content of the UART's receiver hardware
FIFO and stores it in the receiver buffer that is passed in as a function
parameter. It copies either the full contents of the FIFO into the receiver
buffer, or just enough data from the FIFO to fill the receiver buffer, depending
on the size of the receiver buffer. The size of the receiver buffer is passed in
as a function parameter. UART_get_rx() returns the number of bytes copied into
the receiver buffer. If no data was received at the time the function is called,
the function returns 0.

Note: This function reads and accumulates the receiver status of the CoreUARTapb
instance before reading each byte from the receiver's data register/FIFO. This
allows the driver to maintain a sticky record of any receiver errors that occur
as the UART receives each data byte; receiver errors would otherwise be lost
after each read from the receiver's data register. A call to the
UART_get_rx_status() function returns any receiver errors accumulated during the
execution of the UART_get_rx() function.

Note: When FIFO mode is disabled in the CoreUARTapb hardware configuration, the
driver accumulates a sticky record of any parity errors, framing errors, or
overflow errors. When FIFO mode is enabled, the driver accumulates a sticky
record of overflow errors only; in this case, interrupts must be used to handle
parity errors or framing errors.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_uart
<div id="Functions$UART_get_rx$description$parameters$this_uart" data-type="text" data-name="this_uart">

The this_uart parameter is a pointer to the UART_instance_t structure, which
holds all data regarding this instance of the UART.


</div>

#### rx_buffer
<div id="Functions$UART_get_rx$description$parameters$rx_buffer" data-type="text" data-name="rx_buffer">

The rx_buffer parameter is a pointer to a buffer where the received data is
copied.


</div>

#### buff_size
<div id="Functions$UART_get_rx$description$parameters$buff_size" data-type="text" data-name="buff_size">

The buff_size parameter is the size of the receive buffer in bytes.


</div>

<a name="return"></a>
### Return
<div id="Functions$UART_get_rx$description$return" data-type="text">

This function returns the number of bytes copied into the receive buffer.


</div>

##### Example
<div id="Functions$UART_get_rx$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$UART_get_rx$description$example$Example1" data-type="code" data-name="Example">


```
    #define MAX_RX_DATA_SIZE    256

    uint8_t rx_data[MAX_RX_DATA_SIZE];
    uint8_t rx_size = 0;
            
    rx_size = UART_get_rx( &g_uart, rx_data, sizeof(rx_data) );
```

</div>


 -------------------------------- 
<a name="uartpolledtxstring"></a>
## UART_polled_tx_string
<a name="prototype"></a>
### Prototype 

<div id="Functions$UART_polled_tx_string$prototype" data-type="code">

    void
    UART_polled_tx_string
    (
        UART_instance_t * this_uart,
        const uint8_t * p_sz_string
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$UART_polled_tx_string$description" data-type="text">

The UART_polled_tx_string() function is used to transmit a NULL ('\0')
terminated string. Internally, it polls for the transmit ready status and
transfers the text starting at the address pointed by p_sz_string into the
UART's hardware transmitter FIFO. It is a blocking function and returns only
when the complete string has been transferred to the UART's transmit FIFO.

Note: You should not assume that the data you transmit using this function has
been received at the other end by the time this function returns. The actual
transmission over the serial connection is still be taking place at the time of
the function return.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_uart
<div id="Functions$UART_polled_tx_string$description$parameters$this_uart" data-type="text" data-name="this_uart">

The this_uart parameter is the pointer to a UART_instance_t structure, which
holds all data regarding this instance of the UART.


</div>

#### p_sz_string
<div id="Functions$UART_polled_tx_string$description$parameters$p_sz_string" data-type="text" data-name="p_sz_string">

The p_sz_string parameter is a pointer to a buffer containing the NULL ('\0')
terminated string to be transmitted.


</div>

<a name="return"></a>
### Return
<div id="Functions$UART_polled_tx_string$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$UART_polled_tx_string$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$UART_polled_tx_string$description$example$Example1" data-type="code" data-name="Example">


```
    uint8_t testmsg1[] = {"\r\n\r\nUART_polled_tx_string() test message 1\0"};
    UART_polled_tx_string(&g_uart,(const uint8_t *)&testmsg1);
```

</div>


 -------------------------------- 
<a name="uartgetrxstatus"></a>
## UART_get_rx_status
<a name="prototype"></a>
### Prototype 

<div id="Functions$UART_get_rx_status$prototype" data-type="code">

    uint8_t
    UART_get_rx_status
    (
        UART_instance_t * this_uart
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$UART_get_rx_status$description" data-type="text">

The UART_get_rx_status() function returns the receiver error status of the
CoreUARTapb instance. It reads both the current error status of the receiver and
the accumulated error status from preceding calls to the UART_get_rx() function
and combines them using a bitwise OR. It returns the cumulative parity, framing,
and overflow error status of the receiver, since the previous call to
UART_get_rx_status() as an 8-bit encoded value.

Note: The UART_get_rx() function reads and accumulates the receiver status of
the CoreUARTapb instance before reading each byte from the receiver's data
register/FIFO. The driver maintains a sticky record of the cumulative error
status, which persists after the UART_get_rx() function returns. The
UART_get_rx_status() function clears this accumulated record of receiver errors
before returning.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_uart
<div id="Functions$UART_get_rx_status$description$parameters$this_uart" data-type="text" data-name="this_uart">

The this_uart parameter is a pointer to a UART_instance_t structure which holds
all data regarding this instance of the UART.


</div>

<a name="return"></a>
### Return
<div id="Functions$UART_get_rx_status$description$return" data-type="text">

This function returns the UART receiver error status as an 8-bit encoded value.
The return value is 0, if there are no receiver errors occurred. The driver
provides a set of bit mask constants, which should be compared with and/or used
to mask the returned value to determine the receiver error status.  
When the
return value is compared to the following bit masks, a non-zero result indicates
that the corresponding error occurred:  
UART_APB_PARITY_ERROR (bit mask = 0x01)  
UART_APB_OVERFLOW_ERROR (bit mask = 0x02)  
UART_APB_FRAMING_ERROR (bit mask =
0x04)  
When the return value is compared to the following bit 
</html>
-zero
result indicates that no error occurred:  
UART_APB_NO_ERROR (0x00)


</div>

##### Example
<div id="Functions$UART_get_rx_status$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$UART_get_rx_status$description$example$Example1" data-type="code" data-name="Example">


```
    UART_instance_t g_uart;
    uint8_t rx_data[MAX_RX_DATA_SIZE];
    uint8_t err_status;
    err_status = UART_get_err_status(&g_uart);

    if(UART_APB_NO_ERROR == err_status )
    {
         rx_size = UART_get_rx( &g_uart, rx_data, MAX_RX_DATA_SIZE );
    }
```

</div>


 -------------------------------- 
