<html>

 ------------------------------------

# CoreTimer Bare Metal Driver

-----------------------------------------

## Table of Contents

- [Introduction](#introduction)

- [Driver Configuration](#driver-configuration)

- [Theory of Operation](#theory-of-operation)

- [Types](#types)
  - [timer_instance_t](#timerinstancet)

- [Constants](#constants)
  - [Timer Operating Mode](#timer-operating-mode)
  - [Prescaler Value](#prescaler-value)

- [Functions](#functions)
  - [TMR_init](#tmrinit)
  - [TMR_start](#tmrstart)
  - [TMR_stop](#tmrstop)
  - [TMR_enable_int](#tmrenableint)
  - [TMR_clear_int](#tmrclearint)
  - [TMR_current_value](#tmrcurrentvalue)
  - [TMR_reload](#tmrreload)

<div id="TitlePage" data-type="text">

# Introduction

The CoreTimer is an IP component that provides access to an interrupt-
generating, programmable decrementing counter over an Advanced Peripheral Bus
(APB) slave interface.

This driver provides a set of functions for controlling CoreTimer as part of the
bare metal system where no operating system is available. This driver can be
adapted to be used as a part of an operating system, but the implementation of
the adaptation layer between the driver and the operating system's driver model
is outside the scope of this user guide.

# Driver Configuration

Your application software must configure the CoreTimer driver through calls to
TMR_init() for each CoreTimer instance in the hardware design. This function
configures a default set of parameters that include a CoreTimer hardware
instance base address, operating mode, prescaler value, and timer load value.

The CoreTimer supports the following two modes of operation:

- Continuous mode
- One-Shot Timer mode

Continuous mode: This is the default mode. When zero is reached, the counter is
reloaded with the start value, which is stored in a programmable register, and
continues to count down. If the interrupt is enabled, this mode generates an
interrupt at a regular interval.

One-Shot Timer mode: The counter decrements from its high value and halts at
zero. The timer must be reprogrammed to begin counting down again. This can be
achieved by either loading a new value to the timer's counter using TMR_reload()
or by changing the operating mode through a call to TMR_init().

Either mode of operation can be set during initialization using TMR_init(). If
interrupts are enabled, an interrupt is generated when the decrementing counter
reaches zero.

# Theory of Operation

The timer is loaded by writing to the load register and then counts down to zero
if enabled. To load the timer with an initial value to decrement from, use
TMR_init(). To enable the timer to count down to zero, use TMR_start().

The prescaler value can be set to divide down the clock to decrement the
CoreTimer counter. The prescaler values can be set to trigger a pulse at every
2, 4, 8, 16, 32, 64, 128, 256, 512, or 1024 clock periods.

Use TMR_reload() to load new values into the timer's load register. If
TMR_reload() is called while the timer is already running, the timer immediately
begins to decrement from the new value.

If interrupts are enabled with TMR_enable_int(), an interrupt is generated when
the timer reaches zero. To clear the interrupt, write to the interrupt clear
register using TMR_clear_int().

If the timer is operating in One-Shot Timer mode, it halts on reaching zero
until the timer is reloaded using TMR_reload() or until the Operating mode is
changed to Continuous mode. When operating in Continuous mode, the timer reloads
the count value that was initially set using TMR_init() and continues to
decrement. The counter effectively generates a periodic interrupt when operating
in Continuous mode.

To read the current counter value at any time, use TMR_current_value().

</div>

# Types

 ----------------
<a name="timerinstancet"></a>

## timer_instance_t

<a name="prototype"></a>

### Prototype

<div id="Types$timer_instance_t$prototype" data-type="code">

```
    typedef struct __timer_instance_t {
        addr_t base_address; 
    } timer_instance_t ;
  ```

</div>

<a name="description"></a>

### Description

<div id="Types$timer_instance_t$description" data-type="text">

There must be one instance of this structure for each instance of CoreTimer in
your system. TMR_init() initializes this structure. It identifies the various
CoreTimer hardware instances in your system. An initialized timer instance
structure must be passed as the first parameter to CoreTimer driver functions to
identify which CoreTimer instance must perform the requested operation. Software
using this driver must only create one single instance of this data structure
for each hardware timer instance in the system.

</div>

 ---------------------------

# Constants

 ----------------
<div id="Constants$TMR_CONTINUOUS_MODE$description" data-type="text">

<a name="tmrcontinuousmode"></a>

## Timer Operating Mode

The following definitions select the operating mode of the CoreTimer driver.
They allow for selecting Continuous or One-Shot Timer modes.

| Constant            | Description                                   |
| ------------------- | --------------------------------------------- |
| TMR_CONTINUOUS_MODE | Coretimer will operate in Continuous mode     |
| TMR_ONE_SHOT_MODE   | Coretimer will operate in One-Shot Timer mode |

</div>

<div id="Constants$PRESCALER_DIV_2$description" data-type="text">

<a name="prescalerdiv2"></a>

## Prescaler Value

The following definitions configure the CoreTimer prescaler. The prescaler
divides down the clock to decrement the CoreTimer counter. It can be configured
to divide the clock by 2, 4, 8, 16, 32, 64, 128, 256, 512, or 1024 clock
periods.

| Constant           | Description                                    |
| ------------------ | ---------------------------------------------- |
| PRESCALER_DIV_2    | Scale down the timer's clock frequency by 2    |
| PRESCALER_DIV_4    | Scale down the timer's clock frequency by 4    |
| PRESCALER_DIV_8    | Scale down the timer's clock frequency by 8    |
| PRESCALER_DIV_16   | Scale down the timer's clock frequency by 16   |
| PRESCALER_DIV_32   | Scale down the timer's clock frequency by 32   |
| PRESCALER_DIV_64   | Scale down the timer's clock frequency by 64   |
| PRESCALER_DIV_128  | Scale down the timer's clock frequency by 128  |
| PRESCALER_DIV_256  | Scale down the timer's clock frequency by 256  |
| PRESCALER_DIV_512  | Scale down the timer's clock frequency by 512  |
| PRESCALER_DIV_1024 | Scale down the timer's clock frequency by 1024 |

</div>

# Functions

 ----------------
<a name="tmrinit"></a>

## TMR_init

<a name="prototype"></a>

### Prototype

<div id="Functions$TMR_init$prototype" data-type="code">

    void
    TMR_init
    (
        timer_instance_t * this_timer,
        addr_t address,
        uint8_t mode,
        uint32_t prescaler,
        uint32_t load_value
    );

</div>

<a name="description"></a>

### Description

<div id="Functions$TMR_init$description" data-type="text">

This function initializes the data structure and sets relevant CoreTimer
registers. It prepares the timer for use in a given hardware or software
configuration. It must be called before any other timer API function. The timer
won't start counting down immediately after this function is called. It is
necessary to call TMR_start() to start the timer decrementing. The CoreTimer
interrupt is disabled as part of this function.

</div>

 ---------------------------

<a name="parameters"></a>

### Parameters

#### this_timer

<div id="Functions$TMR_init$description$parameters$this_timer" data-type="text" data-name="this_timer">

Pointer to a timer_instance_t structure holding all relevant data associated
with the target timer hardware instance. This pointer identifies the target
CoreTimer hardware instance in subsequent calls to the CoreTimer functions.

</div>

#### address

<div id="Functions$TMR_init$description$parameters$address" data-type="text" data-name="address">

Base address in the processor's memory map of the registers of the CoreTimer
instance being initialized.

</div>

#### mode

<div id="Functions$TMR_init$description$parameters$mode" data-type="text" data-name="mode">

This parameter selects the operating mode of the timer instance. This can be
either TMR_CONTINUOUS_MODE or TMR_ONE_SHOT_MODE.

</div>

#### prescaler

<div id="Functions$TMR_init$description$parameters$prescaler" data-type="text" data-name="prescaler">

This parameter selects the prescaler divider that divides down the clock used to
decrement the timer's counter. This can be set using one of the
`PRESCALER_DIV_<n>` definitions, where `<n>` is the divider's value.

</div>

#### load_value

<div id="Functions$TMR_init$description$parameters$load_value" data-type="text" data-name="load_value">

This parameter sets the timer's load value, from which the CoreTimer counter
decrements In Continuous mode, this value reloads the timer's counter whenever
it reaches zero.

</div>

<a name="return"></a>

### Return

<div id="Functions$TMR_init$description$return" data-type="text">

This function does not return any value.

</div>

 --------------------------------
<a name="tmrstart"></a>

## TMR_start

<a name="prototype"></a>

### Prototype

<div id="Functions$TMR_start$prototype" data-type="code">

    void
    TMR_start
    (
        timer_instance_t * this_timer
    );

</div>

<a name="description"></a>

### Description

<div id="Functions$TMR_start$description" data-type="text">

This function starts the timer counting down. This function must be called after
calling TMR_init(). It does not need to be called after each call to
TMR_reload() when the timer is used in One-Shot Timer mode.

</div>

 ---------------------------

<a name="parameters"></a>

### Parameters

#### this_timer

<div id="Functions$TMR_start$description$parameters$this_timer" data-type="text" data-name="this_timer">

Pointer to a timer_instance_t structure holding all relevant data associated
with the target timer hardware instance. This pointer identifies the target
CoreTimer hardware instance.

</div>

<a name="return"></a>

### Return

<div id="Functions$TMR_start$description$return" data-type="text">

This function does not return any value.

</div>

 --------------------------------
<a name="tmrstop"></a>

## TMR_stop

<a name="prototype"></a>

### Prototype

<div id="Functions$TMR_stop$prototype" data-type="code">

    void
    TMR_stop
    (
        timer_instance_t * this_timer
    );

</div>

<a name="description"></a>

### Description

<div id="Functions$TMR_stop$description" data-type="text">

This function stops the timer from counting down. It stops interrupts from being
generated when Continuous mode is used and interrupts must be paused.

</div>

 ---------------------------

<a name="parameters"></a>

### Parameters

#### this_timer

<div id="Functions$TMR_stop$description$parameters$this_timer" data-type="text" data-name="this_timer">

Pointer to a timer_instance_t structure holding all relevant data associated
with the target timer hardware instance. This pointer identifies the target
CoreTimer hardware instance.

</div>

<a name="return"></a>

### Return

<div id="Functions$TMR_stop$description$return" data-type="text">

This function does not return any value.

</div>

 --------------------------------
<a name="tmrenableint"></a>

## TMR_enable_int

<a name="prototype"></a>

### Prototype

<div id="Functions$TMR_enable_int$prototype" data-type="code">

    void
    TMR_enable_int
    (
        timer_instance_t * this_timer
    );

</div>

<a name="description"></a>

### Description

<div id="Functions$TMR_enable_int$description" data-type="text">

This function enables the timer interrupt. It allows the interrupt signal coming
out of CoreTimer to be asserted.

</div>

 ---------------------------

<a name="parameters"></a>

### Parameters

#### this_timer

<div id="Functions$TMR_enable_int$description$parameters$this_timer" data-type="text" data-name="this_timer">

Pointer to a timer_instance_t structure holding all relevant data associated
with the target timer hardware instance. This pointer identifies the target
CoreTimer hardware instance.

</div>

<a name="return"></a>

### Return

<div id="Functions$TMR_enable_int$description$return" data-type="text">

This function does not return any value.

</div>

 --------------------------------
<a name="tmrclearint"></a>

## TMR_clear_int

<a name="prototype"></a>

### Prototype

<div id="Functions$TMR_clear_int$prototype" data-type="code">

    void
    TMR_clear_int
    (
        timer_instance_t * this_timer
    );

</div>

<a name="description"></a>

### Description

<div id="Functions$TMR_clear_int$description" data-type="text">

This function clears the timer interrupt. It must be called within the interrupt
handler, servicing interrupts from the timer. Failure to clear the timer
interrupt results in the interrupt signal generated from CoreTimer remaining
asserted. This assertion may cause the interrupt service routine to be
continuously called, causing the system to lock up.

</div>

 ---------------------------

<a name="parameters"></a>

### Parameters

#### this_timer

<div id="Functions$TMR_clear_int$description$parameters$this_timer" data-type="text" data-name="this_timer">

Pointer to a timer_instance_t structure holding all relevant data associated
with the target timer hardware instance. This pointer identifies the target
CoreTimer hardware instance.

</div>

<a name="return"></a>

### Return

<div id="Functions$TMR_clear_int$description$return" data-type="text">

This function does not return any value.

</div>

 --------------------------------
<a name="tmrcurrentvalue"></a>

## TMR_current_value

<a name="prototype"></a>

### Prototype

<div id="Functions$TMR_current_value$prototype" data-type="code">

    uint32_t
    TMR_current_value
    (
        timer_instance_t * this_timer
    );

</div>

<a name="description"></a>

### Description

<div id="Functions$TMR_current_value$description" data-type="text">

This function returns the current value of the counter.

</div>

 ---------------------------

<a name="parameters"></a>

### Parameters

#### this_timer

<div id="Functions$TMR_current_value$description$parameters$this_timer" data-type="text" data-name="this_timer">

Pointer to a timer_instance_t structure holding all relevant data associated
with the target timer hardware instance. This pointer identifies the target
CoreTimer hardware instance.

</div>

<a name="return"></a>

### Return

<div id="Functions$TMR_current_value$description$return" data-type="text">

Returns the current value of the timer counter value.

</div>

 --------------------------------
<a name="tmrreload"></a>

## TMR_reload

<a name="prototype"></a>

### Prototype

<div id="Functions$TMR_reload$prototype" data-type="code">

    void
    TMR_reload
    (
        timer_instance_t * this_timer,
        uint32_t load_value
    );

</div>

<a name="description"></a>

### Description

<div id="Functions$TMR_reload$description" data-type="text">

This function is used in One-Shot Timer mode. It reloads the timer counter with
the value passed as a parameter. The timer counter begins to decrement again
after calling TMR_reload(), it is not required to call TMR_start() again.

</div>

 ---------------------------

<a name="parameters"></a>

### Parameters

#### this_timer

<div id="Functions$TMR_reload$description$parameters$this_timer" data-type="text" data-name="this_timer">

Pointer to a timer_instance_t structure holding all relevant data associated
with the target timer hardware instance. This pointer identifies the target
CoreTimer hardware instance.

</div>

#### load_value

<div id="Functions$TMR_reload$description$parameters$load_value" data-type="text" data-name="load_value">

This parameter sets the timer's load value, from which the CoreTimer counter
decrements.

</div>

<a name="return"></a>

### Return

<div id="Functions$TMR_reload$description$return" data-type="text">

This function does not return any value.

</div>

 --------------------------------
