<html>
 
 ------------------------------------ 

# PolarFire® SoC MSS Timer Bare Metal Driver.
 
-----------------------------------------
## Table of Contents 
  - [Introduction](#introduction)

  - [Theory of Operation](#theory-of-operation)

- [Types](#types)
  - [mss_timer_mode_t](#msstimermodet)

- [Constants](#constants)

- [Functions](#functions)
  - [MSS_TIM1_init](#msstim1init)
  - [MSS_TIM1_start](#msstim1start)
  - [MSS_TIM1_stop](#msstim1stop)
  - [MSS_TIM1_get_current_value](#msstim1getcurrentvalue)
  - [MSS_TIM1_load_immediate](#msstim1loadimmediate)
  - [MSS_TIM1_load_background](#msstim1loadbackground)
  - [MSS_TIM1_enable_irq](#msstim1enableirq)
  - [MSS_TIM1_disable_irq](#msstim1disableirq)
  - [MSS_TIM1_clear_irq](#msstim1clearirq)
  - [MSS_TIM2_init](#msstim2init)
  - [MSS_TIM2_start](#msstim2start)
  - [MSS_TIM2_stop](#msstim2stop)
  - [MSS_TIM2_get_current_value](#msstim2getcurrentvalue)
  - [MSS_TIM2_load_immediate](#msstim2loadimmediate)
  - [MSS_TIM2_load_background](#msstim2loadbackground)
  - [MSS_TIM2_enable_irq](#msstim2enableirq)
  - [MSS_TIM2_disable_irq](#msstim2disableirq)
  - [MSS_TIM2_clear_irq](#msstim2clearirq)
  - [MSS_TIM64_init](#msstim64init)
  - [MSS_TIM64_start](#msstim64start)
  - [MSS_TIM64_stop](#msstim64stop)
  - [MSS_TIM64_get_current_value](#msstim64getcurrentvalue)
  - [MSS_TIM64_load_immediate](#msstim64loadimmediate)
  - [MSS_TIM64_load_background](#msstim64loadbackground)
  - [MSS_TIM64_enable_irq](#msstim64enableirq)
  - [MSS_TIM64_disable_irq](#msstim64disableirq)
  - [MSS_TIM64_clear_irq](#msstim64clearirq)

<div id="TitlePage" data-type="text">

# Introduction
The PolarFire® SoC Microprocessor Subsystem (MSS) includes a timer hardware
block which is used as two independent 32-bit timers or as a single 64-bit timer
in Periodic or One-shot mode.

This driver provides a set of functions for controlling the MSS Timer as part of
a bare metal system where no operating system is available. These drivers can be
adapted for use as part of an operating system, but the implementation of the
adaptation layer between this driver and the operating system's driver model is
outside the scope of this driver.

# Theory of Operation
The PolarFire SoC MSS Timer is used in one of two mutually exclusive modes,
either as a single 64-bit timer or as two independent 32-bit timers. The MSS
Timer is used in either Periodic mode or One-shot mode. A timer, when configured
for Periodic mode, generates an interrupt and reloads its down-counter when it
reaches zero. The timer then continues decrementing from its reload value
without waiting for the interrupt to be cleared. A timer, when configured for
One-shot mode, generates an interrupt once when its down-counter reaches zero.
It must be explicitly reloaded to start decrementing again.

The MSS Timer driver functions are grouped into the following categories:

  - Initialization and configuration
  - Timer control
  - Interrupt control

The MSS Timer driver provides three initialization functions:

  - MSS_TIM1_init()
  - MSS_TIM2_init()
  - MSS_TIM64_init()

The MSS Timer driver is initialized through calls to these functions and at
least one of them must be called before any other MSS Timer driver function can
be called. You should only use MSS_TIM1_init() and MSS_TIM2_init() if you intend
to use the timer in 32-bit mode. Use MSS_TIM64_init() if you intend to use the
MSS Timer as a single 64-bit timer. Initialization functions take a single
parameter specifying the operating mode of the timer being initialized.

Once initialized, a timer is controlled using the following functions:

  - MSS_TIM1_load_immediate()
  - MSS_TIM1_load_background()
  - MSS_TIM1_get_current_value()
  - MSS_TIM1_start()
  - MSS_TIM1_stop()
  - MSS_TIM2_load_immediate()
  - MSS_TIM2_load_background()
  - MSS_TIM2_get_current_value()
  - MSS_TIM2_start()
  - MSS_TIM2_stop()
  - MSS_TIM64_load_immediate()
  - MSS_TIM64_load_background()
  - MSS_TIM64_get_current_value()
  - MSS_TIM64_start()
  - MSS_TIM64_stop()

Timer interrupts are controlled using the following functions:

  - MSS_TIM1_enable_irq()
  - MSS_TIM1_disable_irq()
  - MSS_TIM1_clear_irq()
  - MSS_TIM2_enable_irq()
  - MSS_TIM2_disable_irq()
  - MSS_TIM2_clear_irq()
  - MSS_TIM64_enable_irq()
  - MSS_TIM64_disable_irq()
  - MSS_TIM64_clear_irq()

The timer interrupt handlers have the following function prototypes:

  - void Timer1_IRQHandler(void)
  - void Timer2_IRQHandler(void)

Entries for these interrupt handlers are provided in the PolarFire SoC MPFS HAL
vector table. To add a Timer 1 interrupt handler, you must implement
Timer1_IRQHandler() as part of your application code. To add a Timer 2 interrupt
handler, you must implement Timer2_IRQHandler() as part of your application
code. When using the MSS Timer as a 64-bit timer, you must implement
Timer1_IRQHandler() as part of your application code. The Timer 2 interrupt is
not used when the MSS Timer is configured as a 64-bit timer.

</div>


# Types

 ---------------- 
<a name="msstimermodet"></a>
## mss_timer_mode_t
<a name="prototype"></a>
### Prototype 

<div id="Types$mss_timer_mode_t$prototype" data-type="code">

 ``` 
    typedef enum {
        MSS_TIMER_PERIODIC_MODE = 0,
        MSS_TIMER_ONE_SHOT_MODE = 1
    } mss_timer_mode_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$mss_timer_mode_t$description" data-type="text">

This enumeration selects between the two possible timer modes of operation:
Periodic and One-shot mode. It is used as an argument to the MSS_TIM1_init(),
MSS_TIM2_init(), and MSS_TIM64_init() function.

MSS_TIMER_PERIODIC_MODE: The timer generates interrupts at constant intervals.
On reaching zero, the counter of the timer is reloaded with a value held in a
register and begins counting down again.

MSS_TIMER_ONE_SHOT_MODE: The timer generates a single interrupt in this mode. On
reaching zero, the counter of the timer halts until reprogrammed by the user.


</div>


 --------------------------- 

# Constants

 ---------------- 

# Functions

 ---------------- 
<a name="msstim1init"></a>
## MSS_TIM1_init
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM1_init$prototype" data-type="code">

    void
    MSS_TIM1_init
    (
        TIMER_TypeDef * timer,
        mss_timer_mode_t mode
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM1_init$description" data-type="text">

MSS_TIM1_init() initializes the MSS Timer block for use as a 32-bit timer and
selects the operating mode for Timer 1. The MSS Timer block is out of reset
before executing this function. MSS_TIM1_init() stops Timer 1, disables its
interrupt, and sets the Timer 1 operating mode.

Note: The MSS Timer block cannot be used both as a 64-bit and 32-bit timer. When
the MSS_TIM1_init() function is invoked, it overwrites any previous
configuration of the MSS Timer as a 64-bit timer.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM1_init$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

#### mode
<div id="Functions$MSS_TIM1_init$description$parameters$mode" data-type="text" data-name="mode">

The mode parameter specifies whether the timer operates in Periodic or One-shot
mode.

  - Following are the allowed values:
      - MSS_TIMER_PERIODIC_MODE
      - MSS_TIMER_ONE_SHOT_MODE


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM1_init$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim1start"></a>
## MSS_TIM1_start
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM1_start$prototype" data-type="code">

    void
    MSS_TIM1_start
    (
        TIMER_TypeDef * timer
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM1_start$description" data-type="text">

MSS_TIM1_start() enables Timer 1 and starts its down-counter, decrementing from
the load_value specified in previous calls to MSS_TIM1_load_immediate() or
MSS_TIM1_load_background().

Note: MSS_TIM1_start() is also used to resume the down-counter if previously
stopped using MSS_TIM1_stop().

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM1_start$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM1_start$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim1stop"></a>
## MSS_TIM1_stop
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM1_stop$prototype" data-type="code">

    void
    MSS_TIM1_stop
    (
        TIMER_TypeDef * timer
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM1_stop$description" data-type="text">

MSS_TIM1_stop() disables Timer 1 and stops its down-counter.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM1_stop$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM1_stop$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim1getcurrentvalue"></a>
## MSS_TIM1_get_current_value
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM1_get_current_value$prototype" data-type="code">

    uint32_t
    MSS_TIM1_get_current_value
    (
        TIMER_TypeDef * timer
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM1_get_current_value$description" data-type="text">

MSS_TIM1_get_current_value() returns the current value of the Timer 1 down-
counter.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM1_get_current_value$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM1_get_current_value$description$return" data-type="text">

This function returns the 32-bit current value of the Timer 1 down-counter.


</div>


 -------------------------------- 
<a name="msstim1loadimmediate"></a>
## MSS_TIM1_load_immediate
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM1_load_immediate$prototype" data-type="code">

    void
    MSS_TIM1_load_immediate
    (
        TIMER_TypeDef * timer,
        uint32_t load_value
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM1_load_immediate$description" data-type="text">

MSS_TIM1_load_immediate() loads the value passed by the load_value parameter
into the Timer 1 down-counter. The counter decrements immediately from this
value once Timer 1 is enabled. The MSS Timer generates an interrupt when the
counter reaches zero, if Timer 1 interrupts are enabled. This function is
intended to be used when Timer 1 is configured for One-shot mode to time a
single delay.

Note: The value passed by the load_value parameter is loaded immediately into
the down-counter regardless of whether Timer 1 is operating in Periodic or One-
shot mode.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM1_load_immediate$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

#### load_value
<div id="Functions$MSS_TIM1_load_immediate$description$parameters$load_value" data-type="text" data-name="load_value">

The load_value parameter specifies the value from which the Timer 1 down-counter
starts decrementing from.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM1_load_immediate$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim1loadbackground"></a>
## MSS_TIM1_load_background
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM1_load_background$prototype" data-type="code">

    void
    MSS_TIM1_load_background
    (
        TIMER_TypeDef * timer,
        uint32_t load_value
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM1_load_background$description" data-type="text">

MSS_TIM1_load_background() specifies the value that reloads into the Timer 1
down-counter the next time the counter reaches zero. This function is typically
used when Timer 1 is configured for Periodic mode to select or change the delay
period between the interrupts generated by Timer 1.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM1_load_background$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

#### load_value
<div id="Functions$MSS_TIM1_load_background$description$parameters$load_value" data-type="text" data-name="load_value">

The load_value parameter specifies the value that loads into the Timer 1 down-
counter the next time the down-counter reaches zero. The Timer 1 down-counter
starts decrementing from this value after the current count expires.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM1_load_background$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim1enableirq"></a>
## MSS_TIM1_enable_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM1_enable_irq$prototype" data-type="code">

    void
    MSS_TIM1_enable_irq
    (
        TIMER_TypeDef * timer
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM1_enable_irq$description" data-type="text">

MSS_TIM1_enable_irq() enables interrupt generation for Timer 1. This function
also enables the interrupt in the RISC-V PLIC. Timer1_IRQHandler() is called
when a Timer 1 interrupt occurs.

Note: A Timer1_IRQHandler() default implementation is defined, with weak
linkage, in the MPFS HAL. You must provide your own implementation of
Timer1_IRQHandler(), which overrides the default implementation, to suit your
application.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM1_enable_irq$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM1_enable_irq$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim1disableirq"></a>
## MSS_TIM1_disable_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM1_disable_irq$prototype" data-type="code">

    void
    MSS_TIM1_disable_irq
    (
        TIMER_TypeDef * timer
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM1_disable_irq$description" data-type="text">

MSS_TIM1_disable_irq() disables interrupt generation for Timer 1. This function
also disables the interrupt in the RISC-V PLIC.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM1_disable_irq$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM1_disable_irq$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim1clearirq"></a>
## MSS_TIM1_clear_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM1_clear_irq$prototype" data-type="code">

    void
    MSS_TIM1_clear_irq
    (
        TIMER_TypeDef * timer
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM1_clear_irq$description" data-type="text">

MSS_TIM1_clear_irq() clears a pending interrupt from Timer 1. This function also
clears the interrupt in the RISC-V PLIC.

Note: You must call MSS_TIM1_clear_irq() as part of your implementation of the
Timer1_IRQHandler() Timer 1 Interrupt Service Routine (ISR) in order to prevent
the same interrupt event retriggering a call to the ISR.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM1_clear_irq$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM1_clear_irq$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim2init"></a>
## MSS_TIM2_init
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM2_init$prototype" data-type="code">

    void
    MSS_TIM2_init
    (
        TIMER_TypeDef * timer,
        mss_timer_mode_t mode
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM2_init$description" data-type="text">

MSS_TIM2_init() initializes the MSS Timer block for use as a 32-bit timer and
selects the operating mode for Timer 2. The MSS Timer block is already out of
reset before executing MSS_TIM2_init(). This function stops Timer 2, disables
its interrupt, and sets the Timer 2 operating mode.

Note: The MSS Timer block cannot be used both as a 64-bit and 32-bit timer. When
MSS_TIM2_init() is invoked, it overwrites any previous configuration of the MSS
Timer as a 64-bit timer.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM2_init$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

#### mode
<div id="Functions$MSS_TIM2_init$description$parameters$mode" data-type="text" data-name="mode">

The mode parameter specifies whether the timer operates in Periodic or One-shot
mode.

  - Following are the allowed values:
      - MSS_TIMER_PERIODIC_MODE
      - MSS_TIMER_ONE_SHOT_MODE


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM2_init$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim2start"></a>
## MSS_TIM2_start
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM2_start$prototype" data-type="code">

    void
    MSS_TIM2_start
    (
        TIMER_TypeDef * timer
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM2_start$description" data-type="text">

MSS_TIM2_start() enables Timer 2 and starts its down-counter, decrementing from
the load_value specified in previous calls to MSS_TIM2_load_immediate() or
MSS_TIM2_load_background().

Note: MSS_TIM2_start() is also used to resume the down-counter if previously
stopped using MSS_TIM2_stop().

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM2_start$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM2_start$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim2stop"></a>
## MSS_TIM2_stop
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM2_stop$prototype" data-type="code">

    void
    MSS_TIM2_stop
    (
        TIMER_TypeDef * timer
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM2_stop$description" data-type="text">

MSS_TIM2_stop() disables Timer 2 and stops its down-counter.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM2_stop$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM2_stop$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim2getcurrentvalue"></a>
## MSS_TIM2_get_current_value
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM2_get_current_value$prototype" data-type="code">

    uint32_t
    MSS_TIM2_get_current_value
    (
        TIMER_TypeDef * timer
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM2_get_current_value$description" data-type="text">

MSS_TIM2_get_current_value() returns the current value of the Timer 2 down-
counter.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM2_get_current_value$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM2_get_current_value$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim2loadimmediate"></a>
## MSS_TIM2_load_immediate
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM2_load_immediate$prototype" data-type="code">

    void
    MSS_TIM2_load_immediate
    (
        TIMER_TypeDef * timer,
        uint32_t load_value
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM2_load_immediate$description" data-type="text">

MSS_TIM2_load_immediate() loads the value passed by the load_value parameter
into the Timer 2 down-counter. The counter decrements immediately from this
value once Timer 2 is enabled. The MSS Timer generates an interrupt when the
counter reaches zero if Timer 2 interrupts are enabled. This function is
intended to be used when Timer 2 is configured for One-shot mode to time a
single delay.

Note: The value passed by the load_value parameter is loaded immediately into
the down-counter regardless of whether Timer 2 is operating in Periodic or One-
shot mode.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM2_load_immediate$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

#### load_value
<div id="Functions$MSS_TIM2_load_immediate$description$parameters$load_value" data-type="text" data-name="load_value">

The load_value parameter specifies the value from which the Timer 2 down-counter
starts decrementing.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM2_load_immediate$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim2loadbackground"></a>
## MSS_TIM2_load_background
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM2_load_background$prototype" data-type="code">

    void
    MSS_TIM2_load_background
    (
        TIMER_TypeDef * timer,
        uint32_t load_value
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM2_load_background$description" data-type="text">

MSS_TIM2_load_background() specifies the value that is reloaded into the Timer 2
down-counter the next time the counter reaches zero. This function is typically
used when Timer 2 is configured for Periodic mode to select or change the delay
period between the interrupts generated by Timer 2.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM2_load_background$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

#### load_value
<div id="Functions$MSS_TIM2_load_background$description$parameters$load_value" data-type="text" data-name="load_value">

The load_value parameter specifies the value that is loaded into the Timer 2
down-counter the next time the down-counter reaches zero. The Timer 2 down-
counter starts decrementing from this value after the current count expires.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM2_load_background$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim2enableirq"></a>
## MSS_TIM2_enable_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM2_enable_irq$prototype" data-type="code">

    void
    MSS_TIM2_enable_irq
    (
        TIMER_TypeDef * timer
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM2_enable_irq$description" data-type="text">

MSS_TIM2_enable_irq() enables interrupt generation for Timer 2. This function
also enables the interrupt in the RISC-V PLIC. Timer2_IRQHandler() is called
when a Timer 2 interrupt occurs.

Note: A Timer2_IRQHandler() default implementation is defined, with weak
linkage, in the MPFS HAL. You must provide your own implementation of
Timer2_IRQHandler(), which overrides the default implementation, to suit your
application.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM2_enable_irq$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM2_enable_irq$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim2disableirq"></a>
## MSS_TIM2_disable_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM2_disable_irq$prototype" data-type="code">

    void
    MSS_TIM2_disable_irq
    (
        TIMER_TypeDef * timer
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM2_disable_irq$description" data-type="text">

MSS_TIM2_disable_irq() disables interrupt generation for Timer 2. This function
also disables the interrupt in the RISC-V PLIC.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM2_disable_irq$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM2_disable_irq$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim2clearirq"></a>
## MSS_TIM2_clear_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM2_clear_irq$prototype" data-type="code">

    void
    MSS_TIM2_clear_irq
    (
        TIMER_TypeDef * timer
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM2_clear_irq$description" data-type="text">

MSS_TIM2_clear_irq() clears a pending interrupt from Timer 2. This function also
clears the interrupt in the RISC-V PLIC.

Note: You must call MSS_TIM2_clear_irq() as part of your implementation of the
Timer2_IRQHandler() Timer 2 Interrupt Service Routine (ISR) in order to prevent
the same interrupt event retriggering a call to the ISR.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM2_clear_irq$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM2_clear_irq$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim64init"></a>
## MSS_TIM64_init
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM64_init$prototype" data-type="code">

    void
    MSS_TIM64_init
    (
        TIMER_TypeDef * timer,
        mss_timer_mode_t mode
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM64_init$description" data-type="text">

MSS_TIM64_init() initializes the MSS Timer block for use as a single 64-bit
timer and selects the operating mode of the timer. The MSS Timer block is
already out of reset before executing MSS_TIM64_init(). This function stops the
timer, disables its interrupts, and sets the timer's operating mode.

Note: The MSS Timer block cannot be used both as a 64-bit and 32-bit timer. When
the MSS_TIM64_init() function is invoked, it overwrites any previous
configuration of the MSS Timer as a 32-bit timer.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM64_init$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

#### mode
<div id="Functions$MSS_TIM64_init$description$parameters$mode" data-type="text" data-name="mode">

The mode parameter specifies whether the timer operates in Periodic or One-shot
mode.

  - Following are the allowed values:
      - MSS_TIMER_PERIODIC_MODE
      - MSS_TIMER_ONE_SHOT_MODE


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM64_init$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim64start"></a>
## MSS_TIM64_start
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM64_start$prototype" data-type="code">

    void
    MSS_TIM64_start
    (
        TIMER_TypeDef * timer
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM64_start$description" data-type="text">

MSS_TIM64_start() enables the 64-bit timer and starts its down-counter,
decrementing from the load_value specified in previous calls to
MSS_TIM64_load_immediate() or MSS_TIM64_load_background().

Note: MSS_TIM64_start() is also used to resume the down-counter if previously
stopped using MSS_TIM64_stop().

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM64_start$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM64_start$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim64stop"></a>
## MSS_TIM64_stop
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM64_stop$prototype" data-type="code">

    void
    MSS_TIM64_stop
    (
        TIMER_TypeDef * timer
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM64_stop$description" data-type="text">

MSS_TIM64_stop() disables the 64-bit timer and stops its down-counter.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM64_stop$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM64_stop$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim64getcurrentvalue"></a>
## MSS_TIM64_get_current_value
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM64_get_current_value$prototype" data-type="code">

    void
    MSS_TIM64_get_current_value
    (
        TIMER_TypeDef * timer,
        uint32_t * load_value_u,
        uint32_t * load_value_l
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM64_get_current_value$description" data-type="text">

MSS_TIM64_get_current_value() is used to read the current value of the 64-bit
timer down-counter.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM64_get_current_value$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

#### load_value_u
<div id="Functions$MSS_TIM64_get_current_value$description$parameters$load_value_u" data-type="text" data-name="load_value_u">

The load_value_u parameter is a pointer to a 32-bit variable where the upper 32
bits of the current value of the 64-bit timer down-counter are copied.


</div>

#### load_value_l
<div id="Functions$MSS_TIM64_get_current_value$description$parameters$load_value_l" data-type="text" data-name="load_value_l">

The load_value_l parameter is a pointer to a 32-bit variable where the lower 32
bits of the current value of the 64-bit timer down-counter are copied.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM64_get_current_value$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$MSS_TIM64_get_current_value$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$MSS_TIM64_get_current_value$description$example$Example1" data-type="code" data-name="Example">


```
    uint32_t current_value_u = 0;
    uint32_t current_value_l = 0;
    MSS_TIM64_get_current_value(&current_value_u, &current_value_l);
```

</div>


 -------------------------------- 
<a name="msstim64loadimmediate"></a>
## MSS_TIM64_load_immediate
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM64_load_immediate$prototype" data-type="code">

    void
    MSS_TIM64_load_immediate
    (
        TIMER_TypeDef * timer,
        uint32_t load_value_u,
        uint32_t load_value_l
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM64_load_immediate$description" data-type="text">

MSS_TIM64_load_immediate() loads the values passed by the load_value_u and
load_value_l parameters into the 64-bit timer down-counter. The counter
decrements immediately from the concatenated 64-bit value once the 64-bit timer
is enabled. The MSS Timer generates an interrupt when the counter reaches zero
if 64-bit timer interrupts are enabled. This function is intended to be used
when the 64-bit timer is configured for One-shot mode to time a single delay.

Note: The value passed by the load_value parameter is loaded immediately into
the down-counter regardless of whether the 64-bit timer is operating in Periodic
or One-shot mode.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM64_load_immediate$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

#### load_value_u
<div id="Functions$MSS_TIM64_load_immediate$description$parameters$load_value_u" data-type="text" data-name="load_value_u">

The load_value_u parameter specifies the upper 32 bits of the 64-bit timer load
value from which the 64-bit timer down-counter starts decrementing.


</div>

#### load_value_l
<div id="Functions$MSS_TIM64_load_immediate$description$parameters$load_value_l" data-type="text" data-name="load_value_l">

The load_value_l parameter specifies the lower 32 bits of the 64-bit timer load
value from which the 64-bit timer down-counter starts decrementing.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM64_load_immediate$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim64loadbackground"></a>
## MSS_TIM64_load_background
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM64_load_background$prototype" data-type="code">

    void
    MSS_TIM64_load_background
    (
        TIMER_TypeDef * timer,
        uint32_t load_value_u,
        uint32_t load_value_l
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM64_load_background$description" data-type="text">

MSS_TIM64_load_background() specifies the 64-bit value that reloads into the
64-bit timer down-counter the next time the counter reaches zero. This function
is typically used when the 64-bit timer is configured for Periodic mode to
select or change the delay period between the interrupts generated by the 64-bit
timer.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM64_load_background$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

#### load_value_u
<div id="Functions$MSS_TIM64_load_background$description$parameters$load_value_u" data-type="text" data-name="load_value_u">

The load_value_u parameter specifies the upper 32 bits of the 64-bit timer load
value. The concatenated 64-bit value formed from load_value_u and load_value_l
loads into the 64-bit timer down-counter the next time the down-counter reaches
zero. The 64-bit timer down-counter starts decrementing from the concatenated
64-bit value after the current count expires.


</div>

#### load_value_l
<div id="Functions$MSS_TIM64_load_background$description$parameters$load_value_l" data-type="text" data-name="load_value_l">

The load_value_l parameter specifies the lower 32 bits of the 64-bit timer load
value. The concatenated 64-bit value formed from load_value_u and load_value_l
loads into the 64-bit timer down-counter the next time the down-counter reaches
zero. The 64-bit timer down-counter starts decrementing from the concatenated
64-bit value after the current count expires.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM64_load_background$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim64enableirq"></a>
## MSS_TIM64_enable_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM64_enable_irq$prototype" data-type="code">

    void
    MSS_TIM64_enable_irq
    (
        TIMER_TypeDef * timer
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM64_enable_irq$description" data-type="text">

MSS_TIM64_enable_irq() enables interrupt generation for the 64-bit timer. This
function also enables the interrupt in the RISC-V PLIC. Timer1_IRQHandler() is
called when a 64-bit timer interrupt occurs.

Note: A Timer1_IRQHandler() default implementation is defined, with weak
linkage, in the MPFS HAL. You must provide your own implementation of
Timer1_IRQHandler(), which overrides the default implementation, to suit your
application.

Note: MSS_TIM64_enable_irq() enables and uses Timer 1 interrupts for the 64-bit
timer. Timer 2 interrupts remain disabled.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM64_enable_irq$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM64_enable_irq$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim64disableirq"></a>
## MSS_TIM64_disable_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM64_disable_irq$prototype" data-type="code">

    void
    MSS_TIM64_disable_irq
    (
        TIMER_TypeDef * timer
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM64_disable_irq$description" data-type="text">

MSS_TIM64_disable_irq() disables interrupt generation for the 64-bit timer. This
function also disables the interrupt in the RISC-V PLIC.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM64_disable_irq$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM64_disable_irq$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msstim64clearirq"></a>
## MSS_TIM64_clear_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_TIM64_clear_irq$prototype" data-type="code">

    void
    MSS_TIM64_clear_irq
    (
        TIMER_TypeDef * timer
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_TIM64_clear_irq$description" data-type="text">

MSS_TIM64_clear_irq() clears a pending interrupt from the 64-bit timer. This
function also clears the interrupt in the RISC-V PLIC.

Note: You must call MSS_TIM64_clear_irq() as part of your implementation of the
Timer1_IRQHandler() 64-bit timer Interrupt Service Routine (ISR) in order to
prevent the same interrupt event retriggering a call to the ISR.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### timer
<div id="Functions$MSS_TIM64_clear_irq$description$parameters$timer" data-type="text" data-name="timer">

The timer parameter specifies the Timer block being used.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_TIM64_clear_irq$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
