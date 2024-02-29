<html>
 
 ------------------------------------ 

# CorePWM Bare Metal Driver.
 
-----------------------------------------
## Table of Contents 
  - [Introduction](#introduction)

  - [Driver Configuration](#driver-configuration)

     - [Configured PWM Register Widths](#configured-pwm-register-widths)

     - [Fixed PWM Period or Prescale Register Values](#fixed-pwm-period-or-prescale-register-values)

     - [Fixed PWM Positive or Negative Edge Register Values](#fixed-pwm-positive-or-negative-edge-register-values)

     - [Synchronized Update of PWM Output Waveforms](#synchronized-update-of-pwm-output-waveforms)

  - [Theory of Operation](#theory-of-operation)

     - [Initialization](#initialization)

     - [PWM Output Control](#pwm-output-control)

     - [Tachometer Control](#tachometer-control)

     - [Interrupt Control](#interrupt-control)

- [Types](#types)
  - [pwm_id_t](#pwmidt)
  - [pwm_tach_id_t](#pwmtachidt)
  - [pwm_wave_align_t](#pwmwavealignt)
  - [pwm_tach_prescale_t](#pwmtachprescalet)
  - [pwm_instance_t](#pwminstancet)

- [Constants](#constants)
  - [Tachometer Measurement Mode.](#tachometer-measurement-mode.)

- [Functions](#functions)
  - [PWM_init](#pwminit)
  - [PWM_enable](#pwmenable)
  - [PWM_disable](#pwmdisable)
  - [PWM_enable_synch_update](#pwmenablesynchupdate)
  - [PWM_disable_synch_update](#pwmdisablesynchupdate)
  - [PWM_set_duty_cycle](#pwmsetdutycycle)
  - [PWM_set_edges](#pwmsetedges)
  - [PWM_get_duty_cycle](#pwmgetdutycycle)
  - [PWM_generate_aligned_wave](#pwmgeneratealignedwave)
  - [PWM_enable_stretch_pulse](#pwmenablestretchpulse)
  - [PWM_disable_stretch_pulse](#pwmdisablestretchpulse)
  - [PWM_tach_init](#pwmtachinit)
  - [PWM_tach_set_mode](#pwmtachsetmode)
  - [PWM_tach_read_value](#pwmtachreadvalue)
  - [PWM_tach_clear_status](#pwmtachclearstatus)
  - [PWM_tach_read_status](#pwmtachreadstatus)
  - [PWM_tach_get_irq_source](#pwmtachgetirqsource)
  - [PWM_tach_enable_irq](#pwmtachenableirq)
  - [PWM_tach_disable_irq](#pwmtachdisableirq)
  - [PWM_tach_clear_irq](#pwmtachclearirq)

<div id="TitlePage" data-type="text">

# Introduction
The CorePWM hardware IP includes up to 16 Pulse Width Modulated (PWM) outputs
and up to 16 tachometer inputs. The CorePWM bare metal software driver is
designed for use in systems with no operating system.

The CorePWM driver provides:

  - Functions to control the duty cycle of each independent PWM output. The duty
 cycle control functions are identical for both the General Purpose PWM and Low 
Ripple DAC modes of CorePWM.
  - A function to control the positive and negative edges of each independent 
PWM output. This function can only be used for CorePWM outputs configured as 
General Purpose PWM. This function is useful for controlling the phase between 
different PWM outputs.
  - A function to generate left, centre, or right aligned PWM output waveforms.
  - Functions to enable and disable synchronous update of the PWM output 
waveforms.
  - Functions to enable and disable pulse stretching of the PWM output 
waveforms.
  - Functions to configure and control the tachometer and measure the period of 
tachometer input signals.
  - Functions to control tachometer interrupts.

# Driver Configuration
The CorePWM driver is configured through calling the PWM_init() function for
each CorePWM instance in the hardware design. The configuration parameters
include the CorePWM hardware instance base address and other runtime parameters,
such as the PWM clock prescale and period.

No CorePWM hardware configuration parameters are used by the driver, apart from
the CorePWM hardware instance base address. Hence, no additional configuration
files are required to use the driver.

## Configured PWM Register Widths
Each CorePWM instance in the hardware design is configured for the width of the
APB data bus, by the APB_DWIDTH configuration parameter. This parameter also
configures the width of CorePWM's PRESCALE, PERIOD, PWMx_POSEDGE, PWMx_NEGEDGE,
and DACx_LEVELOUT registers. The CorePWM driver's PWM_init(),
PWM_set_duty_cycle(), PWM_set_edges(), and PWM_generate_aligned_wave() functions
write values to these registers through one or more of the parameters prescale,
period, duty_cycle, pos_edge, or neg_edge - all of type uint32_t. It is your
responsibility to ensure that the values passed by these parameters are within
the range 0 to (2^APB_DWIDTH - 1), where APB_DWIDTH is 8, 16, or 32 dependent
upon your CorePWM hardware configuration, The return value from the
PWM_get_duty_cycle() function is also within the range 0 to (2^APB_DWIDTH - 1).

Note: Failure to keep the prescale, period, duty_cycle, pos_edge, and neg_edge
parameter values within the range 0 to (2^APB_DWIDTH - 1) results in unintended
CorePWM register settings.

## Fixed PWM Period or Prescale Register Values
The prescale and period parameter values passed to the PWM_init() function may
not have any effect if fixed values were selected for the PRESCALE and PERIOD
registers in the hardware configuration of CorePWM. When fixed values are
selected for these registers, the driver cannot overwrite the fixed values.

## Fixed PWM Positive or Negative Edge Register Values
The pos_edge and neg_edge parameter values passed to the PWM_set_edges()
function, and the duty_cycle parameter value passed to the PWM_set_duty_cycle()
and PWM_generate_aligned_wave() functions, may not have the desired effect if
fixed values were selected for the PWMx_POSEDGE (positive edge) or PWMx_NEGEDGE
(negative edge) registers in the hardware configuration of CorePWM. When fixed
values are selected for these registers, the driver cannot overwrite the fixed
values.

## Synchronized Update of PWM Output Waveforms
The configuration of the CorePWM instance in the hardware design must enable the
shadow update register for each PWM channel that requires synchronous output
waveform updates. The PWM_enable_synch_update() and PWM_disable_synch_update()
functions only affect PWM channels that have their shadow update registers
enabled in hardware.

# Theory of Operation
The CorePWM software driver is designed to control the multiple instances of
CorePWM. Each instance of CorePWM in the hardware design is associated with a
single instance of the pwm_instance_t structure in the software.

You need to allocate memory for one unique pwm_instance_t structure instance for
each CorePWM hardware instance.The contents of these data structures are
initialized by calling the PWM_init() function. A pointer to the structure is
passed to subsequent driver functions in order to identify the CorePWM hardware
instance you wish to perform the requested operation on.

Note: Do not attempt to directly manipulate the content of pwm_instance_t
structures. This structure is only intended to be modified by the driver
function.

## Initialization
The PWM granularity is configured through the PWM_init() function prescale
parameter. The PWM granularity specifies the resolution of the PWM output
waveforms for the targeted CorePWM instance. It is also sometimes called the PWM
period count time base. It is defined by the following equation:

`PWM_GRANULARITY = SYS_CLOCK_PERIOD * ( prescale + 1 )`

Where SYS_CLOCK_PERIOD is the period of the system clock used to clock the
CorePWM hardware instance and prescale is the value of the prescale parameter
passed to the PWM_init() function.

The PWM period is configured through the PWM_init() function period parameter.
It specifies the period of the PWM output waveforms for the targeted CorePWM
instance. The PWM period is defined by the following equation:

`PWM_PERIOD = PWM_GRANULARITY * ( period + 1 )`

Note: The prescale and period values passed to the PWM_init() function may not
have any effect if fixed values were selected for the prescale and period in the
hardware configuration of CorePWM.

Note: The prescale and period parameters are not relevant to any PWM outputs
that were configured for Low Ripple DAC mode in the hardware configuration of
CorePWM. In this mode, their only role is to set the interval between
synchronized register updates when the shadow register is enabled for the PWM
output.

## PWM Output Control
The waveform for each PWM output is specified by calling PWM_set_duty_cycle() or
PWM_set_edges() functions. Each PWM output is configured as either a General
Purpose PWM output or a Low Ripple DAC output in the hardware configuration of
CorePWM. In General Purpose PWM mode, either of the PWM_set_duty_cycle() or
PWM_set_edges() functions may be used to specify the PWM output waveform. In Low
Ripple DAC mode, only the PWM_set_duty_cycle() function may be used to specify
the PWM output waveform. The duty cycle of the PWM is read by calling the
PWM_get_duty_cyle() function.

The waveform alignment of General Purpose PWM outputs is set by calling the
PWM_generate_aligned_wave() function.

The PWM_enable_synch_update() function is used to enable the synchronous update
of a selected group of PWM channels. The PWM_disable_synch_update() is used to
terminate the synchronous update cycle after at least one PWM period has
elapsed. In synchronous mode, the channel output waveforms are updated at the
beginning of the PWM period, which is useful for motor control and also keeps a
constant dead band space between channel waveforms. The configuration of the
CorePWM instance in the hardware design must enable the shadow update register
for each PWM channel that requires synchronous output waveform updates. When the
shadow register is enabled, the PWM_set_duty_cycle(), PWM_set_edges(), and
PWM_generate_aligned_wave() functions set the new output waveform edge values in
the channel's shadow register instead of directly in the edge registers. Then,
call the PWM_enable_synch_update() function to update the PWM channel's output
waveform at the beginning of the next PWM period. Finally, calling the
PWM_disable_synch_update() function completes the update cycle.

Note: The PWM_enable_synch_update() and PWM_disable_synch_update() functions
have no affect on any PWM channels that do not have their shadow update
registers configured in hardware. These channels are always updated immediately,
asynchronous to the PWM period.

Note: The pos_edge and neg_edge values passed to the PWM_set_edges() function,
and the duty_cycle value passed to the PWM_set_duty_cycle() and
PWM_generate_aligned_wave() functions, may not have the desired effect if fixed
values were selected for the positive or negative edge registers in the hardware
configuration of CorePWM.

A typical sequence of function calls for PWM outputs is:

  - Call the PWM_init() function to initialize the CorePWM instance and pass it 
the values for the prescale and period.
  - Call either PWM_set_duty_cycle() or PWM_set_edges(), or 
PWM_generate_aligned_wave() for each PWM output to specify the output waveform.
  - Call PWM_enable() to enable a PWM output.
  - Call PWM_disable() to disable a PWM output.
  - The function PWM_set_duty_cycle() and PWM_generate_aligned_wave() also 
enables the PWM output if it is not already enabled.

## Tachometer Control
CorePWM also provides a tachometer interface. During the hardware design, the
CorePWM configuration mode should be set to either 1 - (PWM and TACH mode) or 2-
(TACH only mode) which enables the tachometer interface. Each CorePWM hardware
instance supports up to 16 tachometer inputs.

The CorePWM tachometer is initialized through calling the PWM_tach_init()
function. Call the PWM_tach_init() function before any other CorePWM driver
tachometer functions to set up the tachometer prescale value, initialize the
measurement mode, and disable interrupts.

The PWM_tach_set_mode() function is used to configure the tachometer input
channels for continuous or one time measurement of the input signal period. The
PWM_tach_clear_status() and PWM_tach_read_status() functions are used to clear
and read the measurement status bits for the tachometer input channels. When the
status bit indicates that a new input signal period measurement is available,
the PWM_tach_read_value() function is used to read the measured signal period
value.

A typical sequence of function calls for measurement of a tachometer input
signal is:

  - Call the PWM_init() function to initialize the CorePWM instance and pass it 
the values for the prescale and period.
  - Call the PWM_tach_init() function to initialize a tachometer input and pass 
it a tachometer prescale value.
  - Call the PWM_tach_set_mode() function to set the tachometer measurement 
mode.
  - Call the PWM_tach_enable_irq() function, if interrupts are required.
  - Call the PWM_enable_stretch_pulse() function to enable pulse stretching of 
the PWM output.
  - Call the PWM_tach_clear_status() to trigger new measurement of the 
tachometer input signal period.
  - Call the PWM_tach_read_status()function, if interrupts are not used, to 
verify that input period value has been updated in the TACHPULSEDUR register.
  - Call the PWM_tach_read_value()function to read the measured period of 
tachometer input signal.
  - Call PWM_disable_stretch_pulse() function to disable pulse stretching of the
 PWM output and resume a previously programmed PWM output waveform pattern.

## Interrupt Control
Interrupts generated by CorePWM tachometer status changes are controlled using
the following functions:

  - PWM_tach_enable_irq()
  - PWM_tach_disable_irq()
  - PWM_tach_clear_irq()
  - PWM_tach_get_irq_source()

The PWM_tach_get_irq_source() function is used to identify which CorePWM
tachometer input channel has generated an interrupt, among all the tachometer
input channels that have interrupts enabled.

</div>


# Types

 ---------------- 
<a name="pwmidt"></a>
## pwm_id_t
<a name="prototype"></a>
### Prototype 

<div id="Types$pwm_id_t$prototype" data-type="code">

 ``` 
    typedef enum {
        PWM_1 = 1,
        PWM_2,
        PWM_3,
        PWM_4,
        PWM_5,
        PWM_6,
        PWM_7,
        PWM_8,
        PWM_9,
        PWM_10,
        PWM_11,
        PWM_12,
        PWM_13,
        PWM_14,
        PWM_15,
        PWM_16
    } pwm_id_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$pwm_id_t$description" data-type="text">

PWM identifiers. The identifiers defined in this enumeration are used to
identify individual PWM outputs. They are used as argument to most CorePWM
driver functions to identify the PWM controlled by a function call.


</div>


 --------------------------- 
<a name="pwmtachidt"></a>
## pwm_tach_id_t
<a name="prototype"></a>
### Prototype 

<div id="Types$pwm_tach_id_t$prototype" data-type="code">

 ``` 
    typedef enum {
        PWM_TACH_INVALID = 0,
        PWM_TACH_1,
        PWM_TACH_2,
        PWM_TACH_3,
        PWM_TACH_4,
        PWM_TACH_5,
        PWM_TACH_6,
        PWM_TACH_7,
        PWM_TACH_8,
        PWM_TACH_9,
        PWM_TACH_10,
        PWM_TACH_11,
        PWM_TACH_12,
        PWM_TACH_13,
        PWM_TACH_14,
        PWM_TACH_15,
        PWM_TACH_16
    } pwm_tach_id_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$pwm_tach_id_t$description" data-type="text">

The pwm_tach_id_t type is used to identify individual CorePWM tachometer inputs.
The inputs are used as an argument to most CorePWM driver functions to identify
the PWM tachometer input controlled by a function call.


</div>


 --------------------------- 
<a name="pwmwavealignt"></a>
## pwm_wave_align_t
<a name="prototype"></a>
### Prototype 

<div id="Types$pwm_wave_align_t$prototype" data-type="code">

 ``` 
    typedef enum {
        PWM_LEFT_ALIGN,
        PWM_CENTER_ALIGN,
        PWM_RIGHT_ALIGN
    } pwm_wave_align_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$pwm_wave_align_t$description" data-type="text">

The pwm_wave_align_t type is used to align the duty cycle of the PWM output
waveform within the period of the PWM output waveform. A value of this type is
used as an argument to the PWM_generate_aligned_wave() function.

Note: The duty cycle corresponds to the number of period ticks for which the PWM
output remains high.


</div>


 --------------------------- 
<a name="pwmtachprescalet"></a>
## pwm_tach_prescale_t
<a name="prototype"></a>
### Prototype 

<div id="Types$pwm_tach_prescale_t$prototype" data-type="code">

 ``` 
    typedef enum {
        TACH_PRESCALE_PCLK_DIV_1 = 0x0000,
        TACH_PRESCALE_PCLK_DIV_2 = 0x0001,
        TACH_PRESCALE_PCLK_DIV_4 = 0x0002,
        TACH_PRESCALE_PCLK_DIV_8 = 0x0003,
        TACH_PRESCALE_PCLK_DIV_16 = 0x0004,
        TACH_PRESCALE_PCLK_DIV_32 = 0x0005,
        TACH_PRESCALE_PCLK_DIV_64 = 0x0006,
        TACH_PRESCALE_PCLK_DIV_128 = 0x0007,
        TACH_PRESCALE_PCLK_DIV_256 = 0x0008,
        TACH_PRESCALE_PCLK_DIV_512 = 0x0009,
        TACH_PRESCALE_PCLK_DIV_1024 = 0x000A,
        TACH_PRESCALE_PCLK_DIV_2048 = 0x000B
    } pwm_tach_prescale_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$pwm_tach_prescale_t$description" data-type="text">

The pwm_tach_prescale_t type is used to select the PCLK prescale divisor for the
CorePWM tachometer. A value of this type is used as an argument to the
PWM_tach_init() function.


</div>


 --------------------------- 
<a name="pwminstancet"></a>
## pwm_instance_t
<a name="prototype"></a>
### Prototype 

<div id="Types$pwm_instance_t$prototype" data-type="code">

``` 
    typedef struct pwm_instance {
        addr_t address; 
    } pwm_instance_t ;
  ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$pwm_instance_t$description" data-type="text">

This structure is used to identify the various CorePWM hardware instances in
your system. Your application software should declare one instance of this
structure for each instance of CorePWM in your system. The function PWM_init()
initializes this structure. A pointer to an initialized instance of the
structure should be passed as the first parameter to the CorePWM driver
functions, to identify which CorePWM hardware instance should perform the
requested operation.


</div>


 --------------------------- 

# Constants

 ---------------- 
<div id="Constants$TACH_CONTINUOUS$description" data-type="text">

<a name="tachcontinuous"></a>
## Tachometer Measurement Mode.
CorePWM allows the tachometer input measurement mode to be configured. The
following constants are used as an argument to the PWM_tach_set_mode() function
to specify the tachometer input measurement mode.

| Constant    | Description     | 
| -----|-----|
| TACH_CONTINUOUS    | Continuous measurement mode     | 
| TACH_ONE_SHOT    | One time measurement mode    | 

</div>


# Functions

 ---------------- 
<a name="pwminit"></a>
## PWM_init
<a name="prototype"></a>
### Prototype 

<div id="Functions$PWM_init$prototype" data-type="code">

    void
    PWM_init
    (
        pwm_instance_t * pwm_inst,
        addr_t base_addr,
        uint32_t prescale,
        uint32_t period
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PWM_init$description" data-type="text">

The PWM_init() function initializes a CorePWM hardware instance and the data
structure associated with the CorePWM hardware instance. It disables all PWM
outputs and sets the prescale and period value for the PWM. This function should
be called before any other CorePWM driver functions.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### pwm_inst
<div id="Functions$PWM_init$description$parameters$pwm_inst" data-type="text" data-name="pwm_inst">

Pointer to a PWM_instance_t structure holding all relevant data associated with
the target CorePWM hardware instance. This pointer is used to identify the
target CorePWM hardware instance in subsequent calls to the CorePWM functions.


</div>

#### base_addr
<div id="Functions$PWM_init$description$parameters$base_addr" data-type="text" data-name="base_addr">

The base_address parameter is the base address in the processor's memory map for
the registers of the CorePWM hardware instance being initialized.


</div>

#### prescale
<div id="Functions$PWM_init$description$parameters$prescale" data-type="text" data-name="prescale">

The prescale parameter is used to specify the PWM period count time base. It
specifies the PWM granularity. The value of this parameter should be between 0
and (2^APB_DWIDTH - 1), where APB_DWIDTH is the value selected for the
APB_DWIDTH in the instantiation of the CorePWM DirectCore hardware instance.
`PWM_GRANULARITY = system_clock_period * (prescale + 1)`


</div>

#### period
<div id="Functions$PWM_init$description$parameters$period" data-type="text" data-name="period">

The period parameter specifies the period of the PWM cycles. The value of this
parameter should be between 1 and (2^APB_DWIDTH - 1), where APB_DWIDTH is the
value selected for the APB_DWIDTH in the instantiation of the CorePWM DirectCore
hardware instance. `PWM_PERIOD = PWM_GRANULARITY * (period + 1)`


</div>

<a name="return"></a>
### Return
<div id="Functions$PWM_init$description$return" data-type="text">

None


</div>

##### Example
<div id="Functions$PWM_init$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$PWM_init$description$example$Example1" data-type="code" data-name="Example">


```
    #define COREPWM_BASE_ADDR  0xC0000000
    #define PWM_PRESCALE       0
    #define PWM_PERIOD         10

    pwm_instance_t the_pwm;

    void system_init( void )
    {
         PWM_init( &the_pwm, COREPWM_BASE_ADDR, PWM_PRESCALE, PWM_PERIOD ) ;
    }
```

</div>


 -------------------------------- 
<a name="pwmenable"></a>
## PWM_enable
<a name="prototype"></a>
### Prototype 

<div id="Functions$PWM_enable$prototype" data-type="code">

    void
    PWM_enable
    (
        pwm_instance_t * pwm_inst,
        pwm_id_t pwm_id
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PWM_enable$description" data-type="text">

The PWM_enable() function enables the specified PWM output.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### pwm_inst
<div id="Functions$PWM_enable$description$parameters$pwm_inst" data-type="text" data-name="pwm_inst">

Pointer to a pwm_inst structure holding all relevant data associated with the
target CorePWM hardware instance. This pointer is used to identify the target
CorePWM hardware instance.


</div>

#### pwm_id
<div id="Functions$PWM_enable$description$parameters$pwm_id" data-type="text" data-name="pwm_id">

The pwm_id parameter identifies the target PWM output.


</div>

<a name="return"></a>
### Return
<div id="Functions$PWM_enable$description$return" data-type="text">

None


</div>

##### Example
<div id="Functions$PWM_enable$description$example$Example1" data-type="text" data-name="Example">
The following call enables PWM 1.


</div>

<div id="Functions$PWM_enable$description$example$Example1" data-type="code" data-name="Example">


```
    PWM_enable(&the_pwm, PWM_1);
```

</div>


 -------------------------------- 
<a name="pwmdisable"></a>
## PWM_disable
<a name="prototype"></a>
### Prototype 

<div id="Functions$PWM_disable$prototype" data-type="code">

    void
    PWM_disable
    (
        pwm_instance_t * pwm_inst,
        pwm_id_t pwm_id
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PWM_disable$description" data-type="text">

The PWM_disable() function disables the specified PWM output.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### pwm_inst
<div id="Functions$PWM_disable$description$parameters$pwm_inst" data-type="text" data-name="pwm_inst">

Pointer to a pwm_inst structure holding all relevant data associated with the
target CorePWM hardware instance. This pointer is used to identify the target
CorePWM hardware instance.


</div>

#### pwm_id
<div id="Functions$PWM_disable$description$parameters$pwm_id" data-type="text" data-name="pwm_id">

The pwm_id parameter identifies the target PWM output.


</div>

<a name="return"></a>
### Return
<div id="Functions$PWM_disable$description$return" data-type="text">

None


</div>

##### Example
<div id="Functions$PWM_disable$description$example$Example1" data-type="text" data-name="Example">
The following call disables PWM 1.


</div>

<div id="Functions$PWM_disable$description$example$Example1" data-type="code" data-name="Example">


```
    PWM_disable(&the_pwm, PWM_1);
```

</div>


 -------------------------------- 
<a name="pwmenablesynchupdate"></a>
## PWM_enable_synch_update
<a name="prototype"></a>
### Prototype 

<div id="Functions$PWM_enable_synch_update$prototype" data-type="code">

    void
    PWM_enable_synch_update
    (
        pwm_instance_t * pwm_inst
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PWM_enable_synch_update$description" data-type="text">

The PWM_enable_synch_update() function enables synchronous update of PWM
outputs. In synchronous mode, a selected group of PWM outputs are updated at the
beginning of the PWM period, which is useful for motor control and also keeps a
constant dead band space between output waveforms. Configuration updates for all
of the selected PWM outputs are synchronized to the beginning of the PWM period,
allowing precise updates, and maintaining phase alignments between outputs.

Note: The configuration of the CorePWM instance in the hardware design must
enable the shadow update register for each PWM channel that requires synchronous
output waveform updates. This function has no affect on any PWM channel that
does not have its shadow update register configured in hardware.

Note: The PWM_set_duty_cycle(), PWM_set_edges(), or PWM_generate_aligned_wave()
functions must be called to set the new output waveform edge values in the
channel's shadow register before calling the PWM_enable_synch_update() function
to enable the update.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### pwm_inst
<div id="Functions$PWM_enable_synch_update$description$parameters$pwm_inst" data-type="text" data-name="pwm_inst">

Pointer to a pwm_inst structure holding all relevant data associated with the
target CorePWM hardware instance. This pointer is used to identify the target
CorePWM hardware instance.


</div>

<a name="return"></a>
### Return
<div id="Functions$PWM_enable_synch_update$description$return" data-type="text">

None


</div>

##### Example
<div id="Functions$PWM_enable_synch_update$description$example$Example1" data-type="text" data-name="Example">
Enable synchronous update of the duty cycle for the PWM 1 and PWM 2 outputs.


</div>

<div id="Functions$PWM_enable_synch_update$description$example$Example1" data-type="code" data-name="Example">


```
    uint32_t duty_cycle = 2;
    PWM_set_duty_cycle( &the_pwm, PWM_1, duty_cycle );
    PWM_set_duty_cycle( &the_pwm, PWM_2, duty_cycle );
    PWM_enable_synch_update( &the_pwm );
    wait_more_than_one_period();
    PWM_disable_synch_update( &the_pwm );
```

</div>


 -------------------------------- 
<a name="pwmdisablesynchupdate"></a>
## PWM_disable_synch_update
<a name="prototype"></a>
### Prototype 

<div id="Functions$PWM_disable_synch_update$prototype" data-type="code">

    void
    PWM_disable_synch_update
    (
        pwm_instance_t * pwm_inst
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PWM_disable_synch_update$description" data-type="text">

The PWM_disable_synch_update() function disables synchronous update of PWM
outputs. The PWM_disable_synch_update() is used to terminate a synchronous
update cycle after at least one PWM period has elapsed.

Note: The configuration of the CorePWM instance in the hardware design must
enable the shadow update register for each PWM channel that requires synchronous
output waveform updates. This function has no affect on any PWM channel that
does not have its shadow update register configured in hardware.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### pwm_inst
<div id="Functions$PWM_disable_synch_update$description$parameters$pwm_inst" data-type="text" data-name="pwm_inst">

Pointer to a pwm_inst structure holding all relevant data associated with the
target CorePWM hardware instance. This pointer is used to identify the target
CorePWM hardware instance.


</div>

<a name="return"></a>
### Return
<div id="Functions$PWM_disable_synch_update$description$return" data-type="text">

None


</div>

##### Example
<div id="Functions$PWM_disable_synch_update$description$example$Example1" data-type="text" data-name="Example">
calling the PWM_disable_synch_update() below disables the synchronous update of
PWM outputs.


</div>

<div id="Functions$PWM_disable_synch_update$description$example$Example1" data-type="code" data-name="Example">


```
    PWM_enable_synch_update( &the_pwm );
    wait_more_than_one_period();
    PWM_disable_synch_update(&the_pwm);
```

</div>


 -------------------------------- 
<a name="pwmsetdutycycle"></a>
## PWM_set_duty_cycle
<a name="prototype"></a>
### Prototype 

<div id="Functions$PWM_set_duty_cycle$prototype" data-type="code">

    void
    PWM_set_duty_cycle
    (
        pwm_instance_t * pwm_inst,
        pwm_id_t pwm_id,
        uint32_t duty_cycle
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PWM_set_duty_cycle$description" data-type="text">

The PWM_set_duty_cycle() function is used for setting the duty cycle of a PWM
output.

Note: It also enables the PWM output if it is not already enabled.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### pwm_inst
<div id="Functions$PWM_set_duty_cycle$description$parameters$pwm_inst" data-type="text" data-name="pwm_inst">

Pointer to a pwm_inst structure holding all relevant data associated with the
target CorePWM hardware instance. This pointer is used to identify the target
CorePWM hardware instance.


</div>

#### pwm_id
<div id="Functions$PWM_set_duty_cycle$description$parameters$pwm_id" data-type="text" data-name="pwm_id">

The pwm_id parameter identifies the target PWM output.


</div>

#### duty_cycle
<div id="Functions$PWM_set_duty_cycle$description$parameters$duty_cycle" data-type="text" data-name="duty_cycle">

The duty_cycle parameter specifies the PWM output duty cycle.  
In General
Purpose PWM mode: This parameter corresponds to the number of period ticks for
which the PWM output remains high. The value of this parameter should be between
0 and the value of the period selected for the calling the PWM_init().  
In Low
Ripple DAC mode: This parameter corresponds to the average density duty cycle.
The value of this parameter should be between 0 and (2^APB_DWIDTH - 1), where
APB_DWIDTH is the value selected for the APB_DWIDTH in the instantiation of the
CorePWM DirectCore hardware instance. This sets the average density of the PWM
output high pulses to between 0% and 100% in proportion to the duty_cycle value
as a percentage of the (2^APB_DWIDTH - 1) value.  
Note: The only role that
prescale and period play in Low Ripple DAC mode is to set the point when
synchronized register updates will take place, if the shadow register is enabled
for a PWM output.


</div>

<a name="return"></a>
### Return
<div id="Functions$PWM_set_duty_cycle$description$return" data-type="text">

None


</div>

##### Example
<div id="Functions$PWM_set_duty_cycle$description$example$Example1" data-type="text" data-name="Example">
The following example sets the duty cycle of PWM 1 to the value 2.


</div>

<div id="Functions$PWM_set_duty_cycle$description$example$Example1" data-type="code" data-name="Example">


```
    uint32_t duty_cycle = 2;
    PWM_set_duty_cycle( &the_pwm, PWM_1, duty_cycle );
```

</div>


 -------------------------------- 
<a name="pwmsetedges"></a>
## PWM_set_edges
<a name="prototype"></a>
### Prototype 

<div id="Functions$PWM_set_edges$prototype" data-type="code">

    void
    PWM_set_edges
    (
        pwm_instance_t * pwm_inst,
        pwm_id_t pwm_id,
        uint32_t pos_edge,
        uint32_t neg_edge
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PWM_set_edges$description" data-type="text">

The PWM_set_edges() function configures the positive and negative edge of the
specified PWM output. The PWM output waveform is controlled by specifying the
value of period counter at which the output will rise and fall.

Note: The PWM_set_edges() function does not enable the PWM output. You must call
the PWM_enable() function to enable the PWM output, either before or after
calling the PWM_set_edges() function.

Note: If you specify the same value for both the positive edge and the negative
edge, this will set the PWM output waveform to toggle mode (50% duty cycle). The
PWM output waveform will toggle high or low in each succeeding PWM period when
the period counter reaches the edge value set by this function.

Note: The PWM_get_duty_cycle() function will return the value 0 when it is
called after using the PWM_set_edges() function to set the PWM output waveform
to toggle mode (50% duty cycle). A return value of 0 from the
PWM_get_duty_cycle() function normally means a 0% duty cycle, Therefore, be
alert to this exception if you use the PWM_set_edges() function to manipulate
the PWM output waveform edges.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### pwm_inst
<div id="Functions$PWM_set_edges$description$parameters$pwm_inst" data-type="text" data-name="pwm_inst">

Pointer to a pwm_inst structure holding all relevant data associated with the
target CorePWM hardware instance. This pointer is used to identify the target
CorePWM hardware instance.


</div>

#### pwm_id
<div id="Functions$PWM_set_edges$description$parameters$pwm_id" data-type="text" data-name="pwm_id">

The pwm_id parameter identifies the target PWM output.


</div>

#### pos_edge
<div id="Functions$PWM_set_edges$description$parameters$pos_edge" data-type="text" data-name="pos_edge">

The pos_edge parameter specifies the value of the period counter at which the
PWM output identified by pwm_id will rise from low to high. The value of this
parameter should be between 0 and the value of the period selected for calling
PWM_init().


</div>

#### neg_edge
<div id="Functions$PWM_set_edges$description$parameters$neg_edge" data-type="text" data-name="neg_edge">

The neg_edge parameter specifies the value of the period counter at which the
PWM output identified by pwm_id will fall from high to low. The value of this
parameter should be between 0 and the value of the period selected for calling
PWM_init().


</div>

<a name="return"></a>
### Return
<div id="Functions$PWM_set_edges$description$return" data-type="text">

None


</div>

##### Example
<div id="Functions$PWM_set_edges$description$example$Example1" data-type="text" data-name="Example">
The following example sets the positive and negative edges of PWM 1 to 0 and 2
respectively.


</div>

<div id="Functions$PWM_set_edges$description$example$Example1" data-type="code" data-name="Example">


```
    uint32_t pos_edge = 0;
    uint32_t neg_edge = 2;
    PWM_set_edges( &the_pwm, PWM_1, pos_edge, neg_edge );
```

</div>


 -------------------------------- 
<a name="pwmgetdutycycle"></a>
## PWM_get_duty_cycle
<a name="prototype"></a>
### Prototype 

<div id="Functions$PWM_get_duty_cycle$prototype" data-type="code">

    uint32_t
    PWM_get_duty_cycle
    (
        pwm_instance_t * pwm_inst,
        pwm_id_t pwm_id
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PWM_get_duty_cycle$description" data-type="text">

The PWM_get_duty_cycle() function returns the current duty cycle of the
specified PWM output. The duty cycle corresponds to the number of period ticks
during which the output remains high and should be between 0 and the value of
the period selected to call PWM_init().

Note: Duty Cycle (in %) is calculated as follows:  
`Duty Cycle (%) = (
(returned duty cycle value) / (period +1) )*100`

Note: The PWM_get_duty_cycle() function is intended to return the duty cycle
previously set by calling the PWM_set_duty_cycle() or
PWM_generate_aligned_wave() functions.

Note: A returned duty cycle value of 0 normally means a 0% duty cycle. However,
the PWM_get_duty_cycle() function also returns the value 0 when it is called
after using the PWM_set_edges() function to set the PWM output waveform to
toggle mode. In this case, a return value of 0 does not mean a 0% duty cycle;
rather, it means that the PWM output waveform is in toggle mode. See the
description of the PWM_set_edges() function for a description of toggle mode.
You must be alert to this exception if you use the PWM_set_edges() function to
manipulate the PWM output waveform edges.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### pwm_inst
<div id="Functions$PWM_get_duty_cycle$description$parameters$pwm_inst" data-type="text" data-name="pwm_inst">

Pointer to a pwm_inst structure holding all relevant data associated with the
target CorePWM hardware instance. This pointer is used to identify the target
CorePWM hardware instance.


</div>

#### pwm_id
<div id="Functions$PWM_get_duty_cycle$description$parameters$pwm_id" data-type="text" data-name="pwm_id">

The pwm_id parameter identifies the target PWM output.


</div>

<a name="return"></a>
### Return
<div id="Functions$PWM_get_duty_cycle$description$return" data-type="text">

This function returns the duty cycle of the PWM output as a 32-bit unsigned
integer. The duty cycle is the number of period ticks during which the output is
high.  
Note: A returned duty cycle value of 0 normally means the PWM output is
constantly low, However, it may also indicate that the PWM output waveform is
set to toggle mode (50% duty cycle). See the note in the function description.


</div>

##### Example
<div id="Functions$PWM_get_duty_cycle$description$example$Example1" data-type="text" data-name="Example">
Read and assigns current duty cycle value to a variable.


</div>

<div id="Functions$PWM_get_duty_cycle$description$example$Example1" data-type="code" data-name="Example">


```
    uint32_t duty_cycle ;
    duty_cycle = PWM_get_duty_cycle( &the_pwm, PWM_1 );
```

</div>


 -------------------------------- 
<a name="pwmgeneratealignedwave"></a>
## PWM_generate_aligned_wave
<a name="prototype"></a>
### Prototype 

<div id="Functions$PWM_generate_aligned_wave$prototype" data-type="code">

    void
    PWM_generate_aligned_wave
    (
        pwm_instance_t * pwm_inst,
        pwm_id_t pwm_id,
        uint32_t duty_cycle,
        pwm_wave_align_t alignment_type
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PWM_generate_aligned_wave$description" data-type="text">

The PWM_generate_aligned_wave() function aligns the duty_cycle of the PWM output
waveform within the period of the PWM output waveform. The following three PWM
output waveform alignment options are supported:

  - Left Aligned Waveform
  - Center Aligned Waveform
  - Right Aligned Waveform

The PWM_generate_aligned_wave() function sets the appropriate positive edge and
negative edge values to achieve the specified alignment for the PWM output
waveform.

Note: It also enables the PWM output if it is not already enabled.

The following diagram shows the possible waveform alignments: Assume PWM
Prescale = 0, so that PWM Period granularity = clock period, PWM Period = 4, and
Duty_cycle = 1.

![Timing Diagram](https://www.plantuml.com/plantuml/svg/TP5D3i8W64JtdE9BJs3_zfYwPD4G4zTeAILIcb3KU7jRcvHPwEx1pBoGmEUeGdoCZivsjDxGoIeJrCYkrglmfgnnq-sUaPgftU-4xYCTdJLUTtHHJzrFOVnsHDff7tNutMhsbhHc-AEIZF43QydED2mjnlFjuAONumLm2LpXe4x1gXAe4g0Ie1AW4gWIg6hsLWvSRBmjIWWgu3hnQf9Wac2Iy2QPh1K4TMB6jU-MJ_m0 "Timing Diagram")

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### pwm_inst
<div id="Functions$PWM_generate_aligned_wave$description$parameters$pwm_inst" data-type="text" data-name="pwm_inst">

Pointer to a pwm_inst structure holding all relevant data associated with the
target CorePWM hardware instance. This pointer is used to identify the target
CorePWM hardware instance.


</div>

#### pwm_id
<div id="Functions$PWM_generate_aligned_wave$description$parameters$pwm_id" data-type="text" data-name="pwm_id">

The pwm_id parameter identifies the target PWM output.


</div>

#### duty_cycle
<div id="Functions$PWM_generate_aligned_wave$description$parameters$duty_cycle" data-type="text" data-name="duty_cycle">

The duty_cycle parameter specifies the PWM output duty cycle. The duty cycle
corresponds to the number of period ticks for which the PWM output remains high.
The value of this parameter should be between 0 and the value of the period
selected to call PWM_init().


</div>

#### alignment_type
<div id="Functions$PWM_generate_aligned_wave$description$parameters$alignment_type" data-type="text" data-name="alignment_type">

The alignment_type parameter specifies required alignment for the PWM output
waveform. Allowed values of type pwm_wave_align_t are: PWM_LEFT_ALIGN
PWM_CENTER_ALIGN PWM_RIGHT_ALIGN


</div>

<a name="return"></a>
### Return
<div id="Functions$PWM_generate_aligned_wave$description$return" data-type="text">

None


</div>

##### Example
<div id="Functions$PWM_generate_aligned_wave$description$example$Example1" data-type="text" data-name="Example">
The following example sets up PWM channels 1, 2, and 3 to generate left, center,
and right aligned output waveforms respectively.


</div>

<div id="Functions$PWM_generate_aligned_wave$description$example$Example1" data-type="code" data-name="Example">


```
    PWM_generate_aligned_wave(&the_pwm, PWM_1, duty_cycle, LEFT_ALIGN);
    PWM_generate_aligned_wave(&the_pwm, PWM_2, duty_cycle, CENTER_ALIGN);
    PWM_generate_aligned_wave(&the_pwm, PWM_3, duty_cycle, RIGHT_ALIGN);
```

</div>


 -------------------------------- 
<a name="pwmenablestretchpulse"></a>
## PWM_enable_stretch_pulse
<a name="prototype"></a>
### Prototype 

<div id="Functions$PWM_enable_stretch_pulse$prototype" data-type="code">

    void
    PWM_enable_stretch_pulse
    (
        pwm_instance_t * pwm_inst,
        pwm_id_t pwm_id
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PWM_enable_stretch_pulse$description" data-type="text">

The PWM_enable_stretch_pulse() function enables pulse stretching for the
specified PWM output. To accurately measure the speed of 3-wire fans, it is
necessary to turn on the fan periodically for long enough to get a complete
measurement of the tachometer signal from the fan. This stretching of the PWM
output pulse for a long duration is referred as PWM pulse stretching. When pulse
stretching is enabled, the PWM output is set to high or low, dependent upon the
configuration of the CorePWM instance in the hardware design. The
PWM_disable_stretch_pulse() function must be used to disable PWM pulse
stretching once the fan speed measurement is completed.

Note: The configuration of the CorePWM instance in the hardware design must
select the PWM output level - high or low - that is set when PWM pulse
stretching is enabled. For example, to measure 3-wire fan speed, the CorePWM
hardware configuration should select a high stretch level for the PWM output.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### pwm_inst
<div id="Functions$PWM_enable_stretch_pulse$description$parameters$pwm_inst" data-type="text" data-name="pwm_inst">

Pointer to a pwm_inst structure holding all relevant data associated with the
target CorePWM hardware instance. This pointer is used to identify the target
CorePWM hardware instance.


</div>

#### pwm_id
<div id="Functions$PWM_enable_stretch_pulse$description$parameters$pwm_id" data-type="text" data-name="pwm_id">

The pwm_id parameter identifies the target PWM output.


</div>

<a name="return"></a>
### Return
<div id="Functions$PWM_enable_stretch_pulse$description$return" data-type="text">

None


</div>

##### Example
<div id="Functions$PWM_enable_stretch_pulse$description$example$Example1" data-type="text" data-name="Example">
The following function call enables the stretching of PWM output 1 pulse for
tachometer input measurement.


</div>

<div id="Functions$PWM_enable_stretch_pulse$description$example$Example1" data-type="code" data-name="Example">


```
    PWM_enable_stretch_pulse(&the_pwm, PWM_1);
```

</div>


 -------------------------------- 
<a name="pwmdisablestretchpulse"></a>
## PWM_disable_stretch_pulse
<a name="prototype"></a>
### Prototype 

<div id="Functions$PWM_disable_stretch_pulse$prototype" data-type="code">

    void
    PWM_disable_stretch_pulse
    (
        pwm_instance_t * pwm_inst,
        pwm_id_t pwm_id
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PWM_disable_stretch_pulse$description" data-type="text">

The PWM_disable_stretch_pulse() function disables pulse stretching for the
specified PWM output. This function must be called once your application has
completed the measurement of the tachometer input. When pulse stretching is
disabled, the PWM output resumes generation of its previously programmed output
waveform pattern.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### pwm_inst
<div id="Functions$PWM_disable_stretch_pulse$description$parameters$pwm_inst" data-type="text" data-name="pwm_inst">

Pointer to a pwm_inst structure holding all relevant data associated with the
target CorePWM hardware instance. This pointer is used to identify the target
CorePWM hardware instance.


</div>

#### pwm_id
<div id="Functions$PWM_disable_stretch_pulse$description$parameters$pwm_id" data-type="text" data-name="pwm_id">

The pwm_id parameter identifies the target PWM output.


</div>

<a name="return"></a>
### Return
<div id="Functions$PWM_disable_stretch_pulse$description$return" data-type="text">

None


</div>

##### Example
<div id="Functions$PWM_disable_stretch_pulse$description$example$Example1" data-type="text" data-name="Example">
The following function call disables the stretching of the PWM 1 output pulse
after completing the tachometer input measurement.


</div>

<div id="Functions$PWM_disable_stretch_pulse$description$example$Example1" data-type="code" data-name="Example">


```
    PWM_disable_stretch_pulse(&the_pwm, PWM_1);
```

</div>


 -------------------------------- 
<a name="pwmtachinit"></a>
## PWM_tach_init
<a name="prototype"></a>
### Prototype 

<div id="Functions$PWM_tach_init$prototype" data-type="code">

    void
    PWM_tach_init
    (
        pwm_instance_t * pwm_inst,
        pwm_tach_prescale_t tach_prescale
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PWM_tach_init$description" data-type="text">

The PWM_tach_init() function initializes the configuration of all of the CorePWM
tachometer inputs. It sets the tachometer's TACHPRESCALE register to the value
specified by the tach_prescale parameter. It sets the tachometer measurement
mode to continuous measurement for all tachometer inputs. It disables the
interrupts for all tachometer inputs and clears any pending interrupts.

Note: This function should be called before any other CorePWM driver tachometer
functions.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### pwm_inst
<div id="Functions$PWM_tach_init$description$parameters$pwm_inst" data-type="text" data-name="pwm_inst">

Pointer to a pwm_inst structure holding all relevant data associated with the
target CorePWM hardware instance. This pointer is used to identify the target
CorePWM hardware instance.


</div>

#### tach_prescale
<div id="Functions$PWM_tach_init$description$parameters$tach_prescale" data-type="text" data-name="tach_prescale">

The tach_prescale value is used to select the PCLK prescale divisor for the
CorePWM tachometer. This determines the period counter time base for tachometer
measurements. Following are the values allowed for type pwm_tach_prescale_t:

  - TACH_PRESCALE_PCLK_DIV_1
  - TACH_PRESCALE_PCLK_DIV_2
  - TACH_PRESCALE_PCLK_DIV_4
  - TACH_PRESCALE_PCLK_DIV_8
  - TACH_PRESCALE_PCLK_DIV_16
  - TACH_PRESCALE_PCLK_DIV_32
  - TACH_PRESCALE_PCLK_DIV_64
  - TACH_PRESCALE_PCLK_DIV_128
  - TACH_PRESCALE_PCLK_DIV_256
  - TACH_PRESCALE_PCLK_DIV_512
  - TACH_PRESCALE_PCLK_DIV_1024
  - TACH_PRESCALE_PCLK_DIV_2048


</div>

<a name="return"></a>
### Return
<div id="Functions$PWM_tach_init$description$return" data-type="text">

None


</div>

##### Example
<div id="Functions$PWM_tach_init$description$example$Example1" data-type="text" data-name="Example">
The following call to PWM_tach_init() initializes the CorePWM tachometer and
sets the PCLK prescale divisor to 8 for tachometer measurements.


</div>

<div id="Functions$PWM_tach_init$description$example$Example1" data-type="code" data-name="Example">


```
    pwm_tach_prescale_t tach_prescale = TACH_PRESCALE_PCLK_DIV_8;
    PWM_tach_init(&the_pwm, tach_prescale);
```

</div>


 -------------------------------- 
<a name="pwmtachsetmode"></a>
## PWM_tach_set_mode
<a name="prototype"></a>
### Prototype 

<div id="Functions$PWM_tach_set_mode$prototype" data-type="code">

    void
    PWM_tach_set_mode
    (
        pwm_instance_t * pwm_inst,
        pwm_tach_id_t pwm_tach_id,
        uint16_t pwm_tachmode
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PWM_tach_set_mode$description" data-type="text">

The PWM_tach_set_mode() function sets the measurement mode for the specified
tachometer input. This function selects how frequently the tachometer input
should be measured. There are two options for the measurement mode:

  - Continuous measurement of tachometer input
  - One time measurement of tachometer input

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### pwm_inst
<div id="Functions$PWM_tach_set_mode$description$parameters$pwm_inst" data-type="text" data-name="pwm_inst">

Pointer to a pwm_inst structure holding all relevant data associated with the
target CorePWM hardware instance. This pointer is used to identify the target
CorePWM hardware instance.


</div>

#### pwm_tach_id
<div id="Functions$PWM_tach_set_mode$description$parameters$pwm_tach_id" data-type="text" data-name="pwm_tach_id">

The pwm_tach_id parameter identifies the target tachometer input.


</div>

#### pwm_tachmode
<div id="Functions$PWM_tach_set_mode$description$parameters$pwm_tachmode" data-type="text" data-name="pwm_tachmode">

The pwm_tachmode parameter is used to specify the tachometer input measurement
mode. Following values are allowed for pwm_tachmode:

  - TACH_CONTINUOUS
  - TACH_ONE_SHOT


</div>

<a name="return"></a>
### Return
<div id="Functions$PWM_tach_set_mode$description$return" data-type="text">

None


</div>

##### Example
<div id="Functions$PWM_tach_set_mode$description$example$Example1" data-type="text" data-name="Example">
The following example sets up tachometer input 1 for one time measurement.


</div>

<div id="Functions$PWM_tach_set_mode$description$example$Example1" data-type="code" data-name="Example">


```
    PWM_tach_set_mode(&the_pwm, PWM_TACH_1,TACH_ONE_SHOT);
```

</div>


 -------------------------------- 
<a name="pwmtachreadvalue"></a>
## PWM_tach_read_value
<a name="prototype"></a>
### Prototype 

<div id="Functions$PWM_tach_read_value$prototype" data-type="code">

    uint16_t
    PWM_tach_read_value
    (
        pwm_instance_t * pwm_inst,
        pwm_tach_id_t pwm_tach_id
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PWM_tach_read_value$description" data-type="text">

The PWM_tach_read_value() function reads the measured period of the tachometer
input signal from the TACHPULSEDUR register corresponding to the specified
tachometer input. The value returned is the number of timer ticks between two
successive positive (or negative) edges of the tachometer input signal.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### pwm_inst
<div id="Functions$PWM_tach_read_value$description$parameters$pwm_inst" data-type="text" data-name="pwm_inst">

Pointer to a pwm_inst structure holding all relevant data associated with the
target CorePWM hardware instance. This pointer is used to identify the target
CorePWM hardware instance.


</div>

#### pwm_tach_id
<div id="Functions$PWM_tach_read_value$description$parameters$pwm_tach_id" data-type="text" data-name="pwm_tach_id">

The pwm_tach_id parameter identifies the target tachometer input.


</div>

<a name="return"></a>
### Return
<div id="Functions$PWM_tach_read_value$description$return" data-type="text">

It returns the number of timer ticks between two successive positive (or
negative) edges of the tachometer input signal as a 16-bit unsigned integer.


</div>

##### Example
<div id="Functions$PWM_tach_read_value$description$example$Example1" data-type="text" data-name="Example">
Read and assign the value of the current period measurement for tachometer input
1 to a variable.


</div>

<div id="Functions$PWM_tach_read_value$description$example$Example1" data-type="code" data-name="Example">


```
    uint16_t tach_input_value ;
    tach_input_value = PWM_tach_read_value(&the_pwm, PWM_TACH_1);
```

</div>


 -------------------------------- 
<a name="pwmtachclearstatus"></a>
## PWM_tach_clear_status
<a name="prototype"></a>
### Prototype 

<div id="Functions$PWM_tach_clear_status$prototype" data-type="code">

    void
    PWM_tach_clear_status
    (
        pwm_instance_t * pwm_inst,
        pwm_tach_id_t pwm_tach_id
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PWM_tach_clear_status$description" data-type="text">

The PWM_tach_clear_status() function clears the tachometer status bit for the
specified tachometer input. This allows the tachometer to take a new measurement
of the input signal period and store it in the TACHPULSEDUR register
corresponding to the specified tachometer input. This function is typically
called when the user want to take a new reading of the tachometer input signal
period.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### pwm_inst
<div id="Functions$PWM_tach_clear_status$description$parameters$pwm_inst" data-type="text" data-name="pwm_inst">

Pointer to a pwm_inst structure holding all relevant data associated with the
target CorePWM hardware instance. This pointer is used to identify the target
CorePWM hardware instance.


</div>

#### pwm_tach_id
<div id="Functions$PWM_tach_clear_status$description$parameters$pwm_tach_id" data-type="text" data-name="pwm_tach_id">

The pwm_tach_id parameter identifies the target tachometer input.


</div>

<a name="return"></a>
### Return
<div id="Functions$PWM_tach_clear_status$description$return" data-type="text">

None


</div>

##### Example
<div id="Functions$PWM_tach_clear_status$description$example$Example1" data-type="text" data-name="Example">
The following call clears the status bit for tachometer input 1 to begin a new
input period measurement.


</div>

<div id="Functions$PWM_tach_clear_status$description$example$Example1" data-type="code" data-name="Example">


```
    PWM_tach_clear_status(&the_pwm, PWM_TACH_1);
```

</div>


 -------------------------------- 
<a name="pwmtachreadstatus"></a>
## PWM_tach_read_status
<a name="prototype"></a>
### Prototype 

<div id="Functions$PWM_tach_read_status$prototype" data-type="code">

    uint16_t
    PWM_tach_read_status
    (
        pwm_instance_t * pwm_inst,
        pwm_tach_id_t pwm_tach_id
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PWM_tach_read_status$description" data-type="text">

The PWM_tach_read_status() function returns the current status for the specified
tachometer input. This function returns 1 if the tachometer input period
measurement has been updated at least once since the status bit was
cleared,otherwise, it returns 0.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### pwm_inst
<div id="Functions$PWM_tach_read_status$description$parameters$pwm_inst" data-type="text" data-name="pwm_inst">

Pointer to a pwm_inst structure holding all relevant data associated with the
target CorePWM hardware instance. This pointer is used to identify the target
CorePWM hardware instance.


</div>

#### pwm_tach_id
<div id="Functions$PWM_tach_read_status$description$parameters$pwm_tach_id" data-type="text" data-name="pwm_tach_id">

The pwm_tach_id parameter identifies the target tachometer input.


</div>

<a name="return"></a>
### Return
<div id="Functions$PWM_tach_read_status$description$return" data-type="text">

It returns 1 if the tachometer input period measurement has been updated at
least once since the status bit was cleared, otherwise, it returns 0.


</div>

##### Example
<div id="Functions$PWM_tach_read_status$description$example$Example1" data-type="text" data-name="Example">
Read and assign the current status of tachometer input 1 to a variable.


</div>

<div id="Functions$PWM_tach_read_status$description$example$Example1" data-type="code" data-name="Example">


```
    uint16_t tach_input_status ;
    tach_input_status = PWM_tach_read_status(&the_pwm, PWM_TACH_1);
```

</div>


 -------------------------------- 
<a name="pwmtachgetirqsource"></a>
## PWM_tach_get_irq_source
<a name="prototype"></a>
### Prototype 

<div id="Functions$PWM_tach_get_irq_source$prototype" data-type="code">

    pwm_tach_id_t
    PWM_tach_get_irq_source
    (
        pwm_instance_t * pwm_inst
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PWM_tach_get_irq_source$description" data-type="text">

The PWM_tach_get_irq_source() function identifies which tachometer input channel
has generated an interrupt, among all the tachometer input channels with
interrupts enabled.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### pwm_inst
<div id="Functions$PWM_tach_get_irq_source$description$parameters$pwm_inst" data-type="text" data-name="pwm_inst">

Pointer to a pwm_inst structure holding all relevant data associated with the
target CorePWM hardware instance. This pointer is used to identify the target
CorePWM hardware instance.


</div>

<a name="return"></a>
### Return
<div id="Functions$PWM_tach_get_irq_source$description$return" data-type="text">

It returns the pwm_tach_id_t identifier of the tachometer input generating the
interrupt. It returns 0, if no tachometer input is generating an interrupt.


</div>

##### Example
<div id="Functions$PWM_tach_get_irq_source$description$example$Example1" data-type="text" data-name="Example">
The following example returns the tachometer 1 identifier, PWM_TACH_1, when
PWM_tach_get_irq_source() is called, after the tachometer input 1 period
measurement has been completed by CorePWM.


</div>

<div id="Functions$PWM_tach_get_irq_source$description$example$Example1" data-type="code" data-name="Example">


```
    pwm_tach_id_t tach_input_no ;
    PWM_tach_enable_irq( &the_pwm, PWM_TACH_1);
    PWM_tach_clear_status( &the_pwm, PWM_TACH_1);
    tach_input_no = PWM_tach_get_irq_source( &the_pwm );
```

</div>


 -------------------------------- 
<a name="pwmtachenableirq"></a>
## PWM_tach_enable_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$PWM_tach_enable_irq$prototype" data-type="code">

    void
    PWM_tach_enable_irq
    (
        pwm_instance_t * pwm_inst,
        pwm_tach_id_t pwm_tach_id
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PWM_tach_enable_irq$description" data-type="text">

The PWM_tach_enable_irq() function enables interrupt generation for the
specified tachometer input channel.

Note: CorePWM asserts its interrupt output signal, when a tachometer input
channel's period measurement has been updated at least once since the channel's
interrupt was last cleared, if the channel is enabled for interrupt generation.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### pwm_inst
<div id="Functions$PWM_tach_enable_irq$description$parameters$pwm_inst" data-type="text" data-name="pwm_inst">

Pointer to a pwm_inst structure holding all relevant data associated with the
target CorePWM hardware instance. This pointer is used to identify the target
CorePWM hardware instance.


</div>

#### pwm_tach_id
<div id="Functions$PWM_tach_enable_irq$description$parameters$pwm_tach_id" data-type="text" data-name="pwm_tach_id">

The pwm_tach_id parameter identifies the target tachometer input.


</div>

<a name="return"></a>
### Return
<div id="Functions$PWM_tach_enable_irq$description$return" data-type="text">

None


</div>

##### Example
<div id="Functions$PWM_tach_enable_irq$description$example$Example1" data-type="text" data-name="Example">
Calling the PWM_tach_enable_irq() function allows tachometer input 1 to generate
an interrupt, when its input signal period measurement value is updated.


</div>

<div id="Functions$PWM_tach_enable_irq$description$example$Example1" data-type="code" data-name="Example">


```
    PWM_tach_enable_irq(&the_pwm, PWM_TACH_1);
```

</div>


 -------------------------------- 
<a name="pwmtachdisableirq"></a>
## PWM_tach_disable_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$PWM_tach_disable_irq$prototype" data-type="code">

    void
    PWM_tach_disable_irq
    (
        pwm_instance_t * pwm_inst,
        pwm_tach_id_t pwm_tach_id
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PWM_tach_disable_irq$description" data-type="text">

The PWM_tach_disable_irq() function disables interrupt generation for the
specified tachometer input channel.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### pwm_inst
<div id="Functions$PWM_tach_disable_irq$description$parameters$pwm_inst" data-type="text" data-name="pwm_inst">

Pointer to a pwm_inst structure holding all relevant data associated with the
target CorePWM hardware instance. This pointer is used to identify the target
CorePWM hardware instance.


</div>

#### pwm_tach_id
<div id="Functions$PWM_tach_disable_irq$description$parameters$pwm_tach_id" data-type="text" data-name="pwm_tach_id">

The pwm_tach_id parameter identifies the target tachometer input.


</div>

<a name="return"></a>
### Return
<div id="Functions$PWM_tach_disable_irq$description$return" data-type="text">

None


</div>

##### Example
<div id="Functions$PWM_tach_disable_irq$description$example$Example1" data-type="text" data-name="Example">
Calling the PWM_tach_disable_irq() function prevents tachometer input 1 from
generating an interrupt when its input signal period measurement value is
updated.


</div>

<div id="Functions$PWM_tach_disable_irq$description$example$Example1" data-type="code" data-name="Example">


```
    PWM_tach_disable_irq(&the_pwm, PWM_TACH_1);
```

</div>


 -------------------------------- 
<a name="pwmtachclearirq"></a>
## PWM_tach_clear_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$PWM_tach_clear_irq$prototype" data-type="code">

    void
    PWM_tach_clear_irq
    (
        pwm_instance_t * pwm_inst,
        pwm_tach_id_t pwm_tach_id
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PWM_tach_clear_irq$description" data-type="text">

The PWM_tach_clear_irq() function clears a pending interrupt generated by the
specified tachometer input channel.

Note: The PWM_tach_clear_irq() function must be called as part of a PWM
tachometer Interrupt Service Routine (ISR) in order to prevent the same
interrupt event retriggering a call to the PWM tachometer ISR.

Note: Interrupts may also need to be cleared in the processor's interrupt
controller.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### pwm_inst
<div id="Functions$PWM_tach_clear_irq$description$parameters$pwm_inst" data-type="text" data-name="pwm_inst">

Pointer to a pwm_inst structure holding all relevant data associated with the
target CorePWM hardware instance. This pointer is used to identify the target
CorePWM hardware instance.


</div>

#### pwm_tach_id
<div id="Functions$PWM_tach_clear_irq$description$parameters$pwm_tach_id" data-type="text" data-name="pwm_tach_id">

The pwm_tach_id parameter identifies the target tachometer input.


</div>

<a name="return"></a>
### Return
<div id="Functions$PWM_tach_clear_irq$description$return" data-type="text">

None


</div>

##### Example
<div id="Functions$PWM_tach_clear_irq$description$example$Example1" data-type="text" data-name="Example">
Calling the PWM_tach_clear_irq() function clears a pending interrupt from
tachometer input 1.


</div>

<div id="Functions$PWM_tach_clear_irq$description$example$Example1" data-type="code" data-name="Example">


```
    PWM_tach_clear_irq(&the_pwm, PWM_TACH_1);
```

</div>


 -------------------------------- 
