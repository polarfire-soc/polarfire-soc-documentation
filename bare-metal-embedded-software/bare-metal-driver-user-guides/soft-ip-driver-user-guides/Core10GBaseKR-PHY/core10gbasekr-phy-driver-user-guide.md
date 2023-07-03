<html>
 
 ------------------------------------ 

# Core10GBaseKR_PHY Bare Metal Driver
 
-----------------------------------------
## Table of Contents 
  - [Introduction](#introduction)

  - [Hardware Flow Dependencies](#hardware-flow-dependencies)

  - [Software Flow Dependencies](#software-flow-dependencies)

  - [Theory of Operation](#theory-of-operation)

     - [Initialization](#initialization)

     - [Configuration](#configuration)

     - [Clause73: Auto-negotiation](#clause73:-auto-negotiation)

     - [Clause72: Link Training](#clause72:-link-training)

     - [10GBASE-KR](#10gbase-kr)

- [Types](#types)
  - [phy10gkr_lane_los_t](#phy10gkrlanelost)
  - [phy10gkr_timer_t](#phy10gkrtimert)
  - [phy10gkr_an_state_t](#phy10gkranstatet)
  - [phy10gkr_an_status_t](#phy10gkranstatust)
  - [phy10gkr_an_api_state_t](#phy10gkranapistatet)
  - [phy10gkr_an_instance_t](#phy10gkraninstancet)
  - [phy10gkr_lt_state_t](#phy10gkrltstatet)
  - [phy10gkr_lt_link_status_t](#phy10gkrltlinkstatust)
  - [phy10gkr_lt_api_state_t](#phy10gkrltapistatet)
  - [phy10gkr_coeff_update_status_t](#phy10gkrcoeffupdatestatust)
  - [phy10gkr_calirbation_request_t](#phy10gkrcalirbationrequestt)
  - [phy10gkr_coeff_status_report_t](#phy10gkrcoeffstatusreportt)
  - [phy10gkr_tx_equalizer_tap_t](#phy10gkrtxequalizertapt)
  - [phy10gkr_tap_cal_state_t](#phy10gkrtapcalstatet)
  - [phy10gkr_local_rxcvr_lock_t](#phy10gkrlocalrxcvrlockt)
  - [phy10gkr_coeff_update_t](#phy10gkrcoeffupdatet)
  - [phy10gkr_lt_instance_t](#phy10gkrltinstancet)
  - [phy10gkr_state_t](#phy10gkrstatet)
  - [phy10gkr_status_t](#phy10gkrstatust)
  - [phy10gkr_instance_t](#phy10gkrinstancet)

- [Constants](#constants)
  - [CORE10GBASEKR_PHY LT MAX/MIN LIMITS](#core10gbasekrphy-lt-maxmin-limits)
  - [CORE10GBASEKR_PHY PRESET](#core10gbasekrphy-preset)
  - [CORE10GBASEKR_PHY INIT](#core10gbasekrphy-init)
  - [CORE10GBASEKR_PHY LP REQUEST](#core10gbasekrphy-lp-request)
  - [CORE10GBASEKR_PHY AN LINK FAIL INHIBIT TIMER](#core10gbasekrphy-an-link-fail-inhibit-timer)

- [Functions](#functions)
  - [PHY10GKR_init](#phy10gkrinit)
  - [PHY10GKR_config](#phy10gkrconfig)
  - [PHY10GKR_autonegotiate_sm](#phy10gkrautonegotiatesm)
  - [PHY10GKR_link_training_sm](#phy10gkrlinktrainingsm)
  - [PHY10GKR_10gbasekr_sm](#phy10gkr10gbasekrsm)
  - [PHY10GKR_set_lane_los_signal](#phy10gkrsetlanelossignal)
  - [PHY10GKR_get_current_time_ms](#phy10gkrgetcurrenttimems)
  - [PHY10GKR_serdes_an_config](#phy10gkrserdesanconfig)
  - [PHY10GKR_serdes_lt_config](#phy10gkrserdesltconfig)
  - [PHY10GKR_serdes_cdr_lock](#phy10gkrserdescdrlock)
  - [PHY10GKR_serdes_dfe_cal](#phy10gkrserdesdfecal)
  - [PHY10GKR_serdes_tx_equalization](#phy10gkrserdestxequalization)

<div id="TitlePage" data-type="text">

# Introduction
Core10GBaseKR_PHY is designed for the IEEE® 802.3-2012 specification and
supports the Core10GBaseKR_PHY interface for Backplane operations. This
configurable core provides the Physical (PHY) layer when used with a transceiver
interface. This IP interfaces with the Ten Gigabit Media Independent Interface
(XGMII) compliant Media Access Control (MAC) at the system side and the
transceiver block at the line side. The physical layer is designed to work
seamlessly with the PolarFire® and PolarFire SoC transceiver using the Physical
Medium Attachment (PMA) mode. This user guide documents the features provided by
the Core10GBaseKR_PHY firmware driver.

# Hardware Flow Dependencies
This driver covers the configuration details of features such as IEEE802.3
clauses 73 and 72.

See the Core10GBaseKR_PHY User Guide for a detailed description of design
requirements when interfacing the Core10GBaseKR_PHY to a transceiver.

# Software Flow Dependencies
A PHY software configuration file must be included when using this driver. This
will include the macro definition "CORE10GBASEKR_PHY". The purpose of this file
is to configure desired PHY model and to override driver default values as
required. The driver configuration should be stored in a location away from the
driver source code. The following with path is used in our recommended directory
structure:

` <project-root>/boards/<board-name>/platform_config/driver_config/phy_sw_cfg.h
`

# Theory of Operation
The Core10GBaseKR_PHY driver functions are grouped into the following
categories:

  - Initialization
  - Configuration
  - Clause73: Auto-negotiation
  - Clause72: Link training
  - 10GBASE-KR

## Initialization
The Core10GBaseKR_PHY driver is initialized through a call to the
PHY10GKR_init() function. The PHY10GKR_init() function must be called before
calling any other Core10GBaseKR_PHY driver functions.

## Configuration
An instance of the Core10GBaseKR_PHY is configured with a call to the
PHY10GKR_config(). The configuration function resets all the PHY instance
structure members other than information such as performance counters. Default
configurations can be overridden be defining any of the Core10GBaseKR_PHY
constants before loading the driver using a PHY software configuration file.

## Clause73: Auto-negotiation
The IEEE802.3 clause 73 auto-negotiation is enabled and executed by calling
PHY10GKR_autonegotiate_sm() function.

## Clause72: Link Training
The IEEE802.3 clause 72 link training is enabled and executed by a calling
PHY10GKR_link_training_sm(). The Core10GBaseKR_PHY IP and the Core10GBaseKR_PHY
embedded software driver together carry out the link training. The driver
initiates the link training and takes appropriate actions depending on the
events indicated by the 10GBaseKR status register bits during the link training
process. The following figure shows an overview of what actions are taken for
each 10GBaseKR status bit:


<div class="mermaid">
%%{init : {"flowchart" : {"curve" : "linear", "useMaxWidth" : false, "useMaxHeight" : false, "nodeSpacing" : 30, "rankSpacing" : 40 }}}%%
    flowchart TB
        Start[/10GBASE-KR Status Register\] --> Rx[Rx Calibration]
        Start --> Tx[Tx Equalization]
        Start --> Fail[Fail]
        Start --> Signal[Signal Detect]
        
        Fail --> LTFail[LT Failure]
        
        Rx --> IntRx[Set RX Calibration Bit to Clear Status Bit]
        IntRx --> Algo[Max/Min Tap Sweep Algorithm]
        Algo --> TxCU[Update the Tx Coefficient Update]
        
        Tx --> SerdesTX[Pass Rx Coefficient Update to Serdes]
        SerdesTX --> TxSR[Update Tx Status Report]
        TxSR --> TxDone[Set Tx Equalization Done to Clear Status Bit]
                
        Signal --> LTComp[LT Complete]

</div>

<style>
    .mermaid svg { 
        max-width: 100%; 
        height: auto;
    }
</style>
Training Failure: The training failure bit is set by the IP when the
Core10GBaseKR_PHY link training timer exceeds 500 ms. The driver also implements
a soft timer as an additional protection layer. The
PHY10GKR_get_current_time_ms() function must be overridden by instantiating this
function in user code so that the current time of a timer will be returned in
milli-seconds. When this status is set by Core10GBaseKR_PHY, the embedded
software must reduce the XCVR data rate by calling PHY10GKR_serdes_an_config()
and restart the auto-negotiation state machine by calling
PHY10GKR_autonegotiate_sm().

Rx Calibration: The IP sets this status bit to indicate that there is a received
status report of Max/Min/Updated that the Rx calibration algorithm must handle.
The maximum to minimum sweep algorithm described in the Core10GBaseKR_PHY User
Guide is implemented by the functions which are defined within
core10gbasekr_phy_link_training.h.

After the Rx calibration algorithm completes, the driver updates the transmit
coefficient with new transmitter tap, which will be sent to the link partner.

Tx Equalization: This status bit indicates that the received coefficient update
has been updated and that the firmware needs to update the transceiver
transmitter taps. The driver hands off the new coefficient settings to the
transceiver using the PHY10GKR_serdes_tx_equalization() weak function, which
must be overridden.

Signal Detect: When the driver identifies that this bit has been set by the
Core10GBaseKR_PHY IP, it sets the link training complete flag. The IP updates
this status bit when both the transmitted and received status reports have the
receiver ready bit set. This indicates that both devices have completed their Rx
calibration algorithm.

## 10GBASE-KR
The full 10GBASE-KR flow is handled by calling PHY10GKR_10gbasekr_sm() function.
As this function is dependent on interacting with a transceiver, the following
weak functions should be overridden for the specific design being implemented.

  - PHY10GKR_get_current_time_ms()
  - PHY10GKR_serdes_an_config()
  - PHY10GKR_serdes_lt_config()
  - PHY10GKR_serdes_cdr_lock()
  - PHY10GKR_serdes_dfe_cal()
  - PHY10GKR_serdes_tx_equalization() 

</div>


# Types

 ---------------- 
<a name="phy10gkrlanelost"></a>
## phy10gkr_lane_los_t
<a name="prototype"></a>
### Prototype 

<div id="Types$phy10gkr_lane_los_t$prototype" data-type="code">

 ``` 
    typedef enum {
        LANE_LOS_LOCK_TO_DATA = 0,
        LANE_LOS_LOCK_TO_REF = 1
    } phy10gkr_lane_los_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_lane_los_t$description" data-type="text">

The phy10gkr_lane_los_t enumeration identifies the state of the lane loss
detection.


</div>


 --------------------------- 
<a name="phy10gkrtimert"></a>
## phy10gkr_timer_t
<a name="prototype"></a>
### Prototype 

<div id="Types$phy10gkr_timer_t$prototype" data-type="code">

``` 
    typedef struct __phy_timer {
        uint32_t start; 
        uint32_t end; 
    } phy10gkr_timer_t ;
  ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_timer_t$description" data-type="text">

The phy10gkr_timer_t structure describes the start and end time instance of the
measurement with the PHY driver.


</div>


 --------------------------- 
<a name="phy10gkranstatet"></a>
## phy10gkr_an_state_t
<a name="prototype"></a>
### Prototype 

<div id="Types$phy10gkr_an_state_t$prototype" data-type="code">

 ``` 
    typedef enum {
        ST_AUTO_NEG_ENABLE = 0x0,
        ST_TRANSMIT_DISABLE = 0x1,
        ST_ABILITY_DETECT = 0x2,
        ST_ACKNOWLEDGE_DETECT = 0x3,
        ST_COMPLETE_ACKNOWLEDEGE = 0x4,
        ST_AN_GOOD_CHECK = 0x5,
        ST_AN_GOOD = 0x6,
        ST_NEXT_PAGE_WAIT = 0x7,
        ST_NEXT_PAGE_WAIT_TX_IDLE = 0x8,
        ST_LINK_STATUS_CHECK = 0x9,
        ST_PARALLEL_DETECTION_FAULT = 0xA
    } phy10gkr_an_state_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_an_state_t$description" data-type="text">

The phy10gkr_an_state_t enumeration is used to identify the state of the auto-
negotiation arbitration state machine, which is implemented by the
Core10GBaseKR_PHY.


</div>


 --------------------------- 
<a name="phy10gkranstatust"></a>
## phy10gkr_an_status_t
<a name="prototype"></a>
### Prototype 

<div id="Types$phy10gkr_an_status_t$prototype" data-type="code">

 ``` 
    typedef enum {
        STATUS_AN_INCOMPLETE,
        STATUS_AN_COMPLETE
    } phy10gkr_an_status_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_an_status_t$description" data-type="text">

The phy10gkr_an_status_t enumeration specifies the status of auto-negotiation.


</div>


 --------------------------- 
<a name="phy10gkranapistatet"></a>
## phy10gkr_an_api_state_t
<a name="prototype"></a>
### Prototype 

<div id="Types$phy10gkr_an_api_state_t$prototype" data-type="code">

 ``` 
    typedef enum {
        AN_API_SM_INIT,
        AN_API_SM_STATUS_UPDATE
    } phy10gkr_an_api_state_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_an_api_state_t$description" data-type="text">

The phy10gkr_an_api_state_t enumeration identifies the state of the auto-
negotiation state machine API.


</div>


 --------------------------- 
<a name="phy10gkraninstancet"></a>
## phy10gkr_an_instance_t
<a name="prototype"></a>
### Prototype 

<div id="Types$phy10gkr_an_instance_t$prototype" data-type="code">

``` 
    typedef struct __an_instance {
        phy10gkr_an_state_t state; 
        phy10gkr_an_api_state_t api_state; 
        uint32_t complete_cnt; 
        phy10gkr_an_status_t status; 
    } phy10gkr_an_instance_t ;
  ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_an_instance_t$description" data-type="text">

The phy10gkr_an_instance_t struct describes an instance of the auto-negotiation
parameters.


</div>


 --------------------------- 
<a name="phy10gkrltstatet"></a>
## phy10gkr_lt_state_t
<a name="prototype"></a>
### Prototype 

<div id="Types$phy10gkr_lt_state_t$prototype" data-type="code">

 ``` 
    typedef enum {
        LT_STATE_IDLE = 0,
        LT_STATE_INITIALIZE = 1,
        LT_STATE_SEND_TRAINING = 3,
        LT_STATE_TRAIN_LOCAL = 2,
        LT_STATE_TRAIN_REMOTE = 6,
        LT_STATE_LINK_READY = 7,
        LT_STATE_SEND_DATA = 5,
        LT_STATE_FAILURE = 4
    } phy10gkr_lt_state_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_lt_state_t$description" data-type="text">

The phy10gkr_lt_state_t enumeration identifies the link training state machine
state implemented by the Core10GBaseKr_PHY IP block.


</div>


 --------------------------- 
<a name="phy10gkrltlinkstatust"></a>
## phy10gkr_lt_link_status_t
<a name="prototype"></a>
### Prototype 

<div id="Types$phy10gkr_lt_link_status_t$prototype" data-type="code">

 ``` 
    typedef enum {
        STATUS_LT_INCOMPLETE,
        STATUS_LT_COMPLETE,
        STATUS_LT_LINK_MAINTAINED,
        STATUS_LT_FAILURE
    } phy10gkr_lt_link_status_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_lt_link_status_t$description" data-type="text">

The phy10gkr_lt_link_status_t enumeration identifies the status and state of the
link with the link partner.


</div>


 --------------------------- 
<a name="phy10gkrltapistatet"></a>
## phy10gkr_lt_api_state_t
<a name="prototype"></a>
### Prototype 

<div id="Types$phy10gkr_lt_api_state_t$prototype" data-type="code">

 ``` 
    typedef enum {
        LT_API_SM_INIT,
        LT_API_SM_STATUS_UPDATE
    } phy10gkr_lt_api_state_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_lt_api_state_t$description" data-type="text">

The phy10gkr_lt_api_state_t enumeration identifies the link training API state
machine state.


</div>


 --------------------------- 
<a name="phy10gkrcoeffupdatestatust"></a>
## phy10gkr_coeff_update_status_t
<a name="prototype"></a>
### Prototype 

<div id="Types$phy10gkr_coeff_update_status_t$prototype" data-type="code">

 ``` 
    typedef enum {
        SWEEP_NOT_STARTED,
        SWEEP_START,
        SWEEP_INCOMPLETE,
        SWEEP_COMPLETE
    } phy10gkr_coeff_update_status_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_coeff_update_status_t$description" data-type="text">

The phy10gkr_coeff_update_status_t enumeration identifies the status of the
coefficient sweep algorithm.


</div>


 --------------------------- 
<a name="phy10gkrcalirbationrequestt"></a>
## phy10gkr_calirbation_request_t
<a name="prototype"></a>
### Prototype 

<div id="Types$phy10gkr_calirbation_request_t$prototype" data-type="code">

 ``` 
    typedef enum {
        C10GBKR_LT_PRESET = 0U,
        C10GBKR_LT_INITALISE = 1U
    } phy10gkr_calirbation_request_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_calirbation_request_t$description" data-type="text">

This enumeration identifies the initial conditions of a device. For example, the
local device will calibrate using a preset request.


</div>


 --------------------------- 
<a name="phy10gkrcoeffstatusreportt"></a>
## phy10gkr_coeff_status_report_t
<a name="prototype"></a>
### Prototype 

<div id="Types$phy10gkr_coeff_status_report_t$prototype" data-type="code">

 ``` 
    typedef enum {
        LT_COEFF_STATUS_NOT_UPDATED = 0U,
        LT_COEFF_STATUS_UPDATED = 1U,
        LT_COEFF_STATUS_MIN = 2U,
        LT_COEFF_STATUS_MAX = 3U
    } phy10gkr_coeff_status_report_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_coeff_status_report_t$description" data-type="text">

The phy10gkr_coeff_status_report_t enumeration identifies the link training
status report update.


</div>


 --------------------------- 
<a name="phy10gkrtxequalizertapt"></a>
## phy10gkr_tx_equalizer_tap_t
<a name="prototype"></a>
### Prototype 

<div id="Types$phy10gkr_tx_equalizer_tap_t$prototype" data-type="code">

 ``` 
    typedef enum {
        PRE_TAP,
        MAIN_TAP,
        POST_TAP
    } phy10gkr_tx_equalizer_tap_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_tx_equalizer_tap_t$description" data-type="text">

This enumeration specifies the three different transmitter taps.


</div>


 --------------------------- 
<a name="phy10gkrtapcalstatet"></a>
## phy10gkr_tap_cal_state_t
<a name="prototype"></a>
### Prototype 

<div id="Types$phy10gkr_tap_cal_state_t$prototype" data-type="code">

 ``` 
    typedef enum {
        TAP_MAX_CAL,
        TAP_MIN_CAL,
        TAP_OPTIMISE_CAL
    } phy10gkr_tap_cal_state_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_tap_cal_state_t$description" data-type="text">

This enumeration specifies the state of the link partner calibration algorithm.


</div>


 --------------------------- 
<a name="phy10gkrlocalrxcvrlockt"></a>
## phy10gkr_local_rxcvr_lock_t
<a name="prototype"></a>
### Prototype 

<div id="Types$phy10gkr_local_rxcvr_lock_t$prototype" data-type="code">

 ``` 
    typedef enum {
        LOCAL_RXCVR_UNLOCKED = 0,
        LOCAL_RXCVR_LOCKED = 1
    } phy10gkr_local_rxcvr_lock_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_local_rxcvr_lock_t$description" data-type="text">

This enumeration specifies the condition of the link partner calibration
algorithm. During training, when the link partner has been calibrated, the local
receiver ready lock get locked and the status report will be updated to notify
the link partner.


</div>


 --------------------------- 
<a name="phy10gkrcoeffupdatet"></a>
## phy10gkr_coeff_update_t
<a name="prototype"></a>
### Prototype 

<div id="Types$phy10gkr_coeff_update_t$prototype" data-type="code">

``` 
    typedef struct coeff_update {
        uint32_t cnt; 
        uint32_t inc_cnt; 
        uint32_t hold_cnt; 
        uint32_t dec_cnt; 
        uint32_t max_cnt; 
        uint32_t min_cnt; 
        uint32_t update_cnt; 
        uint32_t no_update_cnt; 
        uint32_t done_cnt; 
        uint32_t index; 
        phy10gkr_coeff_update_status_t status; 
        phy10gkr_tap_cal_state_t lp_tap_cal_state; 
        uint32_t optimal_index; 
        uint32_t optimal_cnt; 
    } phy10gkr_coeff_update_t ;
  ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_coeff_update_t$description" data-type="text">

The coeff_update_t struct describes an instance of the link training coefficient
update. This structure supports calibrating the link partners transmitter taps.


</div>


 --------------------------- 
<a name="phy10gkrltinstancet"></a>
## phy10gkr_lt_instance_t
<a name="prototype"></a>
### Prototype 

<div id="Types$phy10gkr_lt_instance_t$prototype" data-type="code">

``` 
    typedef struct __lt_instance {
        phy10gkr_lt_state_t state; 
        phy10gkr_lt_api_state_t api_state; 
        phy10gkr_lt_link_status_t status; 
        phy10gkr_timer_t timer; 
        phy10gkr_calirbation_request_t tx_request; 
        phy10gkr_calirbation_request_t rx_request; 
        uint32_t fail_cnt; 
        uint32_t complete_cnt; 
        uint32_t tx_equ_cnt; 
        uint32_t rx_cal_cnt; 
        uint32_t sig_cnt; 
        uint32_t rcvr_cnt; 
        uint32_t sm_cycle_cnt; 
        phy10gkr_tx_equalizer_tap_t lp_cal_sweep_state; 
        phy10gkr_coeff_update_t main; 
        phy10gkr_coeff_update_t post; 
        phy10gkr_coeff_update_t pre; 
        phy10gkr_local_rxcvr_lock_t local_rxcvr; 
    } phy10gkr_lt_instance_t ;
  ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_lt_instance_t$description" data-type="text">

The an_instance_t struct describes an instance of the link training parameters.


</div>


 --------------------------- 
<a name="phy10gkrstatet"></a>
## phy10gkr_state_t
<a name="prototype"></a>
### Prototype 

<div id="Types$phy10gkr_state_t$prototype" data-type="code">

 ``` 
    typedef enum {
        AN_SERDES_CONFIG,
        AN_SM,
        LT_SERDES_CONFIG,
        LT_SM,
        LINK_ESTABLISHED_CHECK
    } phy10gkr_state_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_state_t$description" data-type="text">

The c10gbkr_state_t enumeration identifies the state of the 10GBASE-KR state
machine.


</div>


 --------------------------- 
<a name="phy10gkrstatust"></a>
## phy10gkr_status_t
<a name="prototype"></a>
### Prototype 

<div id="Types$phy10gkr_status_t$prototype" data-type="code">

 ``` 
    typedef enum {
        AN_SERDES_CONFIGURATION = 1,
        AN_IN_PROGRESS = 2,
        AN_COMPLETE = 3,
        LT_SERDES_CONFIGURATION = 4,
        LT_SERDES_CAL_FAILURE = 5,
        LT_SERDES_CAL_COMPLETE = 6,
        LT_IN_PROGRESS = 7,
        LT_FAILURE = 8,
        LINK_BROKEN = 9,
        LINK_ESTABLISHED = 0
    } phy10gkr_status_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_status_t$description" data-type="text">

The c10gbkr_status_t enumeration identifies the status of the 10GBASE-KR state
machine.

This enumeration can identify failures encountered by the 10GBASE-KR algorithm.


</div>


 --------------------------- 
<a name="phy10gkrinstancet"></a>
## phy10gkr_instance_t
<a name="prototype"></a>
### Prototype 

<div id="Types$phy10gkr_instance_t$prototype" data-type="code">

``` 
    typedef struct __phy10gkr_instance {
        addr_t base_addr; 
        addr_t an_base_addr; 
        addr_t lt_base_addr; 
        addr_t tx_ctrl_base_addr; 
        addr_t rx_status_base_addr; 
        phy10gkr_an_instance_t an; 
        phy10gkr_lt_instance_t lt; 
        phy10gkr_state_t c10gbkr_state; 
        phy10gkr_status_t c10gbkr_status; 
        uint32_t serdes_id; 
        uint32_t serdes_lane_id; 
    } phy10gkr_instance_t ;
  ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_instance_t$description" data-type="text">

The phy10gkr_instance_t struct describes an instance of the Core10GBaseKR_PHY
parameters.


</div>


 --------------------------- 

# Constants

 ---------------- 
<div id="Constants$C10GBKR_LT_MAIN_TAP_MAX_LIMIT$description" data-type="text">

<a name="c10gbkrltmaintapmaxlimit"></a>
## CORE10GBASEKR_PHY LT MAX/MIN LIMITS
The max/min limit constants define the XCVR tap coefficient limits. These
constants can be overridden based on the XCVR, which is integrated into a
specific design.

Note: Post and Pre tap maximum limits are absolute.

</div>

<div id="Constants$C10GBKR_LT_PRESET_MAIN_TAP$description" data-type="text">

<a name="c10gbkrltpresetmaintap"></a>
## CORE10GBASEKR_PHY PRESET
The preset constants define the XCVR tap coefficient settings for a preset
request. They can be overridden based on the XCVR, which is integrated into a
specific design.

Note: Post and Pre tap maximum limits are absolute.

</div>

<div id="Constants$C10GBKR_LT_INITIALIZE_MAIN_TAP$description" data-type="text">

<a name="c10gbkrltinitializemaintap"></a>
## CORE10GBASEKR_PHY INIT
The initialize constants define the coefficient settings, which is set when an
initialize request is received from the link partner. These constants should be
updated if there is no desire to calibrate the links.

</div>

<div id="Constants$C10GBKR_LT_INITIAL_REQUEST$description" data-type="text">

<a name="c10gbkrltinitialrequest"></a>
## CORE10GBASEKR_PHY LP REQUEST
This constant defines the request, which will be sent to the link partner and
determines which algorithm will be implemented to calibrate the link partner.

</div>

<div id="Constants$C10GBKR_AN_LINK_FAIL_INHITBIT_TIMER$description" data-type="text">

<a name="c10gbkranlinkfailinhitbittimer"></a>
## CORE10GBASEKR_PHY AN LINK FAIL INHIBIT TIMER
This constant defines the auto-negotiation link fail inhibit timer timeout in
milli-seconds.

</div>


# Functions

 ---------------- 
<a name="phy10gkrinit"></a>
## PHY10GKR_init
<a name="prototype"></a>
### Prototype 

<div id="Functions$PHY10GKR_init$prototype" data-type="code">

    void
    PHY10GKR_init
    (
        phy10gkr_instance_t * this_phy,
        addr_t base_addr
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PHY10GKR_init$description" data-type="text">

The PHY10GKR_init() function initializes the Core10GBaseKR_PHY bare-metal
driver. This function sets the base address of the Auto-negotiation, link-
training, tx control, and rx status registers.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_phy
<div id="Functions$PHY10GKR_init$description$parameters$this_phy" data-type="text" data-name="this_phy">

The this_phy parameter specifies the PHY instance.


</div>

#### base_addr
<div id="Functions$PHY10GKR_init$description$parameters$base_addr" data-type="text" data-name="base_addr">

The base_addr specifies the base address of the IP block.


</div>

<a name="return"></a>
### Return
<div id="Functions$PHY10GKR_init$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$PHY10GKR_init$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$PHY10GKR_init$description$example$Example1" data-type="code" data-name="Example">


```
    #include "phy.h"
    int main(void)
    {
      PHY10GKR_init(&g_phy, CORE10GBKR_0_PHY_BASE_ADDR);
      return (0u);
    }
```

</div>


 -------------------------------- 
<a name="phy10gkrconfig"></a>
## PHY10GKR_config
<a name="prototype"></a>
### Prototype 

<div id="Functions$PHY10GKR_config$prototype" data-type="code">

    void
    PHY10GKR_config
    (
        phy10gkr_instance_t * this_phy
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PHY10GKR_config$description" data-type="text">

The PHY10GKR_config() function configures the PHY registers with the predefined
defaults and user configurations. This function also resets the structures back
to their initial conditions.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_phy
<div id="Functions$PHY10GKR_config$description$parameters$this_phy" data-type="text" data-name="this_phy">

The this_phy parameter specifies the PHY instance.


</div>

<a name="return"></a>
### Return
<div id="Functions$PHY10GKR_config$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$PHY10GKR_config$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$PHY10GKR_config$description$example$Example1" data-type="code" data-name="Example">


```
    #include "phy.h"
    int main(void)
    {
      PHY10GKR_init(&g_phy, CORE10GBKR_0_PHY_BASE_ADDR);
      PHY10GKR_config(&g_phy);
      return (0u);
    }
```

</div>


 -------------------------------- 
<a name="phy10gkrautonegotiatesm"></a>
## PHY10GKR_autonegotiate_sm
<a name="prototype"></a>
### Prototype 

<div id="Functions$PHY10GKR_autonegotiate_sm$prototype" data-type="code">

    void
    PHY10GKR_autonegotiate_sm
    (
        phy10gkr_instance_t * this_phy
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PHY10GKR_autonegotiate_sm$description" data-type="text">

The PHY10GKR_autonegotiate_sm() function enables the auto-negotiation API state
machine, which enables the auto-negotiation registers and then checks the status
of the auto-negotiation state machine to determine if auto-negotiation has
complete.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_phy
<div id="Functions$PHY10GKR_autonegotiate_sm$description$parameters$this_phy" data-type="text" data-name="this_phy">

The this_phy parameter specifies the PHY instance.


</div>

<a name="return"></a>
### Return
<div id="Functions$PHY10GKR_autonegotiate_sm$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$PHY10GKR_autonegotiate_sm$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$PHY10GKR_autonegotiate_sm$description$example$Example1" data-type="code" data-name="Example">


```
    #include "phy.h"
    int main(void)
    {
      PHY10GKR_init(&g_phy, CORE10GBKR_0_PHY_BASE_ADDR);
      while(1)
      {
          PHY10GKR_autonegotiate_sm(&g_phy);
          if(STATUS_AN_COMPLETE == g_phy.an.complete)
          {
              break;
          }
      }
      return (0u);
    }
```

</div>


 -------------------------------- 
<a name="phy10gkrlinktrainingsm"></a>
## PHY10GKR_link_training_sm
<a name="prototype"></a>
### Prototype 

<div id="Functions$PHY10GKR_link_training_sm$prototype" data-type="code">

    void
    PHY10GKR_link_training_sm
    (
        phy10gkr_instance_t * this_phy
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PHY10GKR_link_training_sm$description" data-type="text">

The PHY10GKR_link_training_sm() function enables the link training API state
machine, which enables the link training registers and then runs the link
training algorithm.

The connected transceiver must have a data rate of 10 Gbps and locked to a link
partner with the same data rate for successful link training.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_phy
<div id="Functions$PHY10GKR_link_training_sm$description$parameters$this_phy" data-type="text" data-name="this_phy">

The this_phy parameter specifies the PHY instance.


</div>

<a name="return"></a>
### Return
<div id="Functions$PHY10GKR_link_training_sm$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$PHY10GKR_link_training_sm$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$PHY10GKR_link_training_sm$description$example$Example1" data-type="code" data-name="Example">


```
    #include "phy.h"
    int main(void)
    {
      PHY10GKR_init(&g_phy, CORE10GBKR_0_PHY_BASE_ADDR);
      while(1)
      {
          PHY10GKR_link_training_sm(&g_phy);
          if(STATUS_LT_FAILURE == g_phy.lt.status)
          {
              HAL_ASSERT(0);
          }
      }
      return (0u);
    }
```

</div>


 -------------------------------- 
<a name="phy10gkr10gbasekrsm"></a>
## PHY10GKR_10gbasekr_sm
<a name="prototype"></a>
### Prototype 

<div id="Functions$PHY10GKR_10gbasekr_sm$prototype" data-type="code">

    uint32_t
    PHY10GKR_10gbasekr_sm
    (
        phy10gkr_instance_t * this_phy
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PHY10GKR_10gbasekr_sm$description" data-type="text">

The PHY10GKR_10gbasekr_sm() executes the full 10GBASE-KR flow required to
complete the auto-negotiation and link training.

The 10GBASE-KR status enumeration allows the user to debug the auto-negotiation
and link training algorithms.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_phy
<div id="Functions$PHY10GKR_10gbasekr_sm$description$parameters$this_phy" data-type="text" data-name="this_phy">

The this_phy parameter specifies the PHY instance.


</div>

<a name="return"></a>
### Return
<div id="Functions$PHY10GKR_10gbasekr_sm$description$return" data-type="text">

This function returns a state machine status.


</div>

##### Example
<div id="Functions$PHY10GKR_10gbasekr_sm$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$PHY10GKR_10gbasekr_sm$description$example$Example1" data-type="code" data-name="Example">


```
    #include "phy.h"
    int main(void)
    {
      uint32_t status;
      PHY10GKR_init(&g_phy, CORE10GBKR_0_PHY_BASE_ADDR);
      while(1)
      {
          status = PHY10GKR_10gbasekr_sm(&g_phy);
          if(LINK_ESTABLISHED == status)
          {
              break;
          }
      }
      return (0u);
    }
```

</div>


 -------------------------------- 
<a name="phy10gkrsetlanelossignal"></a>
## PHY10GKR_set_lane_los_signal
<a name="prototype"></a>
### Prototype 

<div id="Functions$PHY10GKR_set_lane_los_signal$prototype" data-type="code">

    void
    PHY10GKR_set_lane_los_signal
    (
        phy10gkr_instance_t * this_phy,
        uint32_t state
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PHY10GKR_set_lane_los_signal$description" data-type="text">

The PHY10GKR_set_lane_los_signal() asserts and deasserts the transceivers Lane
Loss of signal detection.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### state
<div id="Functions$PHY10GKR_set_lane_los_signal$description$parameters$state" data-type="text" data-name="state">

Asserts or deasserts the lane LOS


</div>

<a name="return"></a>
### Return
<div id="Functions$PHY10GKR_set_lane_los_signal$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="phy10gkrgetcurrenttimems"></a>
## PHY10GKR_get_current_time_ms
<a name="prototype"></a>
### Prototype 

<div id="Functions$PHY10GKR_get_current_time_ms$prototype" data-type="code">

    uint32_t
    PHY10GKR_get_current_time_ms
    (
        void 
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PHY10GKR_get_current_time_ms$description" data-type="text">

The PHY10GKR_get_current_time_ms() is a weak function that can be overridden by
the user to get the current time in milli-seconds.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
<div id="" data-type="text" data-name="None">

This function takes no parameters.


</div>

<a name="return"></a>
### Return
<div id="Functions$PHY10GKR_get_current_time_ms$description$return" data-type="text">

This function returns the time in milli-seconds.


</div>


 -------------------------------- 
<a name="phy10gkrserdesanconfig"></a>
## PHY10GKR_serdes_an_config
<a name="prototype"></a>
### Prototype 

<div id="Functions$PHY10GKR_serdes_an_config$prototype" data-type="code">

    void
    PHY10GKR_serdes_an_config
    (
        void 
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PHY10GKR_serdes_an_config$description" data-type="text">

The PHY10GKR_serdes_an_config() is a weak function that can be overridden by the
user to configure the XCVR instance integrated in their design for auto-
negotiation.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
<div id="" data-type="text" data-name="None">

This function takes no parameters.


</div>

<a name="return"></a>
### Return
<div id="Functions$PHY10GKR_serdes_an_config$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="phy10gkrserdesltconfig"></a>
## PHY10GKR_serdes_lt_config
<a name="prototype"></a>
### Prototype 

<div id="Functions$PHY10GKR_serdes_lt_config$prototype" data-type="code">

    uint32_t
    PHY10GKR_serdes_lt_config
    (
        void 
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PHY10GKR_serdes_lt_config$description" data-type="text">

The PHY10GKR_serdes_lt_config() is a weak function that can be overridden by the
user to configure the XCVR instance integrated in their design for link training
at 10 Gbps.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
<div id="" data-type="text" data-name="None">

This function takes no parameters.


</div>

<a name="return"></a>
### Return
<div id="Functions$PHY10GKR_serdes_lt_config$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="phy10gkrserdescdrlock"></a>
## PHY10GKR_serdes_cdr_lock
<a name="prototype"></a>
### Prototype 

<div id="Functions$PHY10GKR_serdes_cdr_lock$prototype" data-type="code">

    uint32_t
    PHY10GKR_serdes_cdr_lock
    (
        void 
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PHY10GKR_serdes_cdr_lock$description" data-type="text">

The PHY10GKR_serdes_cdr_lock() is a weak function that can be overridden by the
user to determine that the XCVR instance integrated has achieved a CDR lock.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
<div id="" data-type="text" data-name="None">

This function t
</html>
arameters.


</div>

<a name="return"></a>
### Return
<div id="Functions$PHY10GKR_serdes_cdr_lock$description$return" data-type="text">

This function returns 0 on success and 1 on failure.


</div>


 -------------------------------- 
<a name="phy10gkrserdesdfecal"></a>
## PHY10GKR_serdes_dfe_cal
<a name="prototype"></a>
### Prototype 

<div id="Functions$PHY10GKR_serdes_dfe_cal$prototype" data-type="code">

    uint32_t
    PHY10GKR_serdes_dfe_cal
    (
        void 
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PHY10GKR_serdes_dfe_cal$description" data-type="text">

The PHY10GKR_serdes_dfe_cal() is a weak function that can be overridden by the
user to determine that the XCVR instance integrated has completed DFE
calibration.

Note this function should constantly check if the link training failure time has
timeout and if so exit the function.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
<div id="" data-type="text" data-name="None">

This function takes no parameters.


</div>

<a name="return"></a>
### Return
<div id="Functions$PHY10GKR_serdes_dfe_cal$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="phy10gkrserdestxequalization"></a>
## PHY10GKR_serdes_tx_equalization
<a name="prototype"></a>
### Prototype 

<div id="Functions$PHY10GKR_serdes_tx_equalization$prototype" data-type="code">

    void
    PHY10GKR_serdes_tx_equalization
    (
        uint32_t tx_main_tap,
        uint32_t tx_post_tap,
        uint32_t tx_pre_tap
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PHY10GKR_serdes_tx_equalization$description" data-type="text">

The PHY10GKR_serdes_tx_equalization() is a weak function that can be overridden
by the user to set the current XCVR tap coefficients.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### 
<div id="" data-type="text" data-name="">


</div>

<a name="return"></a>
### Return
<div id="Functions$PHY10GKR_serdes_tx_equalization$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
