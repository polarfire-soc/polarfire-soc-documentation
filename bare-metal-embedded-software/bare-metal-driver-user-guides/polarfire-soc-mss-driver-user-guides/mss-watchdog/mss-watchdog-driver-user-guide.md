<html>
 
 ------------------------------------ 

# PolarFire® SoC MSS Watchdog Bare Metal Driver
 
-----------------------------------------
## Table of Contents 
  - [Introduction](#introduction)

  - [Hardware Flow Dependencies](#hardware-flow-dependencies)

  - [Theory of Operation](#theory-of-operation)

     - [Initialization and Configuration](#initialization-and-configuration)

     - [Reading the Watchdog Timer Value and Status](#reading-the-watchdog-timer-value-and-status)

     - [Refreshing the Watchdog Timer Value](#refreshing-the-watchdog-timer-value)

     - [Interrupt Control](#interrupt-control)

- [Types](#types)
  - [mss_watchdog_num_t](#msswatchdognumt)
  - [mss_watchdog_config_t](#msswatchdogconfigt)
  - [WATCHDOG_TypeDef](#watchdogtypedef)

- [Constants](#constants)
  - [Watchdog Generic constants](#watchdog-generic-constants)
      - [MSS_WDOG_ENABLE & MSS_WDOG_DISABLE](#msswdogenable-&-msswdogdisable)
      - [MSS_WDOG_REFRESH_KEY](#msswdogrefreshkey)
      - [MSS_WDOG_FORCE_RESET_KEY](#msswdogforceresetkey)

- [Functions](#functions)
  - [MSS_WD_get_config](#msswdgetconfig)
  - [MSS_WD_configure](#msswdconfigure)
  - [MSS_WD_reload](#msswdreload)
  - [MSS_WD_current_value](#msswdcurrentvalue)
  - [MSS_WD_forbidden_status](#msswdforbiddenstatus)
  - [MSS_WD_enable_mvrp_irq](#msswdenablemvrpirq)
  - [MSS_WD_disable_mvrp_irq](#msswddisablemvrpirq)
  - [MSS_WD_clear_timeout_irq](#msswdcleartimeoutirq)
  - [MSS_WD_clear_mvrp_irq](#msswdclearmvrpirq)
  - [MSS_WD_timeout_occured](#msswdtimeoutoccured)
  - [MSS_WD_force_reset](#msswdforcereset)

<div id="TitlePage" data-type="text">

# Introduction
The PolarFire SoC Microprocessor SubSystem (MSS) includes five instances of
watchdog timer that detects system lockups. This software driver provides a set
of functions to control the MSS Watchdog as part of a bare metal system where no
operating system is available. The driver can be adapted for use as part of an
operating system, but the implementation of the adaptation layer between the
driver and the operating system's driver model is outside the scope of this
driver.

The MSS Watchdog driver provides support for the following features:

  - Initialization of the MSS Watchdog
  - Reading the current value and status of the watchdog timer
  - Refreshing the watchdog timer value
  - Enabling, disabling, and clearing timeout and MVRP interrupts

# Hardware Flow Dependencies
The configuration of all the features of the PolarFire SoC MSS Watchdog is
covered by this driver. Besides, this driver does not require any other
configuration.

On PolarFire SoC, an AXI switch forms a bus matrix interconnect among multiple
masters and multiple slaves. Five RISC-V CPUs connect to the Master ports M10 to
M14 of the AXI switch. By default, all the APB peripherals are accessible on
AXI-Slave 5 of the AXI switch using the AXI to AHB and AHB to APB bridges
(referred as main APB bus). However, to support logical separation in the
Asymmetric Multi-Processing (AMP) mode of operation, the APB peripherals can
alternatively be accessed on the AXI-Slave 6 using the AXI to AHB and AHB to APB
bridges (referred as the AMP APB bus).

Application must make sure that the desired MSS Watchdog instance is
appropriately configured on one of the APB slaves described above by configuring
the PolarFire SoC System Registers (SYSREG) as per the application need. The
data structure corresponding to the desired MSS Watchdog instance must be
provided as parameter to the functions of this driver.

The base address and register addresses are defined in this driver as constants.
The interrupt number assignment for the MSS Watchdog peripherals are defined as
constants in the MPFS HAL. You must ensure that the latest MPFS HAL is included
in the project settings of the SoftConsole tool chain and that it is generated
into your project.

# Theory of Operation
The MSS watchdog driver functions are grouped into the following categories:

  - Initialization and configuration
  - Reading the current value and status of the watchdog timer
  - Refreshing the watchdog timer value
  - Support for enabling, disabling, and clearing time-out and MVRP interrupts

## Initialization and Configuration
The MSS Watchdog driver provides the MSS_WD_configure() function to configure
the MSS Watchdog with desired configuration values. It also provides the
MSS_WD_get_config() function to read back the current configurations of the MSS
Watchdog. Use this function to retrieve the current configurations and then
overwrite them with the application specific values, such as initial watchdog
timer value, Maximum Value (up to which) Refresh (is) Permitted, watchdog time-
out value, enable/disable forbidden region, enable/disable MVRP interrupt, and
interrupt type.

The occurrence of a time out event before the system reset is detected using the
MSS_WD_timeout_occured() function. This function is used at the start of the
application to detect whether the application is starting as a result of a
power-on reset or a watchdog reset. The time out event must be cleared by
calling the function MSS_WD_clear_timeout_event() to allow the detection of
subsequent time out events or differentiating between a RISC-V initiated system
reset and watchdog reset.

## Reading the Watchdog Timer Value and Status
MSS Watchdog is a down-counter. A refresh-forbidden window can be created by
configuring the watchdog Maximum Value up to which Refresh is Permitted (MVRP).
When the current value of the watchdog timer is greater than the MVRP value,
refreshing the watchdog is forbidden. Attempting to refresh the watchdog timer
in the forbidden window asserts a timeout interrupt. The
MSS_WD_forbidden_status() function is used to know whether the watchdog timer is
in forbidden window or has crossed it. By default, the forbidden window is
disabled and gets enabled by providing an appropriate value as parameter to the
MSS_WD_configure() function. When the forbidden window is disabled, the watchdog
timer can be refreshed at any time.

The current value of the watchdog timer can be read using the
MSS_WD_current_value() function. This function can be called at any time.

## Refreshing the Watchdog Timer Value
The watchdog timer value is refreshed using the MSS_WD_reload() function. The
value reloaded into the watchdog timer down-counter is specified at the
configuration time with an appropriate value as parameter to the
MSS_WD_get_config() function.

## Interrupt Control
The PolarFire SoC MSS Watchdog generates two interrupts, the MVRP interrupt and
the timeout interrupt.

The MVRP interrupt is generated when the watchdog down-counter crosses the MVRP.
Following functions are used to control MVRP interrupt:

  - MSS_WD_enable_mvrp_irq
  - MSS_WD_disable_mvrp_irq
  - MSS_WD_clear_mvrp_irq

The timeout interrupt is generated when the watchdog down-counter crosses the
watchdog timeout value. The timeout value is a non-zero value and it can be set
to a maximum value of 4095us. The non-maskable interrupt is generated when the
watchdog crosses this timeout value. The down-counter keeps on down counting and
a reset signal is generated when it reaches zero. Following functions are used
to control timeout interrupt:

  - MSS_WD_enable_timeout_irq
  - MSS_WD_disable_timeout_irq
  - MSS_WD_clear_timeout_irq 

</div>


# Types

 ---------------- 
<a name="msswatchdognumt"></a>
## mss_watchdog_num_t
<a name="prototype"></a>
### Prototype 

<div id="Types$mss_watchdog_num_t$prototype" data-type="code">

 ``` 
    typedef enum {
        MSS_WDOG0_LO = 0,
        MSS_WDOG1_LO = 1,
        MSS_WDOG2_LO = 2,
        MSS_WDOG3_LO = 3,
        MSS_WDOG4_LO = 4,
        MSS_WDOG0_HI = 5,
        MSS_WDOG1_HI = 6,
        MSS_WDOG2_HI = 7,
        MSS_WDOG3_HI = 8,
        MSS_WDOG4_HI = 9
    } mss_watchdog_num_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$mss_watchdog_num_t$description" data-type="text">

The mss_watchdog_num_t is the watchdog module number enumeration. MSS_WDOG0_LO
to MSS_WDOG4_LO correspond to the watchdog module number 0 to 4 when they appear
on the AXI switch slave 5. MSS_WDOG0_HI to MSS_WDOG4_HI correspond to the
watchdog module number 0 to 4 when they appear on the AXI switch slave 6.


</div>


 --------------------------- 
<a name="msswatchdogconfigt"></a>
## mss_watchdog_config_t
<a name="prototype"></a>
### Prototype 

<div id="Types$mss_watchdog_config_t$prototype" data-type="code">

``` 
    typedef struct mss_watchdog_config {
        uint32_t time_val; 
        uint32_t mvrp_val; 
        uint32_t timeout_val; 
        uint8_t forbidden_en; 
        uint8_t intr_type; 
    } mss_watchdog_config_t ;
  ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$mss_watchdog_config_t$description" data-type="text">

The mss_watchdog_config_t type is for the watchdog configuration structure. This
type is used as a parameter for the MSS_WD_configure() and the
MSS_WD_get_config() functions.

The following parameters are the values of this structure.

| Parameter    | Description     | 
| -----|-----|
| time_val    | The value from which the watchdog timer counts down     | 
| mvrp_val    | The watchdog MVRP value     | 
| timeout_val    | The watchdog timeout value     | 
| forbidden_en    | Enable/disable the forbidden window     | 
|   | When set, if a refresh occurs in the forbidden window,     | 
|   | the watchdog timeout interrupt will be generated.    | 

Time calculation example:

time_val = 0xFFFFF0u

mvrp_val = 0x989680u

timeout_val = 0x3e8u

A prescaler = 256 is used on the MSS AXI clock.

Considering AXI clock = 25 MHz:

The MVRP interrupt will happen after: (0xFFFFF0 - 0x989680) * (1/25MHz/256) =
69s after system reset

Timeout interrupt will happen after: (0xFFFFF0 - 0x3e8) * ( 1/25MHz/256) = 171s
after system reset


</div>


 --------------------------- 
<a name="watchdogtypedef"></a>
## WATCHDOG_TypeDef
<a name="prototype"></a>
### Prototype 

<div id="Types$WATCHDOG_TypeDef$prototype" data-type="code">

``` 
    typedef struct WATCHDOG_TypeDef {
        volatile uint32_t REFRESH; 
        volatile uint32_t CONTROL; 
        volatile uint32_t STATUS; 
        volatile uint32_t TIME; 
        volatile uint32_t MSVP; 
        volatile uint32_t TRIGGER; 
        volatile uint32_t FORCE; 
    } WATCHDOG_TypeDef ;
  ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$WATCHDOG_TypeDef$description" data-type="text">

The WATCHDOG_TypeDef is the hardware register structure for the PolarFire SoC
MSS Watchdog.


</div>


 --------------------------- 

# Constants

 ---------------- 
<div id="Constants$MSS_WDOG_ENABLE$description" data-type="text">

<a name="msswdogenable"></a>
## Watchdog Generic constants
<a name="msswdogenable-&-msswdogdisable"></a>
### MSS_WDOG_ENABLE & MSS_WDOG_DISABLE
The following constants can be used to configure the MSS Watchdog where a zero
or non-zero value, such as disable or enable, is provided as input parameter:

wd0lo_config.forbidden_en = MSS_WDOG_DISABLE;

MSS_WD_configure(MSS_WDOG0_LO, &wd0lo_config);

</div>

<div id="Constants$MSS_WDOG_REFRESH_KEY$description" data-type="text">

<a name="msswdogrefreshkey"></a>
<a name="msswdogrefreshkey"></a>
### MSS_WDOG_REFRESH_KEY
The MSS_WDOG_REFRESH_KEY macro holds the value that causes a reload of the
watchdog's down-counter when written to the watchdog's WDOGREFRESH register.

</div>

<div id="Constants$MSS_WDOG_FORCE_RESET_KEY$description" data-type="text">

<a name="msswdogforceresetkey"></a>
<a name="msswdogforceresetkey"></a>
### MSS_WDOG_FORCE_RESET_KEY
The MSS_WDOG_FORCE_RESET_KEY macro holds the value which will force a reset if
the watchdog is already timed out (gone past the timeout value). Writing any
other value or writing TRIGGER register at other times will trigger the watchdog
NMI sequence (that is, raise the timeout interrupt).

</div>


# Functions

 ---------------- 
<a name="msswdgetconfig"></a>
## MSS_WD_get_config
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_WD_get_config$prototype" data-type="code">

    void
    MSS_WD_get_config
    (
        mss_watchdog_num_t wd_num,
        mss_watchdog_config_t * config
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_WD_get_config$description" data-type="text">

The MSS_WD_get_config() function returns the current configuration of the
PolarFire SoC MSS Watchdog. The MSS Watchdog is pre-initialized by the flash
bits at the design time. When used for the first time before calling the
MSS_WD_configure() function, this function returns the default configuration as
configured at the design time. You can reconfigure the MSS Watchdog using
MSS_WD_configure() function. Calling the MSS_WD_get_config() function will then
return the current configuration values set by a previous call to
MSS_WD_configure() function. You may not need to use this function if you do not
want to know the current configuration. In that case, you can directly use the
MSS_WD_configure() function to configure the MSS Watchdog to the values of your
choice.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### wd_num
<div id="Functions$MSS_WD_get_config$description$parameters$wd_num" data-type="text" data-name="wd_num">

The wd_num parameter is the watchdog module number in the PolarFire SoC MSS on
which the operation needs to be performed. Choose the watchdog module number
using mss_watchdog_num_t. MSS_WDOG0_LO to MSS_WDOG4_LO correspond to the
watchdog modules 0 to 4 when they appear on the AXI switch slave 5. MSS_WDOG0_HI
to MSS_WDOG4_HI correspond to the watchdog modules 0 to 4 when they appear on
the AXI switch slave 6.


</div>

#### config
<div id="Functions$MSS_WD_get_config$description$parameters$config" data-type="text" data-name="config">

The config parameter is the parameter that stores the current configuration of
the watchdog module.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_WD_get_config$description$return" data-type="text">

This function does not return any value.


</div>

##### Example
<div id="Functions$MSS_WD_get_config$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$MSS_WD_get_config$description$example$Example1" data-type="code" data-name="Example">


```
    #include "mss_watchdog.h"
    mss_watchdog_config_t wd0lo_config;

    void e51( void )
    {
        MSS_WD_get_config(MSS_WDOG0_LO, &wd0lo_config);

        wd0lo_config.forbidden_en = WDOG_ENABLE;
        wd0lo_config.mvrp_val = 0xFFFF000u;

        MSS_WD_configure(MSS_WDOG0_LO, &wd0lo_config);

        for(;;)
        {
            main_task();
        }
    }
```

</div>


 -------------------------------- 
<a name="msswdconfigure"></a>
## MSS_WD_configure
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_WD_configure$prototype" data-type="code">

    uint8_t
    MSS_WD_configure
    (
        mss_watchdog_num_t wd_num,
        const mss_watchdog_config_t * config
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_WD_configure$description" data-type="text">

The MSS_WD_configure() function configures the desired watchdog module. The
watchdog module is pre-initialized by the flash bits at the design time to the
default values. You can reconfigure the watchdog module using MSS_WD_configure()
function.

Note: The MSS_WD_configure() function can be used only once, as it writes into
the TIME register. After performing the write operation into the TIME register,
the TIME, TRIGGER, and MSVP register values are frozen and can't be altered
again unless a system reset happens.

Note: The MSS Watchdog is not enabled at reset. Calling this function starts the
watchdog. It cannot be disabled then and must be refreshed periodically.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### wd_num
<div id="Functions$MSS_WD_configure$description$parameters$wd_num" data-type="text" data-name="wd_num">

The wd_num parameter is the watchdog module number in the PolarFire SoC MSS on
which the operation needs to be performed. Choose the watchdog module number
using mss_watchdog_num_t. MSS_WDOG0_LO to MSS_WDOG4_LO correspond to the
watchdog modules 0 to 4 when they appear on the AXI switch slave 5. MSS_WDOG0_HI
to MSS_WDOG4_HI correspond to the watchdog modules 0 to 4 when they appear on
the AXI switch slave 6.


</div>

#### config
<div id="Functions$MSS_WD_configure$description$parameters$config" data-type="text" data-name="config">

The config parameter is the input parameter where the configuration details
applied to the watchdog module is provided by the application. For more details,
see the description of mss_watchdog_config_t.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_WD_configure$description$return" data-type="text">

This function returns a zero value when executed successfully. A non-zero value
is returned when the configuration values are out of bound.


</div>

##### Example
<div id="Functions$MSS_WD_configure$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$MSS_WD_configure$description$example$Example1" data-type="code" data-name="Example">


```
    #include "mss_watchdog.h"
    mss_watchdog_config_t wd0lo_config;

    void e51( void )
    {
        MSS_WD_get_config(MSS_WDOG0_LO, &wd0lo_config);

        wd0lo_config.forbidden_en = WDOG_ENABLE;
        wd0lo_config.mvrp_val = 0xFFFF000u;

        MSS_WD_configure(MSS_WDOG0_LO, &wd0lo_config);

        for(;;)
        {
            main_task();
        }
    }
```

</div>


 -------------------------------- 
<a name="msswdreload"></a>
## MSS_WD_reload
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_WD_reload$prototype" data-type="code">

    void
    MSS_WD_reload
    (
        mss_watchdog_num_t wd_num
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_WD_reload$description" data-type="text">

The MSS_WD_reload() function causes the watchdog to reload its down-counter
timer with the load value configured through the MSS configurator in the
hardware flow. This function must be called regularly to avoid a system reset or
a timeout interrupt.

Note: The MSS Watchdog is not enabled at reset. Calling this function starts the
watchdog. It cannot be disabled then and must be refreshed periodically.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### wd_num
<div id="Functions$MSS_WD_reload$description$parameters$wd_num" data-type="text" data-name="wd_num">

The wd_num parameter is the watchdog module number in the PolarFire SoC MSS on
which the operation needs to be performed. Choose the watchdog module number
using mss_watchdog_num_t. MSS_WDOG0_LO to MSS_WDOG4_LO correspond to the
watchdog modules 0 to 4 when they appear on the AXI switch slave 5. MSS_WDOG0_HI
to MSS_WDOG4_HI correspond to the watchdog modules 0 to 4 when they appear on
the AXI switch slave 6.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_WD_reload$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msswdcurrentvalue"></a>
## MSS_WD_current_value
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_WD_current_value$prototype" data-type="code">

    uint32_t
    MSS_WD_current_value
    (
        mss_watchdog_num_t wd_num
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_WD_current_value$description" data-type="text">

The MSS_WD_current_value() function returns the current value of the watchdog's
down-counter.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### wd_num
<div id="Functions$MSS_WD_current_value$description$parameters$wd_num" data-type="text" data-name="wd_num">

The wd_num parameter is the watchdog module number in the PolarFire SoC MSS on
which the operation needs to be performed. Choose the watchdog module number
using mss_watchdog_num_t. MSS_WDOG0_LO to MSS_WDOG4_LO correspond to the
watchdog modules 0 to 4 when they appear on the AXI switch slave 5. MSS_WDOG0_HI
to MSS_WDOG4_HI correspond to the watchdog modules 0 to 4 when they appear on
the AXI switch slave 6.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_WD_current_value$description$return" data-type="text">

This function returns the current value of the watchdog’s down-counter as a
32-bit unsigned integer.


</div>


 -------------------------------- 
<a name="msswdforbiddenstatus"></a>
## MSS_WD_forbidden_status
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_WD_forbidden_status$prototype" data-type="code">

    uint32_t
    MSS_WD_forbidden_status
    (
        mss_watchdog_num_t wd_num
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_WD_forbidden_status$description" data-type="text">

The MSS_WD_forbidden_status() function returns the refresh status of the MSS
Watchdog.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### wd_num
<div id="Functions$MSS_WD_forbidden_status$description$parameters$wd_num" data-type="text" data-name="wd_num">

The wd_num parameter is the watchdog module number in the PolarFire SoC MSS on
which the operation needs to be performed. Choose the watchdog module number
using mss_watchdog_num_t. MSS_WDOG0_LO to MSS_WDOG4_LO correspond to the
watchdog modules 0 to 4 when they appear on the AXI switch slave 5. MSS_WDOG0_HI
to MSS_WDOG4_HI correspond to the watchdog modules 0 to 4 when they appear on
the AXI switch slave 6.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_WD_forbidden_status$description$return" data-type="text">

This function returns the refresh status of the watchdog. Value 1 indicates that
watchdog's down-counter is within the forbidden window and that a reload should
not be done. Value 0 indicates that the watchdog's down-counter is within the
permitted window and that a reload is allowed.


</div>


 -------------------------------- 
<a name="msswdenablemvrpirq"></a>
## MSS_WD_enable_mvrp_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_WD_enable_mvrp_irq$prototype" data-type="code">

    void
    MSS_WD_enable_mvrp_irq
    (
        mss_watchdog_num_t wd_num
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_WD_enable_mvrp_irq$description" data-type="text">

The MSS_WD_enable_mvrp_irq() function enables the MVRP interrupt. The MSS
Watchdog 0 to 4 generate a MVRP interrupt to hart 0 to 4 respectively. These
interrupts can be local or PLIC interrupts: depending on your choice, the
appropriate functions from the MPFS HAL must be called to enable the local or
PLIC interrupt. The corresponding interrupt handler is called when the interrupt
occurs.

Note: The watchdog MVRP interrupt handler default implementations are weakly
defined in the PolarFire SoC HAL. Provide your own implementation of these
functions that overrides the default implementation to suit your application.
See mss_ints.h in the MPFS HAL for the actual names and the prototypes of these
functions.

Note: This function must be called from appropriate hart context.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### wd_num
<div id="Functions$MSS_WD_enable_mvrp_irq$description$parameters$wd_num" data-type="text" data-name="wd_num">

The wd_num parameter is the watchdog module number in the PolarFire SoC MSS on
which the operation needs to be performed. Choose the watchdog module number
using mss_watchdog_num_t. MSS_WDOG0_LO to MSS_WDOG4_LO correspond to the
watchdog modules 0 to 4 when they appear on the AXI switch slave 5. MSS_WDOG0_HI
to MSS_WDOG4_HI correspond to the watchdog modules 0 to 4 when they appear on
the AXI switch slave 6.


</div>

#### intr_type
<div id="Functions$MSS_WD_enable_mvrp_irq$description$parameters$intr_type" data-type="text" data-name="intr_type">

The intr_type parameter indicates the type of interrupt that must be enabled.
The MVRP interrupt for each hart can either be local interrupt to that hart or
it can be accessed as a PLIC interrupt.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_WD_enable_mvrp_irq$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$MSS_WD_enable_mvrp_irq$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$MSS_WD_enable_mvrp_irq$description$example$Example1" data-type="code" data-name="Example">


```
    #include "mss_watchdog.h"
    void e51( void )
    {

        MSS_WD_enable_mvrp_irq(wd_num);
        for (;;)
        {
            main_task();
            cortex_sleep();
        }
    }

    void wdog0_mvrp_E51_local_IRQHandler_10(void)
    {
        process_timeout();
        MSS_WD_clear_mvrp_irq();
    }
```

</div>


 -------------------------------- 
<a name="msswddisablemvrpirq"></a>
## MSS_WD_disable_mvrp_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_WD_disable_mvrp_irq$prototype" data-type="code">

    __inline void
    MSS_WD_disable_mvrp_irq
    (
        mss_watchdog_num_t wd_num
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_WD_disable_mvrp_irq$description" data-type="text">

The MSS_WD_disable_mvrp_irq() function disables the generation of the MVRP
interrupt.

Note: This function must be called from appropriate hart context.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### wd_num
<div id="Functions$MSS_WD_disable_mvrp_irq$description$parameters$wd_num" data-type="text" data-name="wd_num">

The wd_num parameter is the watchdog module number in the PolarFire SoC MSS on
which the operation needs to be performed. Choose the watchdog module number
using mss_watchdog_num_t. MSS_WDOG0_LO to MSS_WDOG4_LO correspond to the
watchdog modules 0 to 4 when they appear on the AXI switch slave 5. MSS_WDOG0_HI
to MSS_WDOG4_HI correspond to the watchdog modules 0 to 4 when they appear on
the AXI switch slave 6.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_WD_disable_mvrp_irq$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msswdcleartimeoutirq"></a>
## MSS_WD_clear_timeout_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_WD_clear_timeout_irq$prototype" data-type="code">

    void
    MSS_WD_clear_timeout_irq
    (
        mss_watchdog_num_t wd_num
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_WD_clear_timeout_irq$description" data-type="text">

The MSS_WD_clear_timeout_irq() function clears the watchdog’s timeout interrupt,
which is connected to the RISC-V NMI interrupt. Calling
MSS_WD_clear_timeout_irq() results in clearing the RISC-V NMI interrupt.

Note: You must call the MSS_WD_clear_timeout_irq() function as part of your
implementation of the timeout handler to prevent the same interrupt event from
re-triggering the call to the timeout ISR.

Note: This function must be called from appropriate hart context.

Note: There is no MSS_WD_enable_timeout_irq() and MSS_WD_disable_timeout_irq()
functions because the timeout interrupt is permanently enabled by default and it
can not be disabled.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### wd_num
<div id="Functions$MSS_WD_clear_timeout_irq$description$parameters$wd_num" data-type="text" data-name="wd_num">

The wd_num parameter is the watchdog module number in the PolarFire SoC MSS on
which the operation needs to be performed. Choose the watchdog module number
using mss_watchdog_num_t. MSS_WDOG0_LO to MSS_WDOG4_LO correspond to the
watchdog modules 0 to 4 when they appear on the AXI switch slave 5. MSS_WDOG0_HI
to MSS_WDOG4_HI correspond to the watchdog modules 0 to 4 when they appear on
the AXI switch slave 6.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_WD_clear_timeout_irq$description$return" data-type="text">

This function does not return any value.


</div>


 -------------------------------- 
<a name="msswdclearmvrpirq"></a>
## MSS_WD_clear_mvrp_irq
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_WD_clear_mvrp_irq$prototype" data-type="code">

    void
    MSS_WD_clear_mvrp_irq
    (
        mss_watchdog_num_t wd_num
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_WD_clear_mvrp_irq$description" data-type="text">

The MSS_WD_clear_mvrp_irq() function clears the mvrp interrupt.

Note: You must call the MSS_WD_clear_mvrp_irq() function as part of your
implementation of the MVRP interrupt handler to prevent the same interrupt event
from re-triggering the call to the MVRP ISR.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### wd_num
<div id="Functions$MSS_WD_clear_mvrp_irq$description$parameters$wd_num" data-type="text" data-name="wd_num">

The wd_num parameter is the watchdog module number in the PolarFire SoC MSS on
which the operation needs to be performed. Choose the watchdog module number
using mss_watchdog_num_t. MSS_WDOG0_LO to MSS_WDOG4_LO correspond to the
watchdog modules 0 to 4 when they appear on the AXI switch slave 5. MSS_WDOG0_HI
to MSS_WDOG4_HI correspond to the watchdog modules 0 to 4 when they appear on
the AXI switch slave 6.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_WD_clear_mvrp_irq$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="msswdtimeoutoccured"></a>
## MSS_WD_timeout_occured
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_WD_timeout_occured$prototype" data-type="code">

    uint32_t
    MSS_WD_timeout_occured
    (
        mss_watchdog_num_t wd_num
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_WD_timeout_occured$description" data-type="text">

The MSS_WD_timeout_occured() function reports the occurrence of a timeout event.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### wd_num
<div id="Functions$MSS_WD_timeout_occured$description$parameters$wd_num" data-type="text" data-name="wd_num">

The wd_num parameter is the watchdog module number in the PolarFire SoC MSS on
which the operation needs to be performed. Choose the watchdog module number
using mss_watchdog_num_t. MSS_WDOG0_LO to MSS_WDOG4_LO correspond to the
watchdog modules 0 to 4 when they appear on the AXI switch slave 5. MSS_WDOG0_HI
to MSS_WDOG4_HI correspond to the watchdog modules 0 to 4 when they appear on
the AXI switch slave 6.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_WD_timeout_occured$description$return" data-type="text">

A zero value indicates no watchdog timeout event occurred. Value 1 indicates
that a timeout event occurred.


</div>

##### Example
<div id="Functions$MSS_WD_timeout_occured$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$MSS_WD_timeout_occured$description$example$Example1" data-type="code" data-name="Example">


```
    #include "mss_watchdog.h"
    void e51( void )
    {
        uint32_t wdg_reset;
        mss_watchdog_num_t wd_num;

        wdg_reset = MSS_WD_timeout_occured(wd_num);
        if (wdg_reset)
        {
            log_watchdog_event();
            MSS_WD_clear_timeout_event();
        }

        for(;;)
        {
            main_task();
        }
    }
```

</div>


 -------------------------------- 
<a name="msswdforcereset"></a>
## MSS_WD_force_reset
<a name="prototype"></a>
### Prototype 

<div id="Functions$MSS_WD_force_reset$prototype" data-type="code">

    void
    MSS_WD_force_reset
    (
        mss_watchdog_num_t wd_num
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$MSS_WD_force_reset$description" data-type="text">

The MSS_WD_force_reset() function is used to force an immediate reset if the
watchdog has timed out. Writing any value in this condition causes an NMI
sequence. Moreover, any attempt to force reset when the watchdog is not in timed
out condition also causes an NMI sequence.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### wd_num
<div id="Functions$MSS_WD_force_reset$description$parameters$wd_num" data-type="text" data-name="wd_num">

The wd_num parameter is the watchdog module number in the PolarFire SoC MSS on
which the operation needs to be performed. Choose the watchdog module number
using mss_watchdog_num_t. MSS_WDOG0_LO to MSS_WDOG4_LO correspond to the
watchdog modules 0 to 4 when they appear on the AXI switch slave 5. MSS_WDOG0_HI
to MSS_WDOG4_HI correspond to the watchdog modules 0 to 4 when they appear on
the AXI switch slave 6.


</div>

<a name="return"></a>
### Return
<div id="Functions$MSS_WD_force_reset$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
