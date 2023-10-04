<html>
 
 ------------------------------------ 

# Core10GBaseKR_PHY Bare Metal Driver
 
-----------------------------------------
## Table of Contents 
  - [Introduction](#introduction)

  - [Hardware Flow Dependencies](#hardware-flow-dependencies)

  - [Theory of Operation](#theory-of-operation)

     - [Configuration](#configuration)

     - [Initialization](#initialization)

     - [10GBASE-KR](#10gbase-kr)

     - [Clause74: FEC](#clause74:-fec)

     - [Clause73: Auto-negotiation](#clause73:-auto-negotiation)

     - [Clause72: Link Training](#clause72:-link-training)

- [Types](#types)
  - [phy10gkr_error_t](#phy10gkrerrort)
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
  - [phy10gkr_xcvr_api_t](#phy10gkrxcvrapit)
  - [phy10gkr_cfg_t](#phy10gkrcfgt)
  - [phy10gkr_instance_t](#phy10gkrinstancet)

- [Constants](#constants)
  - [CORE10GBASEKR_PHY VERSION TAGS](#core10gbasekrphy-version-tags)
  - [CORE10GBASEKR_PHY FEC ABILITY](#core10gbasekrphy-fec-ability)
  - [CORE10GBASEKR_PHY NO FEC ABILITY](#core10gbasekrphy-no-fec-ability)
  - [CORE10GBASEKR_PHY ENABLE FEC REQUEST](#core10gbasekrphy-enable-fec-request)
  - [CORE10GBASEKR_PHY DISABLE FEC REQUEST](#core10gbasekrphy-disable-fec-request)
  - [CORE10GBASEKR_PHY FEC NEGOTIATED](#core10gbasekrphy-fec-negotiated)
  - [CORE10GBASEKR_PHY FEC NOT NEGOTIATED](#core10gbasekrphy-fec-not-negotiated)
  - [CORE10GBASEKR_PHY LT MAX/MIN LIMITS](#core10gbasekrphy-lt-maxmin-limits)
  - [CORE10GBASEKR_PHY PRESET](#core10gbasekrphy-preset)
  - [CORE10GBASEKR_PHY INIT](#core10gbasekrphy-init)
  - [CORE10GBASEKR_PHY LP REQUEST](#core10gbasekrphy-lp-request)
  - [CORE10GBASEKR_PHY AN LINK FAIL INHIBIT TIMER](#core10gbasekrphy-an-link-fail-inhibit-timer)

- [Functions](#functions)
  - [PHY10GKR_cfg_struct_def_init](#phy10gkrcfgstructdefinit)
  - [PHY10GKR_init](#phy10gkrinit)
  - [PHY10GKR_10gbasekr_sm](#phy10gkr10gbasekrsm)
  - [PHY10GKR_get_current_time_ms](#phy10gkrgetcurrenttimems)
  - [PHY10GKR_get_ip_version](#phy10gkrgetipversion)
  - [PHY10GKR_get_driver_version](#phy10gkrgetdriverversion)

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
the Core10GBaseKR_PHY embedded software driver.

# Hardware Flow Dependencies
This driver covers the configuration details of features such as the clauses 72
(Link Training), 73 (Auto-negotiation), and 74 (Forward Error Correction) of
IEEE802.3.

See the <a href="https://mi-v-ecosystem.github.io/redirects/miv-rv32-ip-user-
guide-core10gbasekr_phy">Core10GBaseKR_PHY User Guide</a> for a detailed
description of design requirements when interfacing the Core10GBaseKR_PHY to a
transceiver.

# Theory of Operation
The Core10GBaseKR_PHY driver functions are grouped into the following
categories:

  - Configuration
  - Initialization
  - 10GBASE-KR
  - Clause74: Forward Error Correction (FEC)
  - Clause73: Auto-negotiation
  - Clause72: Link training

## Configuration
The Core10GBaseKR_PHY driver requires an instance of a PHY configuration
phy10gkr_cfg_t to be initialised. To load phy10gkr_cfg_t with default
configurations, this configuration instance is passed by reference to
PHY10GKR_cfg_struct_def_init(). This configuration structure may be overridden
with alternative configurations before initialising the driver.

The application must point to an instance of the Transceiver (XCVR) per instance
of a phy10gkr_instance_t. The phy10gkr_instance_t structure members xcvr and
xcvr_api must point to the XCVR that the Core10GBaseKR_PHY IP is interfacing
with and the appropriate XCVR APIs for dynamic configuration.

## Initialization
The Core10GBaseKR_PHY driver is initialized through a call to PHY10GKR_init().
The PHY10GKR_init() function must be called before calling any other
Core10GBaseKR_PHY driver functions.

## 10GBASE-KR
The full 10GBASE-KR flow is handled by calling PHY10GKR_10gbasekr_sm(). As this
function is dependent on interacting with a transceiver, transceiver function
pointers must be configured by the user to point to the transceiver APIs that
dynamically configure the transceiver implemented in the hardware design.

Each time PHY10GKR_10gbasekr_sm() is in the AN_SERDES_CONFIG state, the private
function phy_10gbasekr_reset(), is called, which will load any user
configurations to the Core registers and resets all algorithm and debug
counters.

This API uses two private function calls, phy_10gbasekr_an() and
phy_10gbasekr_lt(), to enable and interact with clause 72 and 73 hardware blocks
within the Core10GBaseKR_PHY IP.

## Clause74: FEC
To enable FEC, FEC must be configured in the Core10GBaseKR_PHY IP. See the <a
href="https://mi-v-ecosystem.github.io/redirects/miv-rv32-ip-user-guide-
core10gbasekr_phy">Core10GBaseKR_PHY User Guide</a> for a detailed description
of how to enable FEC logic using the IP configurator.

During auto-negotiation initialization, if the FEC block has been configured in
the Core, the driver will set the FEC ability bit 46 in the advertisement
ability DME page.

The fec_request member within the phy10gkr_cfg_t structure may be set prior to
driver initialisation to indicate that the driver should set the FEC request bit
47 in the advertisement ability clause 73 DME page. The driver implements error
checking which identifies if fec_request is set when FEC is not configured
within the Core.

## Clause73: Auto-negotiation
The IEEE802.3 clause 73 auto-negotiation is enabled and executed by the private
function, phy_10gbasekr_an(). PHY10GKR_10gbasekr() uses this private function
enable auto-negotiation and determines when the hardware has reached the
AN_GOOD_CHECK state.

## Clause72: Link Training
The IEEE802.3 clause 72 link training is enabled and executed by the private
function phy_10gbasekr_lt(). The Core10GBaseKR_PHY IP and the Core10GBaseKR_PHY
embedded software driver together carry out the link training. The driver
initiates the link training and takes appropriate actions depending on the
events indicated by the 10GBaseKR status register bits during the link training
process.

Training Failure: The training failure bit is set by the IP when the
Core10GBaseKR_PHY link training timer exceeds 500 ms. The driver also implements
a soft timer as an additional protection layer. PHY10GKR_get_current_time_ms()
must be overridden by instantiating this function in user code, so that the
current time of a timer will be returned in milli-seconds. When this status is
set by Core10GBaseKR_PHY, the embedded software must reduce the XCVR data rate
by calling PHY10GKR_serdes_an_config() and restart the auto-negotiation state
machine by calling phy_10gbasekr_an().

Rx Calibration: The IP sets this status bit to indicate that there is a received
status report of Max/Min/Updated that the Rx calibration algorithm must handle.
The maximum to minimum sweep algorithm described in the <a href="https://mi-v-
ecosystem.github.io/redirects/miv-rv32-ip-user-guide-
core10gbasekr_phy">Core10GBaseKR_PHY User Guide</a> is implemented by the
functions which are defined within core10gbasekr_phy_link_training.h.

After the Rx calibration algorithm completes, the driver updates the transmit
coefficient with new transmitter tap, which is sent to the link partner.

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

</div>


# Types

 ---------------- 
<a name="phy10gkrerrort"></a>
## phy10gkr_error_t
<a name="prototype"></a>
### Prototype 

<div id="Types$phy10gkr_error_t$prototype" data-type="code">

 ``` 
    typedef enum {
        PHY10GKR_ERR_USER_CONFIG = 1,
        PHY10GKR_ERR_NO_XCVR = 2,
        PHY10GKR_ERR_XCVR_API_FUNCTION_POINTER = 3
    } phy10gkr_error_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_error_t$description" data-type="text">

The phy10gkr_error_t enumeration acts as an error lookup.

  - PHY10GKR_ERR_USER_CONFIG implies that the config structure is Null.
  - PHY10GKR_ERR_NO_XCVR implies that there is no pointer to an instance of a 
XCVR.
  - PHY10GKR_ERR_XCVR_API_FUCNTION_POINTER implies that at least one XCVR API 
function pointer is null. 


</div>


 --------------------------- 
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
        uint64_t adv_ability; 
        uint64_t lp_bp_adv_ability; 
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

The phy10gkr_calirbation_request_t enumeration identifies the initial conditions
of a device. For example, the local device will calibrate using a preset
request.


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

The phy10gkr_tx_equalizer_tap_t enumeration specifies the three different
transmitter taps.


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

The phy10gkr_tap_cal_state_t enumeration specifies the state of the link partner
calibration algorithm.


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

The phy10gkr_local_rxcvr_lock_t enumeration specifies the condition of the link
partner calibration algorithm. During training, when the link partner has been
calibrated, the local receiver ready lock is locked and the status report is
updated to notify the link partner.


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
        uint32_t dec_cnt; 
        phy10gkr_tap_cal_state_t lp_tap_cal_state; 
        uint32_t optimal_index; 
        uint32_t optimal_cnt; 
    } phy10gkr_coeff_update_t ;
  ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_coeff_update_t$description" data-type="text">

The phy10gkr_coeff_update_t struct describes an instance of the link training
coefficient update. This structure supports calibrating the link partners
transmitter taps.


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

The phy10gkr_lt_instance_t struct describes an instance of the link training
parameters.


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

The phy10gkr_state_t enumeration identifies the state of the 10GBASE-KR state
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

The phy10gkr_status_t enumeration identifies the status of the 10GBASE-KR state
machine.

This enumeration can identify failures encountered by the 10GBASE-KR algorithm.


</div>


 --------------------------- 
<a name="phy10gkrxcvrapit"></a>
## phy10gkr_xcvr_api_t
<a name="prototype"></a>
### Prototype 

<div id="Types$phy10gkr_xcvr_api_t$prototype" data-type="code">

``` 
    typedef struct __phy10gkr_xcvr_api {
        uint8_t(*init; 
        uint8_t(*auto_neg_data_rate; 
        uint8_t(*link_training_data_rate; 
        uint8_t(*cdr_lock; 
        uint8_t(*ctle_cal; 
        uint8_t(*ctle_cal_status; 
        uint8_t(*dfe_cal; 
        uint8_t(*dfe_cal_status; 
        uint8_t(*reset_pcs_rx; 
        uint8_t(*tx_equalization; 
    } phy10gkr_xcvr_api_t ;
  ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_xcvr_api_t$description" data-type="text">

The phy10gkr_xcvr_api_t structure identifies the required XCVR APIs that this
driver requires to complete IEEE802.3 Clause 72 and 73. All function pointers
require a return value of type uint8_t. Each function that the pointers point to
require void pointer to an XCVR instance, with the exception of tx_equalization
which requires three additional parameters, absolute tap coefficients.

  - init: Initialise the XCVR which is implemented in the hardware design.
  - auto_neg_data_rate: Configure the XCVR for auto-negotiation.
  - link_training_data_rate: Configure the XCVR for link training.
  - cdr_lock: Check if CDR is locked and return success or failure, where 0 is 
success.
  - ctle_cal: Start CTLE calibration.
  - ctle_cal_status: Check if CTLE is complete and return success or failure, 
where 0 is success.
  - dfe_cal: Start DFE calibration.
  - dfe_cal_status: Check if DFE is complete and return success or failure, 
where 0 is success.
  - reset_pcs_rx: Reset the XCVR PCS RX path.
  - tx_equalization: Set the XCVR coefficients which are passed as parameters, 
where tx_main_tap implies C(0), tx_post_tap implies C(+1) and tx_pre_tap implies
 C(-1). These coefficients are absolute but the XCVR may require the signed 
value. 


</div>


 --------------------------- 
<a name="phy10gkrcfgt"></a>
## phy10gkr_cfg_t
<a name="prototype"></a>
### Prototype 

<div id="Types$phy10gkr_cfg_t$prototype" data-type="code">

``` 
    typedef struct __phy10gkr_cfg {
        phy10gkr_xcvr_api_t xcvr_api; 
        uint32_t fec_request; 
        uint32_t rx_calibration_request; 
        uint32_t main_preset_tap_coeff; 
        uint32_t post_preset_tap_coeff; 
        uint32_t pre_preset_tap_coeff; 
        uint32_t main_initialize_tap_coeff; 
        uint32_t post_initialize_tap_coeff; 
        uint32_t pre_initialize_tap_coeff; 
        uint32_t main_max_tap_ceoff; 
        uint32_t main_min_tap_ceoff; 
        uint32_t post_max_tap_ceoff; 
        uint32_t post_min_tap_ceoff; 
        uint32_t pre_max_tap_ceoff; 
        uint32_t pre_min_tap_ceoff; 
    } phy10gkr_cfg_t ;
  ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$phy10gkr_cfg_t$description" data-type="text">

The phy10gkr_cfg_t struct describes an instance of the Core10GBaseKR_PHY
configuration parameters.


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
        uint32_t fec_configured; 
        uint32_t fec_negotiated; 
        void *xcvr; 
        phy10gkr_xcvr_api_t xcvr_api; 
        uint32_t fec_request; 
        phy10gkr_calirbation_request_t rx_calibration_request; 
        uint32_t main_preset_tap_coeff; 
        uint32_t post_preset_tap_coeff; 
        uint32_t pre_preset_tap_coeff; 
        uint32_t main_initialize_tap_coeff; 
        uint32_t post_initialize_tap_coeff; 
        uint32_t pre_initialize_tap_coeff; 
        uint32_t main_max_tap_ceoff; 
        uint32_t main_min_tap_ceoff; 
        uint32_t post_max_tap_ceoff; 
        uint32_t post_min_tap_ceoff; 
        uint32_t pre_max_tap_ceoff; 
        uint32_t pre_min_tap_ceoff; 
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
<div id="Constants$C10GBKR_VERSION_MAJOR$description" data-type="text">

<a name="c10gbkrversionmajor"></a>
## CORE10GBASEKR_PHY VERSION TAGS
The version tags define the major, minor and patch driver release tags

</div>

<div id="Constants$C10GBKR_FEC_ABILITY$description" data-type="text">

<a name="c10gbkrfecability"></a>
## CORE10GBASEKR_PHY FEC ABILITY
This defines that the auto-negotiation advertisement FEC ability bit should be
set as the IP is configured with FEC logic.

</div>

<div id="Constants$C10GBKR_NO_FEC_ABILITY$description" data-type="text">

<a name="c10gbkrnofecability"></a>
## CORE10GBASEKR_PHY NO FEC ABILITY
This defines that the auto-negotiation advertisement FEC ability bit should be
cleared as the IP is not configured with FEC logic.

</div>

<div id="Constants$C10GBKR_ENABLE_FEC_REQUEST$description" data-type="text">

<a name="c10gbkrenablefecrequest"></a>
## CORE10GBASEKR_PHY ENABLE FEC REQUEST
This defines that the auto-negotiation advertisement FEC request bit should be
set if the IP is configured with the FEC logic.

</div>

<div id="Constants$C10GBKR_DISABLE_FEC_REQUEST$description" data-type="text">

<a name="c10gbkrdisablefecrequest"></a>
## CORE10GBASEKR_PHY DISABLE FEC REQUEST
This defines that the auto-negotiation advertisement FEC request bit should be
cleared if the IP is configured with the FEC logic.

</div>

<div id="Constants$C10GBKR_FEC_NEGOTIATED$description" data-type="text">

<a name="c10gbkrfecnegotiated"></a>
## CORE10GBASEKR_PHY FEC NEGOTIATED
This defines that FEC was negotiated during auto-negotiation

</div>

<div id="Constants$C10GBKR_FEC_NOT_NEGOTIATED$description" data-type="text">

<a name="c10gbkrfecnotnegotiated"></a>
## CORE10GBASEKR_PHY FEC NOT NEGOTIATED
This defines that FEC was not negotiated during auto-negotiation

</div>

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
<a name="phy10gkrcfgstructdefinit"></a>
## PHY10GKR_cfg_struct_def_init
<a name="prototype"></a>
### Prototype 

<div id="Functions$PHY10GKR_cfg_struct_def_init$prototype" data-type="code">

    void
    PHY10GKR_cfg_struct_def_init
    (
        phy10gkr_cfg_t * cfg
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PHY10GKR_cfg_struct_def_init$description" data-type="text">

PHY10GKR_cfg_struct_def_init() loads the PHY configuration struct as default.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### cfg
<div id="Functions$PHY10GKR_cfg_struct_def_init$description$parameters$cfg" data-type="text" data-name="cfg">

The cfg parameter specifies the PHY configuration instance.


</div>

<a name="return"></a>
### Return
<div id="Functions$PHY10GKR_cfg_struct_def_init$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$PHY10GKR_cfg_struct_def_init$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$PHY10GKR_cfg_struct_def_init$description$example$Example1" data-type="code" data-name="Example">


```
    #include "core10gbasekr_phy.h"
    int main(void)
    {
      PHY10GKR_cfg_struct_def_init(&phy_cfg);
      phy_cfg.fec_requested = 1;

      PHY10GKR_init(&xcvr, &g_phy, CORE10GBKR_0_PHY_BASE_ADDR);
      PHY10GKR_config(&g_phy, &phy_cfg);
      return (0u);
    }
```

</div>


 -------------------------------- 
<a name="phy10gkrinit"></a>
## PHY10GKR_init
<a name="prototype"></a>
### Prototype 

<div id="Functions$PHY10GKR_init$prototype" data-type="code">

    uint32_t
    PHY10GKR_init
    (
        phy10gkr_instance_t * this_phy,
        addr_t base_addr,
        phy10gkr_cfg_t * cfg,
        void * xcvr
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PHY10GKR_init$description" data-type="text">

PHY10GKR_init() initializes the Core10GBaseKR_PHY bare-metal driver. This
function sets the base address of the Auto-negotiation, link-training, tx
control, and rx status registers.

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

#### cfg
<div id="Functions$PHY10GKR_init$description$parameters$cfg" data-type="text" data-name="cfg">

The cfg parameter specifies the PHY configuration instance.


</div>

#### xcvr
<div id="Functions$PHY10GKR_init$description$parameters$xcvr" data-type="text" data-name="xcvr">

This is a pointer to the instance of XCVR which is connected to this IP.


</div>

<a name="return"></a>
### Return
<div id="Functions$PHY10GKR_init$description$return" data-type="text">

This function returns success or failure, 0 implies success. On failure this
function will return phy10gkr_error_t enumeration which indicates the error
type.


</div>

##### Example
<div id="Functions$PHY10GKR_init$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$PHY10GKR_init$description$example$Example1" data-type="code" data-name="Example">


```
    #include "core10gbasekr_phy.h"
    int main(void)
    {
      void * xcvr;
      xcvr.base_addr = 0xFFFFFFFF;
      xcvr.lane = 1;
      xcvr.serdes = 2;

      PHY10GKR_init(&g_phy, CORE10GBKR_0_PHY_BASE_ADDR, &phy_cfg ,&xcvr);
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

PHY10GKR_10gbasekr_sm() executes the full 10GBASE-KR flow required to complete
the auto-negotiation and link training.

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

This function returns the phy10gkr_status_t enumeration which indicates the
status of the API state machine. 0 implies that a link has been established.


</div>

##### Example
<div id="Functions$PHY10GKR_10gbasekr_sm$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$PHY10GKR_10gbasekr_sm$description$example$Example1" data-type="code" data-name="Example">


```
    #include "core10gbasekr_phy.h"
    int main(void)
    {
      uint32_t status;
      PHY10GKR_cfg_struct_def_init(&phy_cfg);
      phy_cfg.fec_requested = 1;
      PHY10GKR_init(&g_phy, CORE10GBKR_0_PHY_BASE_ADDR, &phy_cfg, &xcvr);
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

PHY10GKR_get_current_time_ms() is a weak function that can be overridden by the
user to get the current time in milli-seconds.

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
<a name="phy10gkrgetipversion"></a>
## PHY10GKR_get_ip_version
<a name="prototype"></a>
### Prototype 

<div id="Functions$PHY10GKR_get_ip_version$prototype" data-type="code">

    uint8_t
    PHY10GKR_get_ip_version
    (
        phy10gkr_instance_t * this_phy,
        uint32_t * major,
        uint32_t * minor,
        uint32_t * sub
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PHY10GKR_get_ip_version$description" data-type="text">

PHY10GKR_get_ip_version() retrieves the IP version tags, the tags are passed by
reference as parameters and updated by this function. This function must be
called after initialization as there is a dependency on the instance of a
Core10GBaseKR_PHY instance.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_phy
<div id="Functions$PHY10GKR_get_ip_version$description$parameters$this_phy" data-type="text" data-name="this_phy">

The this_phy parameter specifies the PHY instance.


</div>

#### major
<div id="Functions$PHY10GKR_get_ip_version$description$parameters$major" data-type="text" data-name="major">

This parameter identifies the major version number.


</div>

#### minor
<div id="Functions$PHY10GKR_get_ip_version$description$parameters$minor" data-type="text" data-name="minor">

This parameter identifies the minor version number.


</div>

#### sub
<div id="Functions$PHY10GKR_get_ip_version$description$parameters$sub" data-type="text" data-name="sub">

This parameter identifies the sub version number.


</div>

<a name="return"></a>
### Return
<div id="Functions$PHY10GKR_get_ip_version$description$return" data-type="text">

This function returns 0 on success.


</div>

##### Example
<div id="Functions$PHY10GKR_get_ip_version$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$PHY10GKR_get_ip_version$description$example$Example1" data-type="code" data-name="Example">


```
    #include "core10gbasekr_phy.h"
    int main(void)
    {
      uint32_t major,
      uint32_t minor,
      uint32_t sub,

      PHY10GKR_cfg_struct_def_init(&phy_cfg);
      PHY10GKR_init(&g_phy, CORE10GBKR_0_PHY_BASE_ADDR, &phy_cfg, &xcvr);

      PHY10GKR_get_ip_version(&g_phy, &major, &minor, &sub);
      return (0u);
    }
```

</div>


 -------------------------------- 
<a name="phy10gkrgetdriverversion"></a>
## PHY10GKR_get_driver_version
<a name="prototype"></a>
### Prototype 

<div id="Functions$PHY10GKR_get_driver_version$prototype" data-type="code">

    uint8_t
    PHY10GKR_get_driver_version
    (
        uint32_t * major,
        uint32_t * minor,
        uint32_t * patch
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$PHY10GKR_get_driver_version$description" data-type="text">

PHY10GKR_get_driver_version() retrieves the driver version tags, the tags are
passed by reference as parameters and updated by this function.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### major
<div id="Functions$PHY10GKR_get_driver_version$description$parameters$major" data-type="text" data-name="major">

This parameter identifies the major version number.


</div>

#### minor
<div id="Functions$PHY10GKR_get_driver_version$description$parameters$minor" data-type="text" data-name="minor">

This parameter identifies the minor version number.


</div>

#### patch
<div id="Functions$PHY10GKR_get_driver_version$description$parameters$patch" data-type="text" data-name="patch">

This parameter identifies the patch version number.


</div>

<a name="return"></a>
### Return
<div id="Functions$PHY10GKR_get_driver_version$description$return" data-type="text">

This function returns 0 on success.


</div>

##### Example
<div id="Functions$PHY10GKR_get_driver_version$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$PHY10GKR_get_driver_version$description$example$Example1" data-type="code" data-name="Example">


```
    #include "core10gbasekr_phy.h"
    int main(void)
    {
      uint32_t major,
      uint32_t minor,
      uint32_t patch,

      PHY10GKR_get_driver_version(&major, &minor, &patch);
      return (0u);
    }
```

</div>


 -------------------------------- 
