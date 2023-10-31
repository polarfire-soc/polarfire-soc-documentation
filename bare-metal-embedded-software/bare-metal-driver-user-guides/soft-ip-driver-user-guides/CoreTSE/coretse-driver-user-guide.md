<html>
 
 ------------------------------------ 

# CoreTSE Bare Metal Driver
 
-----------------------------------------
## Table of Contents 
  - [Introduction](#introduction)

  - [Hardware Flow Dependencies](#hardware-flow-dependencies)

     - [PHY Interface Types](#phy-interface-types)

  - [Theory of Operation](#theory-of-operation)

     - [Configuration](#configuration)

     - [Initialization](#initialization)

     - [Transmit Operations](#transmit-operations)

     - [Receive Operations](#receive-operations)

     - [Statistics and Link Status](#statistics-and-link-status)

     - [Address-based Frame Filtering](#address-based-frame-filtering)

     - [Frame Drop Criterion](#frame-drop-criterion)

     - [Wake on LAN (WoL)](#wake-on-lan-(wol))

     - [Jumbo Frames](#jumbo-frames)

- [Types](#types)
  - [tse_speed_t](#tsespeedt)
  - [tse_stat_t](#tsestatt)
  - [tse_transmit_callback_t](#tsetransmitcallbackt)
  - [tse_receive_callback_t](#tsereceivecallbackt)
  - [tse_wol_callback_t](#tsewolcallbackt)
  - [tse_desc_t](#tsedesct)
  - [tse_cfg_t](#tsecfgt)
  - [tse_instance_t](#tseinstancet)

- [Constants](#constants)
  - [State definitions](#state-definitions)
  - [Packet Transmission](#packet-transmission)
  - [Link Status](#link-status)
  - [Maximum MAC frame size (packet size)](#maximum-mac-frame-size-packet-size)
  - [Transmit and receive packet buffer sizes](#transmit-and-receive-packet-buffer-sizes)
  - [Configuration Parameter Definitions](#configuration-parameter-definitions)
      - [FIFO error detection & correction](#fifo-error-detection-&-correction)
      - [Huge frame support](#huge-frame-support)
      - [Length field checking](#length-field-checking)
      - [Padding and CRC](#padding-and-crc)
      - [Append CRC](#append-crc)
      - [Full-duplex mode](#full-duplex-mode)
      - [Loopback mode](#loopback-mode)
      - [Receiver flow control](#receiver-flow-control)
      - [Transmission flow control](#transmission-flow-control)
      - [Excessive defer](#excessive-defer)
      - [No-backoff](#no-backoff)
      - [Backpressure no-backoff](#backpressure-no-backoff)
      - [Alternative binary exponential backoff](#alternative-binary-exponential-backoff)
      - [Suppress preamble](#suppress-preamble)
      - [Auto-scan PHYs](#auto-scan-phys)
      - [WoL event detection masks](#wol-event-detection-masks)
      - [Preamble length default value and maximum value](#preamble-length-default-value-and-maximum-value)
      - [Byte/Nibble mode](#bytenibble-mode)
  - [IPG/IFG values](#ipgifg-values)
  - [Number of maximum retransmission tries](#number-of-maximum-retransmission-tries)
  - [PHY clock divider values](#phy-clock-divider-values)
  - [PHY addresses](#phy-addresses)
  - [Maximum frame length](#maximum-frame-length)
  - [Slot Time (Collision window)](#slot-time-collision-window)
  - [Auto-negotiation options](#auto-negotiation-options)
  - [Speed configuration options](#speed-configuration-options)
  - [Frame Filter configuration options](#frame-filter-configuration-options)
  - [Default framedrop mask](#default-framedrop-mask)
      - [Framedrop if IFG is small](#framedrop-if-ifg-is-small)
      - [Framedrop if short RXDV Event](#framedrop-if-short-rxdv-event)
      - [Framedrop if False Carrier](#framedrop-if-false-carrier)
      - [Framedrop if Code error](#framedrop-if-code-error)
      - [Framedrop if CRC Error](#framedrop-if-crc-error)
      - [Framedrop if Length check error](#framedrop-if-length-check-error)
      - [Framedrop if Length Out of Range](#framedrop-if-length-out-of-range)
      - [Framedrop if an OK frame](#framedrop-if-an-ok-frame)
      - [Framedrop if Multicast frame](#framedrop-if-multicast-frame)
      - [Framedrop if Broadcast frame](#framedrop-if-broadcast-frame)
      - [Framedrop if Dribble Nibble](#framedrop-if-dribble-nibble)
      - [Framedrop if Control Frame](#framedrop-if-control-frame)
      - [Framedrop if PAUSE Control Frame](#framedrop-if-pause-control-frame)
      - [Framedrop if Unsupported Op-code](#framedrop-if-unsupported-op-code)
      - [Framedrop if VLAN Tag](#framedrop-if-vlan-tag)
      - [Framedrop if a Long Event](#framedrop-if-a-long-event)
      - [Framedrop if a Truncated frame](#framedrop-if-a-truncated-frame)
      - [Framedrop if Unicast frame detected](#framedrop-if-unicast-frame-detected)
      - [Framedrop if a short frame received](#framedrop-if-a-short-frame-received)
  - [PHY Address](#phy-address)
  - [Per Packet Override Masks](#per-packet-override-masks)

- [Functions](#functions)
  - [TSE_cfg_struct_def_init](#tsecfgstructdefinit)
  - [TSE_init](#tseinit)
  - [TSE_set_tx_callback](#tsesettxcallback)
  - [TSE_set_rx_callback](#tsesetrxcallback)
  - [TSE_set_wol_callback](#tsesetwolcallback)
  - [TSE_send_pkt](#tsesendpkt)
  - [TSE_receive_pkt](#tsereceivepkt)
  - [TSE_get_link_status](#tsegetlinkstatus)
  - [TSE_read_stat](#tsereadstat)
  - [TSE_clear_statistics](#tseclearstatistics)
  - [TSE_read_phy_reg](#tsereadphyreg)
  - [TSE_write_phy_reg](#tsewritephyreg)
  - [TSE_isr](#tseisr)
  - [TSE_set_address_filter](#tsesetaddressfilter)

<div id="TitlePage" data-type="text">

# Introduction
The Core Triple Speed Ethernet (CoreTSE) is a hardware soft IP core that
implements 10/100/1000 Mbps Ethernet Media Access Control (MAC). The CoreTSE
supports MII/GMII/TBI interfaces to the physical layer devices (PHY).

The only difference between CoreTSE_AHB and CoreTSE soft IP is that the
CoreTSE_AHB has DMA with an AHB interface for transmitting and receiving data
packets whereas the CoreTSE does not. For CoreTSE, no firmware intervention is
needed for packet transmit/receive DMA operations. All the other operations are
the same for CoreTSE_AHB and CoreTSE soft IP and this driver can be used to
execute them.

Note: This document covers the driver functionality for the CoreTSE_AHB and
CoreTSE. For brevity, we use the term CoreTSE to describe both in this document,
unless we are describing functionality specific to the CoreTSE_AHB.

This software driver provides a set of functions for controlling the CoreTSE as
part of a bare metal system where no operating system is available. This driver
can be adapted for use as part of an operating system, but the implementation of
the adaptation layer between the driver and the operating system's driver model
is outside the scope of the driver.

# Hardware Flow Dependencies
The configuration of all features of the CoreTSE_AHB or CoreTSE soft IP is
covered by this driver except for the selection of the Ethernet PHY, the PHY
interface, the size of transmit and receive rings, and the MDIO address of the
TBI/1000BaseX module within CoreTSE.

The PHY interface type is selected using the Firmware Catalogue configuration
dialogue window. This driver supports MII, GMII and TBI interfaces. See the "PHY
Interface Types" sub-section for an explanation of the PHY interface.

The size of the transmit and receive rings defines the maximum number of
transmit and receive packets that can be queued.

The MDIO address of the TBI module within CoreTSE is also selected using the
Firmware Catalogue configuration dialogue window.

Note: This selection is applicable only when CoreTSE is configured to operate in
TBI mode and is ignored when CoreTSE is configured to operate in G/MII mode.

The system may use the CoreTSE in GMII mode and use an external CoreRGMII soft
IP to interface with a PHY using RGMII interface type. If the CoreRGMII is being
used in the system, then the MDIO address of the CoreRGMII also needs to be
configured here for this driver.

Your application software should configure the CoreTSE driver by calling the
`TSE_init()` function for each CoreTSE instance in the hardware design.

## PHY Interface Types
  - MII: CoreTSE operates in MII mode and interfaces with MII PHY directly.
  - GMII: CoreTSE operates in GMII mode and interfaces with GMII PHY directly.
  - TBI: CoreTSE operates in TBI mode (Internal TBI/1000BaseX module is 
enabled).

# Theory of Operation
The CoreTSE software driver is designed to allow the control of multiple
instances of CoreTSE. Each instance of CoreTSE in the hardware design is
associated with a single instance of the tse_instance_t structure in the
software. You need to allocate memory for one unique tse_instance_t structure
instance for each CoreTSE hardware instance. The contents of these data
structures are initialized by calling `TSE_init()`. A pointer to this structure
is passed to subsequent driver functions to identify the CoreTSE hardware
instance you wish to perform the requested operation on.

Note: Do not attempt to directly manipulate the content of tse_instance_t
structures. This structure is only intended to be modified by the driver
function.

The CoreTSE driver functions are grouped into the following categories:

  - Initialization and configuration
  - Transmit operations
  - Receive operations
  - Reading link status and statistics
  - Address-based frame filtering
  - Wake up on LAN (WoL) with unicast match and AMD magic packet detection
  - Jumbo Frames

## Configuration
The CoreTSE driver is initialized and configured by calling the `TSE_init()`
function. The `TSE_init()` function takes a pointer to a configuration data
structure as a parameter. This data structure contains all the configuration
information required to initialize and configure the CoreTSE.

## Initialization
The CoreTSE driver provides the `TSE_cfg_struct_def_init()` function to
initialize the configuration data structure to the default value. It is
recommended using this function to retrieve the default configuration and then
overwrite the defaults with the application-specific settings such as PHY
address, allowed link speeds, link duplex mode, and MAC address.

The `TSE_init()` function must be called before any other CoreTSE driver
functions. The `TSE_cfg_struct_def_init()` is the only function which can be
called before calling the `TSE_init()` function.

The following functions are used as part of the initialization and configuration
process:

  - `TSE_cfg_struct_def_init()`
  - `TSE_init()`

## Transmit Operations
The CoreTSE driver transmit operations are interrupt driven. The application
must register a transmit call-back function with the driver using the
`TSE_set_tx_callback()` function. The CoreTSE driver calls this call-back
function every time a packet is sent.

The application must call the `TSE_send_pkt()` function every time the user
wants to transmit a packet. The application passes a pointer to the buffer
containing the packet to send. It is the user's responsibility to manage the
memory allocated to store the transmit packets. The CoreTSE driver only requires
a pointer to the buffer containing the packet and the packet size. The CoreTSE
driver calls the transmit call-back function registered using the
`TSE_set_tx_callback()` function once a packet is sent. The transmit call-back
function is supplied by the application and can be used to release the memory
used to store the packet that was sent.

The following functions are used as part of transmit and receive operations:

  - `TSE_send_pkt()`
  - `TSE_set_tx_callback()`

Note: This operation is applicable only for CoreTSE_AHB and this operation is
not supported on CoreTSE soft IP.

## Receive Operations
The CoreTSE driver receive operations are interrupt driven. The application must
first register a receive call-back function using the `TSE_set_rx_callback()`
function. The application can then allocate receive buffers to the CoreTSE
driver by calling the `TSE_receive_pkt()` function. This function can be called
multiple times to allocate more than one receive buffer. The CoreTSE driver
calls the receive call-back function whenever a packet is received into one of
the receive buffer. The driver hands back the receive buffer to the application
for packet processing. The CoreTSE driver won't reuse the receive buffer unless
a call to `TSE_receive_pkt()` re-allocates it to the driver.

The following functions are used as part of transmit and receive operations:

  - `TSE_receive_pkt()`
  - `TSE_set_rx_callback()`

Note: This operation is applicable only for CoreTSE_AHB and this operation is
not supported on CoreTSE soft IP.

## Statistics and Link Status
The CoreTSE driver provides the following functions to retrieve the current link
status and statistics.

  - `TSE_get_link_status()`
  - `TSE_read_stat()`

## Address-based Frame Filtering
The CoreTSE performs frame filtering based on the destination MAC address of the
received frame. The `TSE_init()` function initializes the CoreTSE hardware to a
default filtering mode where only broadcast frames and frames with a destination
address equal to the local base station MAC address are passed to the MAC. The
Broadcast frames are usually required by an application to obtain an IP address.
This default configuration does not need the frame filtering hash table to be
filled. This default configuration is returned with the
`TSE_cfg_struct_def_init()` function. You may change the configuration before
passing it to the `TSE_init()` function.

The application can use the `TSE_set_address_filter()` function to overwrite the
frame filter choice, which was selected at initialization time. The application
must provide the list of MAC addresses from which it wants to receive frames
(allowed MAC addresses list) to this driver using the `TSE_set_address_filter()`
function. If a received frame contains one of the MAC addresses contained in the
allowed MAC addresses list then that frame is passed.

The setting of the broadcast, hash-unicast/multicast configuration in Frame Pass
Control register is inferred from the values of the MAC addresses passed to the
driver in the allowed MAC addresses list.

  - If all MAC addresses included in the allowed MAC addresses list are unicast 
then hash-unicast filtering is selected. This results in more unwanted frames 
being rejected.
  - If all MAC addresses included in the allowed MAC addresses list are 
multicast then hash-multicast filtering is selected.
  - If the allowed MAC addresses list contains a mix of unicast and multicast 
MAC addresses, then hash-unicast and hash-multicast filtering is selected.
  - Broadcast filtering is selected if the broadcast MAC address 
`(FF:FF:FF:FF:FF:FF)` is included in the allowed MAC addresses list.

If the allowed MAC addresses list contains a mix of broadcast, unicast, and
multicast MAC addresses then broadcast, hash-unicast and hash-multicast
filtering is selected. The local base station MAC address must be included in
the allowed MAC addresses list if the application wants to receive frames
addressed to it after calling the `TSE_set_address_filter()` function.

The filtering is not perfect since the filtering hardware uses a hash table.
Therefore, some frames with addresses are not included in the allowed MAC
addresses list are still passed because the hash value for their MAC address is
identical to the hash value of an allowed MAC address. The hash
unicast/multicast setting allows further reduction of the number of unwanted
frames passed by both checking the MAC address against the hash table and
checking the received MAC address unicast/multicast bit against the hash
unicast/multicast setting of the filtering IP.

The following function is used as part of the frame filtering

  - `TSE_set_address_filter()`

## Frame Drop Criterion
Apart from the address-based frame filtering, the CoreTSE provides the facility
to drop frames based on certain criteria. The criteria for the frame drop logic
can be configured using the framedrop_mask element of the tse_cfg_t structure.
The value of this element is set to TSE_DEFVAL_FRAMEDROP_MASK by the
`TSE_cfg_struct_def_init()` function. This configuration value enables CoreTSE
to drop frames which have CRC errors, code errors, and frames with unsupported
op-codes. You may change the configuration before passing it to the `TSE_init()`
function.

## Wake on LAN (WoL)
The CoreTSE supports the Wake on LAN feature with unicast match frames and AMD
magic packets. The WoL interrupt is generated when this feature is enabled and
one of the two frames mentioned above is received.

`TSE_cfg_struct_def_init()` disables WoL. You may change the configuration
before passing it to the `TSE_init()` function. You can enable this feature
using TSE_WOL_UNICAST_FRAME_DETECT_EN or TSE_WOL_MAGIC_FRAME_DETECT_EN or both
the constants with a bitwise OR operation.

The callback handler function must be provided to this driver using
`TSE_set_wol_callback()` function. A function of type tse_wol_callback_t must be
provided as a parameter to this function. The driver calls this function when
the WoL event happens.

The following function is used as part of WoL:

  - `TSE_set_wol_callback()`

## Jumbo Frames
The CoreTSE_AHB supports jumbo frames that exceed the 1500 byte max of the
standard Ethernet frame, up to 4000 bytes long. This driver provides
TSE_JUMBO_PACKET_SIZE constant which can be used to define the Tx-Rx buffer size
where jumbo frame support is required.

</div>


# Types

 ---------------- 
<a name="tsespeedt"></a>
## tse_speed_t
<a name="prototype"></a>
### Prototype 

<div id="Types$tse_speed_t$prototype" data-type="code">

 ``` 
    typedef enum {
        TSE_MAC10MBPS = 0x00,
        TSE_MAC100MBPS = 0x01,
        TSE_MAC1000MBPS = 0x02,
        TSE_INVALID_SPEED = 0x03
    } tse_speed_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$tse_speed_t$description" data-type="text">

This type definition provides various MAC interface speeds supported by CoreTSE
IP.


</div>


 --------------------------- 
<a name="tsestatt"></a>
## tse_stat_t
<a name="prototype"></a>
### Prototype 

<div id="Types$tse_stat_t$prototype" data-type="code">

 ``` 
    typedef enum {
        TSE_FRAME_CNT_64, <--18-bit 
        TSE_FRAME_CNT_127, <--18-bit 
        TSE_FRAME_CNT_255, <--18-bit 
        TSE_FRAME_CNT_511, <--18-bit 
        TSE_FRAME_CNT_1K, <--18-bit 
        TSE_FRAME_CNT_MAX, <--18-bit 
        TSE_FRAME_CNT_VLAN, <--18-bit 
        TSE_RX_BYTE_CNT, <--24-bit 
        TSE_RX_PKT_CNT, <--18-bit 
        TSE_RX_FCS_ERR_CNT, <--12-bit 
        TSE_RX_MULTICAST_PKT_CNT, <--18-bit 
        TSE_RX_BROADCAST_PKT_CNT, <--22-bit 
        TSE_RX_CTRL_PKT_CNT, <--18-bit 
        TSE_RX_PAUSE_PKT_CNT, <--12-bit 
        TSE_RX_UNKNOWN_OPCODE_CNT, <--12-bit 
        TSE_RX_ALIGN_ERR_CNT, <--12-bit 
        TSE_RX_FRAMELENGTH_ERR_CNT, <--16-bit 
        TSE_RX_CODE_ERR_CNT, <--12-bit 
        TSE_RX_CS_ERR_CNT, <--12-bit 
        TSE_RX_UNDERSIZE_PKT_CNT, <--12-bit 
        TSE_RX_OVERSIZE_PKT_CNT, <--12-bit 
        TSE_RX_FRAGMENT_CNT, <--12-bit 
        TSE_RX_JABBER_CNT, <--12-bit 
        TSE_RX_DROP_CNT, <--12-bit 
        TSE_TX_BYTE_CNT, <--24-bit 
        TSE_TX_PKT_CNT, <--18-bit 
        TSE_TX_MULTICAST_PKT_CNT, <--18-bit 
        TSE_TX_BROADCAST_PKT_CNT, <--18-bit 
        TSE_TX_PAUSE_PKT_CNT, <--12-bit 
        TSE_TX_DEFFERAL_PKT_CNT, <--12-bit 
        TSE_TX_EXCS_DEFFERAL_PKT_CNT, <--12-bit 
        TSE_TX_SINGLE_COLL_PKT_CNT, <--12-bit 
        TSE_TX_MULTI_COLL_PKT_CNT, <--12-bit 
        TSE_TX_LATE_COLL_PKT_CNT, <--12-bit 
        TSE_TX_EXCSS_COLL_PKT_CNT, <--12-bit 
        TSE_TX_TOTAL_COLL_PKT_CNT, <--13-bit 
        TSE_TX_PAUSE_HONORED_CNT, <--12-bit 
        TSE_TX_DROP_CNT, <--12-bit 
        TSE_TX_JABBER_CNT, <--12-bit 
        TSE_TX_FCS_ERR_CNT, <--12-bit 
        TSE_TX_CNTRL_PKT_CNT, <--12-bit 
        TSE_TX_OVERSIZE_PKT_CNT, <--12-bit 
        TSE_TX_UNDERSIZE_PKT_CNT, <--12-bit 
        TSE_TX_FRAGMENT_CNT, <--12-bit 
        TSE_MAC_LAST_STAT
    } tse_stat_t;


 ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$tse_stat_t$description" data-type="text">

This type definition provides various transmit and receive statistics supported
by this driver. The desired statistics value to be read must be passed to the
`TSE_read_stat()` function. The width of returned statistic value is mentioned
in the comment against the statistic.


</div>


 --------------------------- 
<a name="tsetransmitcallbackt"></a>
## tse_transmit_callback_t
<a name="prototype"></a>
### Prototype 

<div id="Types$tse_transmit_callback_t$prototype" data-type="code">

`typedef void(* tse_transmit_callback_t) (void *p_user_data)`

</div>

<a name="description"></a>
### Description 

<div id="Types$tse_transmit_callback_t$description" data-type="text">

The application must use this function prototype to define the transmit
completion handler call-back function, which can be provided as a parameter to
`TSE_set_tx_callback()`.


</div>


 --------------------------- 
<a name="tsereceivecallbackt"></a>
## tse_receive_callback_t
<a name="prototype"></a>
### Prototype 

<div id="Types$tse_receive_callback_t$prototype" data-type="code">

`typedef void(* tse_receive_callback_t) (uint8_t *p_rx_packet, uint32_t pckt_length, void *p_user_data)`

</div>

<a name="description"></a>
### Description 

<div id="Types$tse_receive_callback_t$description" data-type="text">

The application must use this function prototype to define the receive call-back
listener function, which can be provided as a parameter to
`TSE_set_rx_callback()`.


</div>


 --------------------------- 
<a name="tsewolcallbackt"></a>
## tse_wol_callback_t
<a name="prototype"></a>
### Prototype 

<div id="Types$tse_wol_callback_t$prototype" data-type="code">

`typedef void(* tse_wol_callback_t) (void)`

</div>

<a name="description"></a>
### Description 

<div id="Types$tse_wol_callback_t$description" data-type="text">

The application must use this function prototype to define the WoL event handler
function, which can be provided as a parameter to `TSE_set_wol_callback()`.


</div>


 --------------------------- 
<a name="tsedesct"></a>
## tse_desc_t
<a name="prototype"></a>
### Prototype 

<div id="Types$tse_desc_t$prototype" data-type="code">

``` 
    typedef struct tse_desc {
        uint32_t pkt_start_addr; <--Packet start address 
        uint32_t pkt_size; <--Packet size & Per packet override flags 
        uint32_t next_desriptor; <--Link to next descriptor 
        uint32_t index; <--Index: helps in handling interrupts 
        void *caller_info; <--Pointer to user specific data 
    } tse_desc_t ;
  ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$tse_desc_t$description" data-type="text">

The driver creates and manages two DMA descriptor rings for transmission and
reception.


</div>


 --------------------------- 
<a name="tsecfgt"></a>
## tse_cfg_t
<a name="prototype"></a>
### Prototype 

<div id="Types$tse_cfg_t$prototype" data-type="code">

``` 
    typedef struct tse_cfg {
        uint32_t framefilter; <--Address based frame filter configuration 
        uint32_t speed_duplex_select; <--Link speed and duplex mode allowed to set up a link. 
        uint8_t mac_addr; <--Station's MAC address 
        uint8_t phy_addr; <--Address of Ethernet PHY on MII management interface. 
        uint8_t tx_edc_enable; <--Enable/disable error detection and correction for Tx FIFOs 
        uint8_t rx_edc_enable; <--Enable/disable error detection and correction for Rx FIFOs 
        uint8_t preamble_length; <--4-bit Length of preamble field: default value is 0x7 
        uint8_t hugeframe_enable; <--Enable/disable huge frame support: default is disable 0 
        uint8_t length_field_check; <--Enable/disable length field checking 
        uint8_t pad_n_CRC; <--Enable/disable padding and appending CRC 
        uint8_t append_CRC; <--Enable/disable appending CRC 
        uint8_t fullduplex; <--Enable/disable full duplex: default is disable 0 
        uint8_t loopback; <--Enable/disable loopback mode: default is disable 0 
        uint8_t rx_flow_ctrl; <--Enable/disable receiver flow control: default is disable 0 
        uint8_t tx_flow_ctrl; <--Enable/disable transmitter flow control: default is disable 0 
        uint8_t min_IFG; <--8-bit minimum interframe gap value 
        uint8_t btb_IFG; <--7-bit back to back interframe gap value 
        uint8_t max_retx_tries; <--5-bit maximum retransmission tries value: default is 0xF 
        uint8_t excessive_defer; <--Enable/disable transmission of packets that exceeded max collisions: default is disable 0 
        uint8_t nobackoff; <--Enable/disable back-off. default is disable 0 
        uint8_t backpres_nobackoff; <--Enable/disable back-off in back pressure mode: default is disable 0 
        uint8_t ABEB_enable; <--Enable/disable arbitrary binary exponential back-off: default is disable 0 
        uint8_t ABEB_truncvalue; <--4-bit alternative binary exponential back-off value: default is 0xA 
        uint8_t phyclk; <--3-bit MGMT clock divider value 
        uint8_t supress_preamble; <--Enable/disable preamble suppression at PHY: default is disable 0 
        uint8_t autoscan_phys; <--Enable/disable auto scanning of PHYs with programmed addresses: default is disable 0 
        uint16_t max_frame_length; <--Maximum frame length: default value is 0x0600(1536d) 
        uint16_t non_btb_IFG; <--14-bit non-back-to-back interframe gap value 
        uint16_t slottime; <--10-bit collision window value : default is 0x37 
        uint32_t framedrop_mask; <--18-bit mask to drop frames based on receive statistics 
        uint8_t wol_enable; <--Enable/disable WoL event detect feature 
        uint8_t aneg_enable; <--Enable/disable auto-negotiation in MSGMII module as well as phy (if PHY is in the system) 
    } tse_cfg_t ;
  ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$tse_cfg_t$description" data-type="text">

**CoreTSE Configuration Structure**

You need to create a record of this type to hold the configuration of the
CoreTSE. `TSE_cfg_struct_def_init()` must be used to initialize the
configuration record to the default values. Later, the configuration elements in
the record can be changed to the desired values.

**framefilter**

The framefilter configuration parameter specifies the address-based frame
filtering choice. The framefilter configuration can be set to a bitmask of the
following defines to specify the address-based frame filtering mode:

  - TSE_FC_PASS_BROADCAST_MASK
  - TSE_FC_PASS_MULTICAST_MASK
  - TSE_FC_PASS_UNICAST_MASK
  - TSE_FC_PROMISCOUS_MODE_MASK
  - TSE_FC_PASS_UNICAST_HASHT_MASK
  - TSE_FC_PASS_MULTICAST_HASHT_MASK
  - TSE_FC_DEFAULT_MASK

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_FC_DEFAULT_MASK to indicate that only frames with broadcast and unicast
match should be received - all other frames are dropped.

Note: `TSE_set_address_filter()` must be used to set the list of destination MAC
addresses which are passed when the constants TSE_FC_PASS_UNICAST_HASHT_MASK
and/or TSE_FC_PASS_MULTICAST_HASHT_MASK is/are specified.

**speed_duplex_select**

The speed_duplex_select configuration parameter specifies the allowed link
speeds. It is a bit-mask of the allowed link speed and duplex modes. The
speed_duplex_select configuration can be set to a bitmask of the following
defines to specify the allowed link speed and duplex mode:

  - TSE_ANEG_10M_FD
  - TSE_ANEG_10M_HD
  - TSE_ANEG_100M_FD
  - TSE_ANEG_100M_HD
  - TSE_ANEG_1000M_FD

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_ANEG_ALL_SPEEDS indicating that a link is set up for the best available
speed and duplex combination.

**mac_addr**

The mac_addr configuration parameter is a 6-byte array containing the local MAC
address of CoreTSE.

**phy_address**

The phy_address parameter specifies the address of the PHY device, set in
hardware by the address pins of the PHY device.

**tx_edc_enable**

The tx_edc_enable parameter specifies whether to enable or disable error
detection and correction for Tx FIFOs. The allowed values for the tx_edc_enable
configuration parameter are:

  - TSE_ERR_DET_CORR_ENABLE
  - TSE_ERR_DET_CORR_DISABLE

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_ERR_DET_CORR_ENABLE.

**rx_edc_enable**

The rx_edc_enable parameter specifies whether to enable or disable error
detection and correction for Rx FIFOs. The allowed values for the rx_edc_enable
configuration parameter are:

  - TSE_ERR_DET_CORR_ENABLE
  - TSE_ERR_DET_CORR_DISABLE

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_ERR_DET_CORR_ENABLE.

**preamble_length**

The preamble_length parameter specifies the length of the preamble field of the
packet in bytes. The allowed values for the preamble_length configuration
parameter are:

  - TSE_PREAMLEN_DEFVAL
  - TSE_PREAMLEN_MAXVAL

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_PREAMLEN_DEFVAL.

**hugeframe_enable**

The hugeframe_enable parameter specifies whether to enable or disable huge frame
support. When enabled, it allows frames longer than the standard maximum frame
length to be transmitted and received. When disabled, the hardware limits the
length of frames at the maximum frame length. The allowed values for the
`hugeframe_enable` configuration parameter are:

  - TSE_HUGE_FRAME_ENABLE
  - TSE_HUGE_FRAME_DISABLE

`TSE_cfg_struct_def_init()` sets this configuration parameter to
SE_HUGE_FRAME_DISABLE.

**length_field_check**

The length_field_check parameter specifies whether to enable or disable the
length field check. When enabled, the CoreTSE checks the frame’s length field to
ensure it matches the actual data field length. The allowed values for the
length_field_check configuration parameter are:

  - TSE_LENGTH_FIELD_CHECK_ENABLE
  - TSE_LENGTH_FIELD_CHECK_DISABLE

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_LENGTH_FIELD_CHECK_ENABLE.

**pad_n_CRC**

The pad_n_CRC parameter specifies whether to enable or disable short frame
padding and automatic CRC insertion functionality. When enabled, the CoreTSE
pads all the short frames and appends a CRC to every frame whether or not
padding is required. When disabled, frames presented to the CoreTSE have a valid
length and contain a CRC. The allowed values for the pad_n_CRC configuration
parameter are:

  - TSE_PAD_N_CRC_ENABLE
  - TSE_PAD_N_CRC_DISABLE

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_PAD_N_CRC_ENABLE.

**append_CRC**

The append_CRC parameter specifies whether to enable or disable appending CRC.
When enabled, the CoreTSE appends a CRC to all the frames. When disabled, frames
presented to the CoreTSE, have a valid length and contain a valid CRC. The
allowed values for the append_CRC configuration parameter are:

  - TSE_CRC_ENABLE
  - TSE_CRC_DISABLE

`TSE_cfg_struct_def_init()` sets this configuration parameter to TSE_CRC_ENABLE.

**fullduplex**

The fullduplex parameter specifies whether to enable or disable full duplex.
When enabled, the CoreTSE operates in full-duplex mode. When disabled, the MAC
operates in half-duplex mode. The allowed values for the fullduplex
configuration parameter are:

  - TSE_FULLDUPLEX_ENABLE
  - TSE_FULLDUPLEX_DISABLE

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_FULLDUPLEX_ENABLE.

**loopback**

The loopback parameter specifies whether to enable or disable loopback mode.
When enabled, the CoreTSE_AHB/CoreTSE’s transmit outputs are looped back to its
receiving inputs. The allowed values for the loopback configuration parameter
are:

  - TSE_LOOPBACK_ENABLE
  - TSE_LOOPBACK_DISABLE

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_LOOPBACK_DISABLE.

**rx_flow_ctrl**

The rx_flow_ctrl parameter specifies whether to enable or disable the receiver
flow control functionality. When enabled, the CoreTSE detects and acts on PAUSE
flow control frames. When disabled, it ignores PAUSE flow control frames. The
allowed values for the rx_flow_ctrl configuration parameter are:

  - TSE_RX_FLOW_CTRL_ENABLE
  - TSE_RX_FLOW_CTRL_DISABLE

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_RX_FLOW_CTRL_ENABLE.

**tx_flow_ctrl**

The tx_flow_ctrl parameter specifies whether to enable or disable transmitter
flow control. When enabled, the transmitter sends PAUSE flow control frames when
requested by the system. When disabled, the transmitter is prevented from
sending flow control frames. The allowed values for the tx_flow_ctrl
configuration parameter are:

  - TSE_TX_FLOW_CTRL_ENABLE
  - TSE_TX_FLOW_CTRL_DISABLE

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_TX_FLOW_CTRL_ENABLE.

**min_IFG**

The min_IFG parameter specifies the minimum size of the interframe gap (IFG) to
enforce between frames (expressed in bit times). The allowed values for the
min_IFG configuration parameter are:

  - TSE_MINIFG_DEFVAL
  - TSE_MINIFG_MAXVAL

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_MINIFG_DEFVAL.

**btb_IFG**

The btb_IFG parameter specifies the Interframe gap between back-to-back packets
(expressed in bit times), used exclusively in full-duplex mode when two transmit
packets are sent back-to-back. The allowed values for the btb_IFG configuration
parameter are:

  - TSE_BTBIFG_DEFVAL
  - TSE_BTBIFG_MAXVAL

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_BTBIFG_DEFVAL.

**max_retx_tries**

The max_retx_tries parameter specifies the number of retransmission attempts
following a collision before aborting the packet due to excessive collisions.
The allowed values for the max_retx_tries configuration parameter are:

  - TSE_MAXRETX_DEFVAL
  - TSE_MAXRETX_MAXVAL

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_MAXRETX_DEFVAL.

**excessive_defer**

The excessive_defer parameter specifies whether to enable or disable the
transmission of packets that exceeded maximum collisions. When enabled, the
transmitter allows the transmission of a packet that has been excessively
deferred. When disabled, the transmitter aborts the transmission of a packet
that has been excessively deferred. The allowed values for the excessive_defer
configuration parameter are:

  - TSE_EXSS_DEFER_ENABLE
  - TSE_EXSS_DEFER_DISABLE

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_EXSS_DEFER_DISABLE.

**nobackoff**

The nobackoff parameter specifies whether to enable or disable the back-off
functionality. When enabled, the transmitter immediately re-transmits following
a collision. When disabled, the transmitter follows the binary exponential
backoff rule. The allowed values for the nobackoff configuration parameter are:

  - TSE_NO_BACKOFF_ENABLE
  - TSE_NO_BACKOFF_DISABLE

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_NO_BACKOFF_DISABLE.

**backpres_nobackoff**

The backpres_nobackoff parameter specifies whether to enable or disable back-off
in back pressure mode. When enabled, the transmitter immediately re-transmits
following a collision during an Ethernet back pressure operation. When disabled,
the transmitter follows the binary exponential backoff rule. The allowed values
for the backpres_nobackoff configuration parameter are:

  - TSE_BACKPRESS_NO_BACKOFF_ENABLE
  - TSE_BACKPRESS_NO_BACKOFF_DISABLE

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_BACKPRESS_NO_BACKOFF_DISABLE.

**ABEB_enable**

The ABEB_enable parameter specifies whether to enable or disable arbitrary
binary exponential back-off. When enabled, it configures the transmitter to use
the ABEB_truncvalue value instead of the 802.3 standard 10 collisions. When
disabled, it causes the transmitter to follow the 802.3 standard binary
exponential backoff rule. The allowed values for the ABEB_enable configuration
parameter are:

  - TSE_ABEB_ENABLE
  - TSE_ABEB_DISABLE

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_ABEB_DISABLE.

**ABEB_truncvalue**

The ABEB_truncvalue parameter specifies alternative binary exponential back-off
value. This value is used when the ABEB_enable parameter is enabled. The allowed
values for the ABEB_truncvalue configuration parameter are:

  - TSE_ABEBTRUNC_DEFVAL
  - TSE_ABEBTRUNC_MAXVAL

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_ABEBTRUNC_DEFVAL.

**phyclk**

The phyclk parameter specifies the MII management clock divider value. PCLK is
the source clock. The allowed values for the phyclk configuration parameter are:

  - TSE_DEF_PHY_CLK
  - TSE_BY4_PHY_CLK
  - TSE_BY6_PHY_CLK
  - TSE_BY8_PHY_CLK
  - TSE_BY10_PHY_CLK
  - TSE_BY14_PHY_CLK
  - TSE_BY20_PHY_CLK
  - TSE_BY28_PHY_CLK

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_DEF_PHY_CLK.

**supress_preamble**

The supress_preamble parameter specifies whether to enable or disable preamble
suppression at the PHY. When enabled, MII Management suppresses preamble
generation and reduces the Management cycle from 64 clocks to 32 clocks. When
disabled, MII Management performs Management read/write cycles with the 64
clocks of preamble. The allowed values for the supress_preamble configuration
parameter are:

  - TSE_SUPPRESS_PREAMBLE_ENABLE
  - TSE_SUPPRESS_PREAMBLE_DISABLE

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_SUPPRESS_PREAMBLE_DISABLE.

**autoscan_phys**

The autoscan_phys parameter specifies whether to enable or disable continuous
reading from a set of PHYs of contiguous address space. When enabled, it causes
MDIO management to continually read from a set of PHYs of contiguous address
space. The starting address of the PHY is specified by the content of the PHY
address field. The next PHY to be read will be PHY address + 1. The last PHY to
be queried in this read sequence will be the one residing at address 0x31, after
which the read sequence returns to the PHY specified by the PHY address field.
When disabled, MDIO management performs only one read operation at the specified
PHY address. The allowed values for the autoscan_phys configuration parameter
are:

  - TSE_PHY_AUTOSCAN_ENABLE
  - TSE_PHY_AUTOSCAN_DISABLE

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_PHY_AUTOSCAN_DISABLE.

**max_frame_length**

The max_frame_length parameter specifies the maximum frame size in both the
transmit and receive directions. The allowed values for the max_frame_length
configuration parameter are:

  - TSE_MAXFRAMELEN_DEFVAL
  - TSE_MAXFRAMELEN_MAXVAL

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_MAXFRAMELEN_DEFVAL.

**non_btb_IFG**

The non_btb_IFG parameter specifies the non-back-to-back interframe gap value.
The allowed values for the non_btb_IFG configuration parameter are:

  - TSE_NONBTBIFG_DEFVAL
  - TSE_NONBTBIFG_MAXVAL

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_NONBTBIFG_DEFVAL.

**slottime**

The slottime parameter specifies the slot time or collision window during which
collisions might occur in a properly configured network. The allowed values for
the slot time configuration parameter are:

  - TSE_SLOTTIME_DEFVAL
  - TSE_SLOTTIME_MAXVAL

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_SLOTTIME_DEFVAL.

**framedrop_mask**

The framedrop_mask parameter specifies to drop frames based on receive
statistics. The allowed values for the framedrop_mask configuration parameter
are:

  - TSE_DEFVAL_FRAMEDROP_MASK
  - TSE_SMALL_AFG_FRAMEDROP_MASK
  - TSE_EXDV_EVENT_FRAMEDROP_MASK
  - TSE_FALSE_CARRIER_FRAMEDROP_MASK
  - TSE_CODE_ERR_FRAMEDROP_MASK
  - TSE_CRC_ERR_FRAMEDROP_MASK
  - TSE_LEN_CHKERR_FRAMEDROP_MASK
  - TSE_OUTOFRANGE_LEN_FRAMEDROP_MASK
  - TSE_OK_FRAME_FRAMEDROP_MASK
  - TSE_MULTICAST_FRAMEDROP_MASK
  - TSE_BROADCAST_FRAMEDROP_MASK
  - TSE_DRIBBLE_NIBBLE_FRAMEDROP_MASK
  - TSE_CONTROL_FRAME_FRAMEDROP_MASK
  - TSE_PAUSE_FRAME_FRAMEDROP_MASK
  - TSE_UNSUPPORTED_OPCODE_FRAMEDROP_MASK
  - TSE_VLAN_TAG_FRAMEDROP_MASK
  - TSE_LONG_EVENT_FRAMEDROP_MASK
  - TSE_TRUNCKATED_FRAME_FRAMEDROP_MASK
  - TSE_UNICAST_NO_SAMATCH_FRAMEDROP_MASK
  - TSE_SHORT_FRAME_FRAMEDROP_MASK

`TSE_cfg_struct_def_init()` sets this configuration parameter to
TSE_DEFVAL_FRAMEDROP_MASK.

**wol_enable**

The wol_enable parameter specifies whether to enable or disable the Wake on LAN
(WoL) functionality. When enabled, CoreTSE can detect either the unicast match
frame or AMD magic packet frame and assert the WoL interrupt. When disabled, the
WoL interrupt is not asserted even though unicast match frame or AMD magic
packet frame is detected. The allowed values for the supress_preamble
configuration parameter are:

  - TSE_WOL_UNICAST_FRAME_DETECT_EN
  - TSE_WOL_MAGIC_FRAME_DETECT_EN
  - TSE_WOL_DETECT_DISABLE

**aneg_enable**

The aneg_enable parameter specifies whether to enable or disable the auto-
negotiation functionality in the PHY and “TBI/1000BaseX” module. When enabled,
CoreTSE enables auto-negotiation in the PHY and “TBI/1000BaseX” module. When
disabled, CoreTSE does not enable auto-negotiation in PHY and “TBI/1000BaseX”
module. Disabling auto-negotiation allows the user to use the CoreTSE in point-
to-point topology where speed/duplex configuration is known and auto-negotiation
is not required.

`TSE_cfg_struct_def_init()` sets this configuration parameter to TSE_ENABLE.


</div>


 --------------------------- 
<a name="tseinstancet"></a>
## tse_instance_t
<a name="prototype"></a>
### Prototype 

<div id="Types$tse_instance_t$prototype" data-type="code">

``` 
    typedef struct tse_instance {
        uint32_t base_addr; 
        tse_desc_t tx_desc_tab; <--Transmit descriptor table 
        tse_desc_t rx_desc_tab; <--Receive descriptor table 
        tse_transmit_callback_t tx_complete_handler; 
        tse_receive_callback_t pckt_rx_callback; 
        int16_t nb_available_tx_desc; 
        int16_t first_tx_index; 
        int16_t last_tx_index; 
        int16_t next_tx_index; 
        int16_t nb_available_rx_desc; 
        int16_t next_free_rx_desc_index; 
        int16_t first_rx_desc_index; 
        uint8_t phy_addr; <--PHY address for this instance of CoreTSE 
        tse_wol_callback_t wol_callback; 
    } tse_instance_t ;
  ``` 


</div>

<a name="description"></a>
### Description 

<div id="Types$tse_instance_t$description" data-type="text">

**CoreTSE Hardware Instance**

The CoreTSE hardware instance identifies the various CoreTSE hardware instances
in your system. Your application software must declare one instance of this
structure for each instance of CoreTSE in your system. `TSE_init()` initializes
this structure. A pointer to an initialized instance of the structure must be
passed as the first parameter to the CoreTSE driver functions, to identify which
CoreTSE hardware instances should perform the requested operation.


</div>


 --------------------------- 

# Constants

 ---------------- 
<div id="Constants$TSE_DISABLE$description" data-type="text">

<a name="tsedisable"></a>
## State definitions
| Constant  | Description   | 
| -----|-----|
| TSE_DISABLE  | Disable the CoreTSE   | 
| TSE_ENABLE  | Enable the CoreTSE   | 

</div>

<div id="Constants$TSE_SUCCESS$description" data-type="text">

<a name="tsesuccess"></a>
## Packet Transmission
The following definitions are used with functions `TSE_send_pkt()` and
`TSE_receive_pkt()` to report success or failure in assigning the packet memory
buffer to a transmit/receive descriptor.

| Constant | Description | | --------

</div>

<div id="Constants$TSE_LINK_DOWN$description" data-type="text">

<a name="tselinkdown"></a>
## Link Status
The following definitions are used with `TSE_get_link_status()` to report the
link status.

| Constant  | Description   | 
| -----|-----|
| TSE_LINK_DOWN  | Ethernet link is down. There is no connection to a remote device.   | 
| TSE_LINK_UP  | Ethernet link is up. A connection is established with a remote device.   | 
| TSE_HALF_DUPLEX  | Connection is half-duplex.   | 
| TSE_FULL_DUPLEX  | Connection is full-duplex.   | 

</div>

<div id="Constants$TSE_MAX_PACKET_SIZE$description" data-type="text">

<a name="tsemaxpacketsize"></a>
## Maximum MAC frame size (packet size)
| Constant  | Description   | 
| -----|-----|
| TSE_MAX_PACKET_SIZE  | Maximum packet size of a standard Ethernet frame.   | 
| TSE_JUMBO_PACKET_SIZE  | Maximum size of Jumbo frame packet.   | 

</div>

<div id="Constants$TSE_MAX_TX_BUF_SIZE$description" data-type="text">

<a name="tsemaxtxbufsize"></a>
## Transmit and receive packet buffer sizes
| Constant  | Description   | 
| -----|-----|
| TSE_MAX_TX_BUF_SIZE  | Maximum transmit buffer size.   | 
| TSE_MAX_RX_BUF_SIZE  | Maximum receive buffer size.   | 

</div>

<div id="Constants$TSE_ERR_DET_CORR_ENABLE$description" data-type="text">

<a name="tseerrdetcorrenable"></a>
## Configuration Parameter Definitions
<a name="fifo-error-detection-&-correction"></a>
### FIFO error detection & correction
| Constant  | Description   | 
| -----|-----|
| TSE_ERR_DET_CORR_ENABLE  | Enable FIFO error detection and correction.   | 
| TSE_ERR_DET_CORR_DISABLE  | Disable FIFO error detection and correction.   | 

</div>

<div id="Constants$TSE_HUGE_FRAME_ENABLE$description" data-type="text">

<a name="tsehugeframeenable"></a>
<a name="huge-frame-support"></a>
### Huge frame support
| Constant  | Description   | 
| -----|-----|
| TSE_HUGE_FRAME_ENABLE  | Enable Huge frame support.   | 
| TSE_HUGE_FRAME_DISABLE  | Disable Huge frame support.   | 

</div>

<div id="Constants$TSE_LENGTH_FIELD_CHECK_ENABLE$description" data-type="text">

<a name="tselengthfieldcheckenable"></a>
<a name="length-field-checking"></a>
### Length field checking
| Constant  | Description   | 
| -----|-----|
| TSE_LENGTH_FIELD_CHECK_ENABLE  | Enable length field checking.   | 
| TSE_LENGTH_FIELD_CHECK_DISABLE  | Disable length field checking.   | 

</div>

<div id="Constants$TSE_PAD_N_CRC_ENABLE$description" data-type="text">

<a name="tsepadncrcenable"></a>
<a name="padding-and-crc"></a>
### Padding and CRC
| Constant  | Description   | 
| -----|-----|
| TSE_PAD_N_CRC_ENABLE  | Enable padding and CRC.   | 
| TSE_PAD_N_CRC_DISABLE  | Disable padding and CRC.   | 

</div>

<div id="Constants$TSE_CRC_ENABLE$description" data-type="text">

<a name="tsecrcenable"></a>
<a name="append-crc"></a>
### Append CRC
| Constant  | Description   | 
| -----|-----|
| TSE_CRC_ENABLE  | Enable CRC appending.   | 
| TSE_CRC_DISABLE  | Disable CRC appending.   | 

</div>

<div id="Constants$TSE_FULLDUPLEX_ENABLE$description" data-type="text">

<a name="tsefullduplexenable"></a>
<a name="full-duplex-mode"></a>
### Full-duplex mode
| Constant  | Description   | 
| -----|-----|
| TSE_FULLDUPLEX_ENABLE  | Enable full-duplex mode.   | 
| TSE_FULLDUPLEX_DISABLE  | Disable full-duplex mode.   | 

</div>

<div id="Constants$TSE_LOOPBACK_ENABLE$description" data-type="text">

<a name="tseloopbackenable"></a>
<a name="loopback-mode"></a>
### Loopback mode
| Constant  | Description   | 
| -----|-----|
| TSE_LOOPBACK_ENABLE  | Enable loopback mode.   | 
| TSE_LOOPBACK_DISABLE  | Disable loopback mode.   | 

</div>

<div id="Constants$TSE_RX_FLOW_CTRL_ENABLE$description" data-type="text">

<a name="tserxflowctrlenable"></a>
<a name="receiver-flow-control"></a>
### Receiver flow control
| Constant  | Description   | 
| -----|-----|
| TSE_RX_FLOW_CTRL_ENABLE  | Enable receiver flow control.   | 
| TSE_RX_FLOW_CTRL_DISABLE  | Disable receiver flow control.   | 

</div>

<div id="Constants$TSE_TX_FLOW_CTRL_ENABLE$description" data-type="text">

<a name="tsetxflowctrlenable"></a>
<a name="transmission-flow-control"></a>
### Transmission flow control
| Constant  | Description   | 
| -----|-----|
| TSE_TX_FLOW_CTRL_ENABLE  | Enable Transmission flow control.   | 
| TSE_TX_FLOW_CTRL_DISABLE  | Disable Transmission flow control.   | 

</div>

<div id="Constants$TSE_EXSS_DEFER_ENABLE$description" data-type="text">

<a name="tseexssdeferenable"></a>
<a name="excessive-defer"></a>
### Excessive defer
| Constant  | Description   | 
| -----|-----|
| TSE_EXSS_DEFER_ENABLE  | Enable excessive defer.   | 
| TSE_EXSS_DEFER_DISABLE  | Disable excessive defer.   | 

</div>

<div id="Constants$TSE_NO_BACKOFF_ENABLE$description" data-type="text">

<a name="tsenobackoffenable"></a>
<a name="no-backoff"></a>
### No-backoff
| Constant  | Description   | 
| -----|-----|
| TSE_NO_BACKOFF_ENABLE  | Enable no-backoff.   | 
| TSE_NO_BACKOFF_DISABLE  | Disable no-backoff.   | 

</div>

<div id="Constants$TSE_BACKPRESS_NO_BACKOFF_ENABLE$description" data-type="text">

<a name="tsebackpressnobackoffenable"></a>
<a name="backpressure-no-backoff"></a>
### Backpressure no-backoff
| Constant  | Description   | 
| -----|-----|
| TSE_BACKPRESS_NO_BACKOFF_ENABLE  | Enable backpressure no-backoff.   | 
| TSE_BACKPRESS_NO_BACKOFF_DISABLE  | Disable backpressure no-backoff.   | 

</div>

<div id="Constants$TSE_ABEB_ENABLE$description" data-type="text">

<a name="tseabebenable"></a>
<a name="alternative-binary-exponential-backoff"></a>
### Alternative binary exponential backoff
| Constant  | Description   | 
| -----|-----|
| TSE_ABEB_ENABLE  | Enable alternative binary exponential backoff.   | 
| TSE_ABEB_DISABLE  | Disable alternative binary exponential backoff.   | 

</div>

<div id="Constants$TSE_SUPPRESS_PREAMBLE_ENABLE$description" data-type="text">

<a name="tsesuppresspreambleenable"></a>
<a name="suppress-preamble"></a>
### Suppress preamble
| Constant  | Description   | 
| -----|-----|
| TSE_SUPPRESS_PREAMBLE_ENABLE  | Enable preamble suppression.   | 
| TSE_SUPPRESS_PREAMBLE_DISABLE  | Disable preamble suppression.   | 

</div>

<div id="Constants$TSE_PHY_AUTOSCAN_ENABLE$description" data-type="text">

<a name="tsephyautoscanenable"></a>
<a name="auto-scan-phys"></a>
### Auto-scan PHYs
| Constant  | Description   | 
| -----|-----|
| TSE_PHY_AUTOSCAN_ENABLE  | Enable PHYs auto-scan.   | 
| TSE_PHY_AUTOSCAN_DISABLE  | Disable PHYs auto-scan.   | 

</div>

<div id="Constants$TSE_WOL_UNICAST_FRAME_DETECT_EN$description" data-type="text">

<a name="tsewolunicastframedetecten"></a>
<a name="wol-event-detection-masks"></a>
### WoL event detection masks
| Constant  | Description   | 
| -----|-----|
| TSE_WOL_UNICAST_FRAME_DETECT_EN  | WoL with unicast match frame detection.   | 
| TSE_WOL_MAGIC_FRAME_DETECT_EN  | WoL with AMD magic packet detection.   | 
| TSE_WOL_DETECT_DISABLE  | WoL detection disabled.   | 

</div>

<div id="Constants$TSE_PREAMLEN_DEFVAL$description" data-type="text">

<a name="tsepreamlendefval"></a>
<a name="preamble-length-default-value-and-maximum-value"></a>
### Preamble length default value and maximum value
| Constant  | Description   | 
| -----|-----|
| TSE_PREAMLEN_DEFVAL  | Default preamble length.   | 
| TSE_PREAMLEN_MAXVAL  | Maximum preamble length.   | 

</div>

<div id="Constants$TSE_NIBBLE_MODE$description" data-type="text">

<a name="tsenibblemode"></a>
<a name="bytenibble-mode"></a>
### Byte/Nibble mode
| Constant  | Description   | 
| -----|-----|
| TSE_NIBBLE_MODE  | Nibble mode.   | 
| TSE_BYTE_MODE  | Byte mode.   | 

</div>

<div id="Constants$TSE_MINIFG_MAXVAL$description" data-type="text">

<a name="tseminifgmaxval"></a>
## IPG/IFG values
| Constant  | Description   | 
| -----|-----|
| TSE_MINIFG_MAXVAL  | Minimum interframe gap - maximum value.   | 
| TSE_MINIFG_DEFVAL  | Minimum interframe gap - default value.   | 
| TSE_BTBIFG_MAXVAL  | Back-to-back interframe gap - maximum value   | 
| TSE_BTBIFG_DEFVAL  | Back-to-back interframe gap - default value.   | 
| TSE_NONBTBIFG_DEFVAL  | Non-back-to-back interframe gap - default value.   | 
| TSE_NONBTBIFG_MAXVAL  | Non-back-to-back interframe gap - maximum value.   | 

</div>

<div id="Constants$TSE_MAXRETX_MAXVAL$description" data-type="text">

<a name="tsemaxretxmaxval"></a>
## Number of maximum retransmission tries
| Constant  | Description   | 
| -----|-----|
| TSE_MAXRETX_MAXVAL  | Maximum number of retransmission attempts - maximum value.   | 
| TSE_MAXRETX_DEFVAL  | Maximum number of retransmission attempts - default value.   | 
| TSE_ABEBTRUNC_MAXVAL  | Alternate Binary Exponential Backoff Truncation - maximum value.   | 
| TSE_ABEBTRUNC_DEFVAL  | Alternate Binary Exponential Backoff Truncation - default value.   | 

</div>

<div id="Constants$TSE_DEF_PHY_CLK$description" data-type="text">

<a name="tsedefphyclk"></a>
## PHY clock divider values
| Constant  | Description   | 
| -----|-----|
| TSE_DEF_PHY_CLK  | Default source clock.   | 
| TSE_BY4_PHY_CLK  | Source clock divided by 4.   | 
| TSE_BY6_PHY_CLK  | Source clock divided by 6.   | 
| TSE_BY8_PHY_CLK  | Source clock divided by 8.   | 
| TSE_BY10_PHY_CLK  | Source clock divided by 10.   | 
| TSE_BY14_PHY_CLK  | Source clock divided by 14.   | 
| TSE_BY20_PHY_CLK  | Source clock divided by 20.   | 
| TSE_BY28_PHY_CLK  | Source clock divided by 28.   | 

</div>

<div id="Constants$TSE_DEFAULT_PHY$description" data-type="text">

<a name="tsedefaultphy"></a>
## PHY addresses
| Constant  | Description   | 
| -----|-----|
| TSE_DEFAULT_PHY  | Default PHY address.   | 
| TSE_PHYADDR_MAXVAL  | PHY address - maximum value.   | 
| TSE_PHYREGADDR_MAXVAL  | PHY register address - maximum value.   | 
| TSE_PHY_ADDR_DEFVAL  | PHY address - default value.   | 
| TSE_PHYREGADDR_DEFVAL  | PHY register address - default value.   | 

</div>

<div id="Constants$TSE_MAXFRAMELEN_DEFVAL$description" data-type="text">

<a name="tsemaxframelendefval"></a>
## Maximum frame length
| Constant  | Description   | 
| -----|-----|
| TSE_MAXFRAMELEN_DEFVAL  | Maximum frame length - default value.   | 
| TSE_MAXFRAMELEN_MAXVAL  | Maximum frame length - maximum value.   | 

</div>

<div id="Constants$TSE_SLOTTIME_DEFVAL$description" data-type="text">

<a name="tseslottimedefval"></a>
## Slot Time (Collision window)
| Constant  | Description   | 
| -----|-----|
| TSE_SLOTTIME_DEFVAL  | Slot time (Collision window) default value.   | 
| TSE_SLOTTIME_MAXVAL  | Slot time (Collision window) maximum value.   | 

</div>

<div id="Constants$TSE_ANEG_ENABLE$description" data-type="text">

<a name="tseanegenable"></a>
## Auto-negotiation options
| Constant  | Description   | 
| -----|-----|
| TSE_ANEG_ENABLE  | Enable auto-negotiation.   | 
| TSE_ANEG_DISABLE  | Disable auto-negotiation.   | 

</div>

<div id="Constants$TSE_ANEG_10M_FD$description" data-type="text">

<a name="tseaneg10mfd"></a>
## Speed configuration options
| Constant  | Description   | 
| -----|-----|
| TSE_ANEG_10M_FD  | Auto-negotiate at 10 Mbps full-duplex.   | 
| TSE_ANEG_10M_HD  | Auto-negotiate at 10 Mbps half-duplex.   | 
| TSE_ANEG_100M_FD  | Auto-negotiate at 100 Mbps full-duplex.   | 
| TSE_ANEG_100M_HD  | Auto-negotiate at 100 Mbps half-duplex.   | 
| TSE_ANEG_1000M_FD  | Auto-negotiate at 1000 Mbps full-duplex.   | 
| TSE_ANEG_ALL_SPEEDS  | Auto-negotiate at all speeds.   | 

</div>

<div id="Constants$TSE_FC_PASS_BROADCAST_MASK$description" data-type="text">

<a name="tsefcpassbroadcastmask"></a>
## Frame Filter configuration options
| Constant | Description | | -----------------------------

</div>

<div id="Constants$TSE_DEFVAL_FRAMEDROP_MASK$description" data-type="text">

<a name="tsedefvalframedropmask"></a>
## Default framedrop mask
| Constant  | Description   | 
| -----|-----|
| TSE_DEFVAL_FRAMEDROP_MASK  | Generic default value applicable to most applications.   | 

</div>

<div id="Constants$TSE_SMALL_AFG_FRAMEDROP_MASK$description" data-type="text">

<a name="tsesmallafgframedropmask"></a>
<a name="framedrop-if-ifg-is-small"></a>
### Framedrop if IFG is small
| Constant  | Description   | 
| -----|-----|
| TSE_SMALL_AFG_FRAMEDROP_MASK  | Drop the packet if IFG is small.   | 

</div>

<div id="Constants$TSE_EXDV_EVENT_FRAMEDROP_MASK$description" data-type="text">

<a name="tseexdveventframedropmask"></a>
<a name="framedrop-if-short-rxdv-event"></a>
### Framedrop if short RXDV Event
| Constant  | Description   | 
| -----|-----|
| TSE_EXDV_EVENT_FRAMEDROP_MASK  | Drop the packet if a short RXDV event is received.   | 

Short RXDV event: Indicates that the last received event seen was not long
enough to be a valid packet.

</div>

<div id="Constants$TSE_FALSE_CARRIER_FRAMEDROP_MASK$description" data-type="text">

<a name="tsefalsecarrierframedropmask"></a>
<a name="framedrop-if-false-carrier"></a>
### Framedrop if False Carrier
| Constant  | Description   | 
| -----|-----|
| TSE_FALSE_CARRIER_FRAMEDROP_MASK  | Drop the packet if a False Carrier is received.   | 

#### False Carrier
Indicates that at some time since the last receive statistics vector, a false
carrier was detected, noted, and reported with the next receive statistics. The
false carrier is not associated with this packet. False carrier is an activity
on the receive channel that does not result in a packet receive attempt being
made.

Defined to be RXER = 1, RXDV = 0, RXD[3:0] = 0xE, RXD[7:0] = 0x0E.

</div>

<div id="Constants$TSE_CODE_ERR_FRAMEDROP_MASK$description" data-type="text">

<a name="tsecodeerrframedropmask"></a>
<a name="framedrop-if-code-error"></a>
### Framedrop if Code error
| Constant  | Description   | 
| -----|-----|
| TSE_CODE_ERR_FRAMEDROP_MASK  | Drop the packet if a code error is received.   | 

Code Error: One or more nibbles were signalled as errors during the reception of
the packet.

</div>

<div id="Constants$TSE_CRC_ERR_FRAMEDROP_MASK$description" data-type="text">

<a name="tsecrcerrframedropmask"></a>
<a name="framedrop-if-crc-error"></a>
### Framedrop if CRC Error
| Constant  | Description   | 
| -----|-----|
| TSE_CRC_ERR_FRAMEDROP_MASK  | Drop the packet if a CRC Error is received.   | 

CRC Error: The packet's CRC did not match the internally generated CRC.

</div>

<div id="Constants$TSE_LEN_CHKERR_FRAMEDROP_MASK$description" data-type="text">

<a name="tselenchkerrframedropmask"></a>
<a name="framedrop-if-length-check-error"></a>
### Framedrop if Length check error
| Constant  | Description   | 
| -----|-----|
| TSE_LEN_CHKERR_FRAMEDROP_MASK  | Drop the frame if a length check error is received.   | 

#### Length Check Error
Indicates that the frame length field value in the packet does not match the
actual data byte length and is not a type field.

</div>

<div id="Constants$TSE_OUTOFRANGE_LEN_FRAMEDROP_MASK$description" data-type="text">

<a name="tseoutofrangelenframedropmask"></a>
<a name="framedrop-if-length-out-of-range"></a>
### Framedrop if Length Out of Range
| Constant | Description | | ------------------------------

#### Length Out of Range
Indicates that the frames length was larger than 1518 bytes but smaller than the
hosts maximum frame length value (type field).

</div>

<div id="Constants$TSE_OK_FRAME_FRAMEDROP_MASK$description" data-type="text">

<a name="tseokframeframedropmask"></a>
<a name="framedrop-if-an-ok-frame"></a>
### Framedrop if an OK frame
| Constant  | Description   | 
| -----|-----|
| TSE_OK_FRAME_FRAMEDROP_MASK  | Drop the frame if an OK frame is received.   | 

#### OK frame
The frame contained a valid CRC and did not have a code error.

</div>

<div id="Constants$TSE_MULTICAST_FRAMEDROP_MASK$description" data-type="text">

<a name="tsemulticastframedropmask"></a>
<a name="framedrop-if-multicast-frame"></a>
### Framedrop if Multicast frame
| Constant  | Description   | 
| -----|-----|
| TSE_MULTICAST_FRAMEDROP_MASK  | Drop the frame if a Multicast frame is received.   | 

#### Multicast frame
The packet's destination address contained a multicast address.

</div>

<div id="Constants$TSE_BROADCAST_FRAMEDROP_MASK$description" data-type="text">

<a name="tsebroadcastframedropmask"></a>
<a name="framedrop-if-broadcast-frame"></a>
### Framedrop if Broadcast frame
| Constant  | Description   | 
| -----|-----|
| TSE_BROADCAST_FRAMEDROP_MASK  | Drop the frame if a Broadcast frame is received.   | 

#### Broadcast frame
The packet's destination address contained the broadcast address.

</div>

<div id="Constants$TSE_DRIBBLE_NIBBLE_FRAMEDROP_MASK$description" data-type="text">

<a name="tsedribblenibbleframedropmask"></a>
<a name="framedrop-if-dribble-nibble"></a>
### Framedrop if Dribble Nibble
| Constant  | Description   | 
| -----|-----|
| TSE_DRIBBLE_NIBBLE_FRAMEDROP_MASK  | Drop the frame if a Dribble Nibble is received.   | 

#### Dribble Nibble
Indicates that after the end of the packet, an additional 1 to 7 bits were
received. A single nibble, called the dribble nibble, is formed but not sent to
the system (10/100 Mbps only).

</div>

<div id="Constants$TSE_CONTROL_FRAME_FRAMEDROP_MASK$description" data-type="text">

<a name="tsecontrolframeframedropmask"></a>
<a name="framedrop-if-control-frame"></a>
### Framedrop if Control Frame
| Constant  | Description   | 
| -----|-----|
| TSE_CONTROL_FRAME_FRAMEDROP_MASK  | Drop the frame if a Control Frame is received.   | 

#### Control Frame
The current frame was recognized as a Control frame for having a valid Type-
Length designation.

</div>

<div id="Constants$TSE_PAUSE_FRAME_FRAMEDROP_MASK$description" data-type="text">

<a name="tsepauseframeframedropmask"></a>
<a name="framedrop-if-pause-control-frame"></a>
### Framedrop if PAUSE Control Frame
| Constant  | Description   | 
| -----|-----|
| TSE_PAUSE_FRAME_FRAMEDROP_MASK  | Drop the frame if a PAUSE Control Frame is received.   | 

#### PAUSE Control frame
Current frame was recognized as a Control frame containing a valid PAUSE Frame
Op-code and a valid address.

</div>

<div id="Constants$TSE_UNSUPPORTED_OPCODE_FRAMEDROP_MASK$description" data-type="text">

<a name="tseunsupportedopcodeframedropmask"></a>
<a name="framedrop-if-unsupported-op-code"></a>
### Framedrop if Unsupported Op-code
| Constant  | Description   | 
| -----|-----|
| TSE_UNSUPPORTED_OPCODE_FRAMEDROP_MASK  | Drop the frame if an Unsupported Op-code is received.   | 

#### Unsupported Op-code
The current frame was recognized as a Control frame by the PEMCS, but it
contains an unknown Op-code.

</div>

<div id="Constants$TSE_VLAN_TAG_FRAMEDROP_MASK$description" data-type="text">

<a name="tsevlantagframedropmask"></a>
<a name="framedrop-if-vlan-tag"></a>
### Framedrop if VLAN Tag
| Constant  | Description   | 
| -----|-----|
| TSE_VLAN_TAG_FRAMEDROP_MASK  | Drop the frame if VLAN Tag is detected.   | 

#### VLAN Tag: Received frames length/type field that contains 0x8100, which is the VLAN Protocol
Identifier.

</div>

<div id="Constants$TSE_LONG_EVENT_FRAMEDROP_MASK$description" data-type="text">

<a name="tselongeventframedropmask"></a>
<a name="framedrop-if-a-long-event"></a>
### Framedrop if a Long Event
| Constant  | Description   | 
| -----|-----|
| TSE_LONG_EVENT_FRAMEDROP_MASK  | Drop the frame if a Long Event is received.   | 

</div>

<div id="Constants$TSE_TRUNCKATED_FRAME_FRAMEDROP_MASK$description" data-type="text">

<a name="tsetrunckatedframeframedropmask"></a>
<a name="framedrop-if-a-truncated-frame"></a>
### Framedrop if a Truncated frame
| Constant  | Description   | 
| -----|-----|
| TSE_LONG_EVENT_FRAMEDROP_MASK  | Drop the frame if a Long Event is received.   | 

</div>

<div id="Constants$TSE_UNICAST_NO_SAMATCH_FRAMEDROP_MASK$description" data-type="text">

<a name="tseunicastnosamatchframedropmask"></a>
<a name="framedrop-if-unicast-frame-detected"></a>
### Framedrop if Unicast frame detected
| Constant | Description | | ----------------------------------

</div>

<div id="Constants$TSE_SHORT_FRAME_FRAMEDROP_MASK$description" data-type="text">

<a name="tseshortframeframedropmask"></a>
<a name="framedrop-if-a-short-frame-received"></a>
### Framedrop if a short frame received
| Constant | Description | | ---------------------------

</div>

<div id="Constants$TSE_PHY_ADDRESS_AUTO_DETECT$description" data-type="text">

<a name="tsephyaddressautodetect"></a>
## PHY Address
The application can use this constant to indicate to the driver to probe the PHY
and auto-detect the PHY address that it is configured with. If you already know
the PHY address configured in your hardware system, you can provide that address
to the driver instead of this constant. That way the `TSE_init()` function would
be faster because the auto-detection process of the PHY address is now avoided.

Note: To auto-detect the PHY address, the driver scans the valid MDIO addresses
starting from -0- for valid data. If CoreRGMII and PHY are connected to the
MAC's management interface, then you must make sure that the PHY device MDIO
address is less than the CoreRGMII MDIO address.

| Constant  | Value   | 
| -----|-----|
| TSE_PHY_ADDRESS_AUTO_DETECT  | 255   | 

</div>

<div id="Constants$TSE_FIFO_TX_CTRL_FRAME$description" data-type="text">

<a name="tsefifotxctrlframe"></a>
## Per Packet Override Masks
A 5-bit field containing the per-packet override flags signalled to the FIFO
during packet transmission. The bits are encoded as follows:

| Constant  | Value   | 
| -----|-----|
| TSE_FIFO_TX_CTRL_FRAME  | FIFO Transmit Control Frame   | 
| TSE_FIFO_TX_NO_CTRL_FRAME  | Not FIFO Transmit Control Frame   | 
| TSE_FIFO_TX_PERPKT_PAD_FCS  | FIFO Transmit Per-Packet PAD Mode   | 
| TSE_FIFO_TX_PERPKT_NO_PAD_FCS  | Not FIFO Transmit Per-Packet PAD Mode   | 
| TSE_FIFO_TX_PERPKT_FCS  | FIFO Transmit Per-Packet Generate FCS   | 
| TSE_FIFO_TX_PERPKT_NO_FCS  | Not FIFO Transmit Per-Packet Generate FCS   | 
| TSE_FIFO_TX_PERPKT_ENABLE  | FIFO Transmit Per-Packet Generate FCS   | 
| TSE_FIFO_TX_PERPKT_DISABLE  | Not FIFO Transmit Per-Packet Generate FCS   | 

</div>


# Functions

 ---------------- 
<a name="tsecfgstructdefinit"></a>
## TSE_cfg_struct_def_init
<a name="prototype"></a>
### Prototype 

<div id="Functions$TSE_cfg_struct_def_init$prototype" data-type="code">

    void
    TSE_cfg_struct_def_init
    (
        tse_cfg_t * cfg
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$TSE_cfg_struct_def_init$description" data-type="text">

`TSE_cfg_struct_def_init()` initializes a tse_cfg_t configuration data structure
to default values. The default configuration uses the MII interface connected to
a PHY at address 0x00 which is set to auto-negotiate at all available speeds up
to 1000Mbps. This default configuration can then be used as a parameter to
`TSE_init()`. Typically the default configuration would be modified to suit the
application before being passed to `TSE_init()`.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### cfg
<div id="Functions$TSE_cfg_struct_def_init$description$parameters$cfg" data-type="text" data-name="cfg">

The cfg parameter is a pointer to a tse_cfg_t data structure used as a parameter
to function `TSE_init()`.


</div>

<a name="return"></a>
### Return
<div id="Functions$TSE_cfg_struct_def_init$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$TSE_cfg_struct_def_init$description$example$Example1" data-type="text" data-name="Example">
The example below demonstrates the use of the `TSE_cfg_struct_def_init()`
function. It retrieves the default MAC configuration and modifies it to connect
through an MII Ethernet PHY at MII management interface address 0x01.

This example also demonstrates how to assign the device's MAC address and force
a 100Mbps full duplex link.


</div>

<div id="Functions$TSE_cfg_struct_def_init$description$example$Example1" data-type="code" data-name="Example">


```
    #define TSEMAC_BASE  0x30000000
    tse_cfg_t g_tse_config;
    tse_instance_t g_tse;

    TSE_cfg_struct_def_init(&g_tse_config);

    g_tse_config.phy_addr = TSE_PHY_ADDRESS_AUTO_DETECT;
    g_tse_config.speed_duplex_select = TSE_ANEG_100M_FD;
    g_tse_config.mac_addr[0] = 0xC0;
    g_tse_config.mac_addr[1] = 0xB1;
    g_tse_config.mac_addr[2] = 0x3C;
    g_tse_config.mac_addr[3] = 0x60;
    g_tse_config.mac_addr[4] = 0x60;
    g_tse_config.mac_addr[5] = 0x60;

    TSE_init(&g_tse, TSEMAC_BASE, &g_tse_config);
```

</div>


 -------------------------------- 
<a name="tseinit"></a>
## TSE_init
<a name="prototype"></a>
### Prototype 

<div id="Functions$TSE_init$prototype" data-type="code">

    void
    TSE_init
    (
        tse_instance_t * this_tse,
        uint32_t base_addr,
        tse_cfg_t * cfg
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$TSE_init$description" data-type="text">

`TSE_init()` initializes the CoreTSE hardware and driver internal data
structures. This configuration data structure contains all the information
required to configure the CoreTSE. The `TSE_init()` function initializes the
descriptor rings and their pointers to initial values. `TSE_init()` enables DMA
Rx packet received and Tx packet sent interrupts.

The configuration passed to the `TSE_init()` function specifies the type of
interface used to connect the CoreTSE and Ethernet PHY as well as the PHY MII
management interface address. The application may pass the
TSE_PHY_ADDRESS_AUTO_DETECT constant as a configuration parameter to indicate to
the driver to probe for the PHY address. If you know the hardware configuration
PHY address of the onboard PHY in your system, you may choose to provide that
instead. When the user supplies a PHY address value other than
TSE_PHY_ADDRESS_AUTO_DETECT, the driver will use the supplied value directly,
provided it is within the valid PHY address range (0u - 31u).

The `TSE_init()` function also specifies the allowed link speeds and duplex
modes. It is at this point that the application chooses to auto-negotiate the
link speeds and duplex modes with the link partner or force a specific speed and
duplex mode.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_tse
<div id="Functions$TSE_init$description$parameters$this_tse" data-type="text" data-name="this_tse">

The this_tse parameter is a pointer to the tse_instance_t data structure
instance holding all data regarding the CoreTSE hardware instance being
initialized. A pointer to the same data structure must be provided in subsequent
calls to the various CoreTSE driver functions to identify the CoreTSE instance
that should perform the operation implemented by the called driver function.


</div>

#### base_addr
<div id="Functions$TSE_init$description$parameters$base_addr" data-type="text" data-name="base_addr">

The base_addr parameter is the base address in the processor's memory map for
the registers of the CoreTSE instance being initialized.


</div>

#### cfg
<div id="Functions$TSE_init$description$parameters$cfg" data-type="text" data-name="cfg">

The cfg parameter is a pointer to a data structure of type tse_cfg_t containing
the CoreTSE's requested configuration. You must initialize this data structure
by first calling the `TSE_cfg_struct_def_init()` function to fill the
configuration data structure with default values. You can then overwrite some of
the default settings with the ones specific to your application before passing
this data structure as a parameter to the `TSE_init()` function.


</div>

<a name="return"></a>
### Return
<div id="Functions$TSE_init$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$TSE_init$description$example$Example1" data-type="text" data-name="Example">
This example demonstrates the use of the `TSE_init()` function to configure the
CoreTSE with the default configuration.

Note: A unique MAC address must always be assigned through the configuration
data passed as a parameter to the `TSE_init()` function.


</div>

<div id="Functions$TSE_init$description$example$Example1" data-type="code" data-name="Example">


```
    #define TSEMAC_BASE  0x30000000
    tse_cfg_t g_tse_config;
    tse_instance_t g_tse;

    TSE_cfg_struct_def_init(&g_tse_config);

    g_tse_config.phy_addr = TSE_PHY_ADDRESS_AUTO_DETECT;
    g_tse_config.speed_duplex_select = TSE_ANEG_100M_FD;
    g_tse_config.mac_addr[0] = 0xC0;
    g_tse_config.mac_addr[1] = 0xB1;
    g_tse_config.mac_addr[2] = 0x3C;
    g_tse_config.mac_addr[3] = 0x60;
    g_tse_config.mac_addr[4] = 0x60;
    g_tse_config.mac_addr[5] = 0x60;

    TSE_init(&g_tse, TSEMAC_BASE, &g_tse_config);
```

</div>


 -------------------------------- 
<a name="tsesettxcallback"></a>
## TSE_set_tx_callback
<a name="prototype"></a>
### Prototype 

<div id="Functions$TSE_set_tx_callback$prototype" data-type="code">

    void
    TSE_set_tx_callback
    (
        tse_instance_t * this_tse,
        tse_transmit_callback_t tx_complete_handler
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$TSE_set_tx_callback$description" data-type="text">

`TSE_set_tx_callback()` registers the function that the CoreTSE driver calls
when a packet is sent.

Note: This function is applicable only for CoreTSE_AHB.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_tse
<div id="Functions$TSE_set_tx_callback$description$parameters$this_tse" data-type="text" data-name="this_tse">

The this_tse parameter is a pointer to the tse_instance_t structure holding all
data regarding the CoreTSE instance controlled through this function call.


</div>

#### tx_complete_handler
<div id="Functions$TSE_set_tx_callback$description$parameters$tx_complete_handler" data-type="text" data-name="tx_complete_handler">

The tx_complete_handler parameter is a pointer to the call-back function that is
called when the CoreTSE sends a packet.


</div>

<a name="return"></a>
### Return
<div id="Functions$TSE_set_tx_callback$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$TSE_set_tx_callback$description$example$Example1" data-type="text" data-name="Example">
This example demonstrates the use of the `TSE_set_tx_callback()` function. The
application registers the `tx_complete_callback()` transmit completion callback
function with the CoreTSE driver by a call to `TSE_set_tx_callback()`. The
application dynamically allocates memory for an application-defined packet_t
data structure, builds a packet, and calls send_packet(). `send_packet()`
extracts the pointer to the buffer containing the data to transmit and its
length from the tx_packet data structure and passes these to `TSE_send_pkt()`.
It also passes the pointer to tx_packet as the p_user_data parameter. The
CoreTSE driver calls `tx_complete_callback()` once the packet is sent. The
`tx_complete_callback()` function uses the p_user_data, which points to
tx_packet, to release memory allocated by the application to store the transmit
packet.


</div>

<div id="Functions$TSE_set_tx_callback$description$example$Example1" data-type="code" data-name="Example">


```
    void tx_complete_callback(void * p_user_data);
    tse_instance_t g_tse;

    void init(void)
    {
        TSE_set_tx_callback((&g_tse, tx_complete_callback);
    }

    void tx_complete_callback (void * p_user_data)
    {
        release_packet_memory(p_user_data);
    }

    void send_packet(packet_t * tx_packet)
    {
        TSE_send_pkt(&g_tse, tx_packet->buffer, tx_packet->length, tx_packet);
    }
```

</div>


 -------------------------------- 
<a name="tsesetrxcallback"></a>
## TSE_set_rx_callback
<a name="prototype"></a>
### Prototype 

<div id="Functions$TSE_set_rx_callback$prototype" data-type="code">

    void
    TSE_set_rx_callback
    (
        tse_instance_t * this_tse,
        tse_receive_callback_t rx_callback
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$TSE_set_rx_callback$description" data-type="text">

`TSE_set_rx_callback()` registers the function the CoreTSE driver calls when a
packet is received.

Note: This function is applicable only for CoreTSE_AHB.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_tse
<div id="Functions$TSE_set_rx_callback$description$parameters$this_tse" data-type="text" data-name="this_tse">

The this_tse parameter is a pointer to the tse_instance_t structure holding all
data regarding the CoreTSE instance controlled through this function call.


</div>

#### rx_callback
<div id="Functions$TSE_set_rx_callback$description$parameters$rx_callback" data-type="text" data-name="rx_callback">

The rx_callback parameter is a pointer to the function that is called when the
packet is received by CoreTSE.


</div>

<a name="return"></a>
### Return
<div id="" data-type="text">


</div>

##### Example
<div id="Functions$TSE_set_rx_callback$description$example$Example1" data-type="text" data-name="Example">
The example below demonstrates the use of the `TSE_set_rx_callback()` function.
The `init()` function calls the `TSE_set_rx_callback()` function to register the
`rx_callback()` receive callback function with the CoreTSE driver. The
`TSE_receive_pkt()` function is then called to assign the rx_buffer to an
CoreTSE_AHB descriptor for packet reception. Once a packet is received into the
rx_buffer, the CoreTSE driver calls the rx_callback function. The
`rx_callback()` function calls the `process_rx_packet()` application function to
process the received packet then calls `TSE_receive_pkt()` to reallocate
rx_buffer to receive another packet. Every time a packet is received the
`rx_callback()` function is called to process the received packet and reallocate
rx_buffer for packet reception.


</div>

<div id="Functions$TSE_set_rx_callback$description$example$Example1" data-type="code" data-name="Example">


```
    uint8_t rx_buffer[TSE_MAX_RX_BUF_SIZE];

    void rx_callback
    (
        uint8_t * p_rx_packet,
        uint32_t pckt_length,
        void * p_user_data
    )
    {
        process_rx_packet(p_rx_packet, pckt_length);
        TSE_receive_pkt(rx_buffer, (void *)0);
    }

    void init(void)
    {
        TSE_set_rx_callback(rx_callback);
        TSE_receive_pkt(rx_buffer, (void *)0);
    }
```

</div>


 -------------------------------- 
<a name="tsesetwolcallback"></a>
## TSE_set_wol_callback
<a name="prototype"></a>
### Prototype 

<div id="Functions$TSE_set_wol_callback$prototype" data-type="code">

    void
    TSE_set_wol_callback
    (
        tse_instance_t * this_tse,
        tse_wol_callback_t wol_callback
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$TSE_set_wol_callback$description" data-type="text">

The `TSE_set_wol_callback()` function registers the function that is called by
the CoreTSE driver when Wake on LAN (WoL) is enabled and a WoL event is
detected. The WoL event happens when either a Unicast match frame or AMD magic
packet frame is detected by CoreTSE. The wol_enable parameter in tse_cfg_t
structure decides which type of packets can cause the WoL event.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_tse
<div id="Functions$TSE_set_wol_callback$description$parameters$this_tse" data-type="text" data-name="this_tse">

The this_tse parameter is a pointer to the tse_instance_t structure holding all
data regarding the CoreTSE instance controlled through this function call.


</div>

#### wol_callback
<div id="Functions$TSE_set_wol_callback$description$parameters$wol_callback" data-type="text" data-name="wol_callback">

The wol_callback parameter is a pointer to the function that is called when Wake
on LAN (WoL) feature is enabled and WoL event happens.


</div>

<a name="return"></a>
### Return
<div id="Functions$TSE_set_wol_callback$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$TSE_set_wol_callback$description$example$Example1" data-type="text" data-name="Example">
The example below demonstrates the use of the `TSE_set_wol_callback()` function.
The `init()` function calls the `TSE_set_wol_callback()` function to register
the wol_callback() receive callback function with the CoreTSE driver. The
wol_callback function is called by the CoreTSE driver once a WoL event is
detected. `wol_callback()` can implement the application-specific action on
detecting the WoL event.


</div>

<div id="Functions$TSE_set_wol_callback$description$example$Example1" data-type="code" data-name="Example">


```
    tse_instance_t g_tse;

    void wol_callback
    (
        void * p_user_data
    )
    {
        //Process WoL interrupt here.
    }
    void init(void)
    {
        TSE_set_wol_callback(&g_tse, wol_callback);
    }
```

</div>


 -------------------------------- 
<a name="tsesendpkt"></a>
## TSE_send_pkt
<a name="prototype"></a>
### Prototype 

<div id="Functions$TSE_send_pkt$prototype" data-type="code">

    uint8_t
    TSE_send_pkt
    (
        tse_instance_t * this_tse,
        uint8_t const * tx_buffer,
        uint32_t tx_length,
        void * p_user_data
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$TSE_send_pkt$description" data-type="text">

The `TSE_send_pkt()` function initiates the transmission of a packet. It places
the buffer containing the packet to send into one of the CoreTSE_AHB's transmit
descriptors. This function is non-blocking. It returns immediately without
waiting for the packet to be sent. The CoreTSE driver indicates that the packet
is sent by calling the transmit completion handler registered by a call to
`TSE_set_tx_callback()`.

Note: This function is applicable only for CoreTSE_AHB.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_tse
<div id="Functions$TSE_send_pkt$description$parameters$this_tse" data-type="text" data-name="this_tse">

The this_tse parameter is a pointer to the tse_instance_t structure holding all
data regarding the CoreTSE_AHB instance controlled through this function call.


</div>

#### tx_buffer
<div id="Functions$TSE_send_pkt$description$parameters$tx_buffer" data-type="text" data-name="tx_buffer">

The tx_buffer parameter is a pointer to the buffer containing the packet to
send.


</div>

#### tx_length
<div id="Functions$TSE_send_pkt$description$parameters$tx_length" data-type="text" data-name="tx_length">

The tx_length parameter specifies the length in bytes of the packet to send.


</div>

#### p_user_data
<div id="Functions$TSE_send_pkt$description$parameters$p_user_data" data-type="text" data-name="p_user_data">

This parameter is a pointer to an optional, application-defined data structure.
Its usage is left up to the application. It is intended to help the application
manage memory allocated to store packets. The CoreTSE driver does not make use
of this pointer. The CoreTSE driver passes back this pointer to the application
as part of the call to the transmit completion handler registered by the
application.


</div>

<a name="return"></a>
### Return
<div id="Functions$TSE_send_pkt$description$return" data-type="text">

This function returns 1 on successfully assigning the packet to a transmit
descriptor. It returns 0 otherwise.


</div>

##### Example
<div id="Functions$TSE_send_pkt$description$example$Example1" data-type="text" data-name="Example">
This example demonstrates the use of the `TSE_send_pkt()` function. The
application registers the `tx_complete_callback()` transmit completion callback
function with the CoreTSE driver by a call to `TSE_set_tx_callback()`. The
application dynamically allocates memory for an application-defined packet_t
data structure builds a packet and calls send_packet(). `send_packet()` extracts
the pointer to the buffer containing the data to transmit and its length from
the tx_packet data structure and passes these to `TSE_send_pkt()`. It also
passes the pointer to tx_packet as the p_user_data parameter. The CoreTSE driver
call `tx_complete_callback()` once the packet is sent. The
`tx_complete_callback()` function uses the p_user_data, which points to
tx_packet, to release memory allocated by the application to store the transmit
packet.


</div>

<div id="Functions$TSE_send_pkt$description$example$Example1" data-type="code" data-name="Example">


```
    void tx_complete_callback(void * p_user_data);
    tse_instance_t g_tse;

    void init(void)
    {
        TSE_set_tx_callback((&g_tse, tx_complete_callback);
    }

    void tx_complete_callback (void * p_user_data)
    {
        release_packet_memory(p_user_data);
    }

    void send_packet(packet_t * tx_packet)
    {
        TSE_send_pkt(&g_tse, tx_packet->buffer, tx_packet->length, tx_packet);
    }
```

</div>


 -------------------------------- 
<a name="tsereceivepkt"></a>
## TSE_receive_pkt
<a name="prototype"></a>
### Prototype 

<div id="Functions$TSE_receive_pkt$prototype" data-type="code">

    uint8_t
    TSE_receive_pkt
    (
        tse_instance_t * this_tse,
        uint8_t * rx_pkt_buffer,
        void * p_user_data
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$TSE_receive_pkt$description" data-type="text">

`TSE_receive_pkt()` assigns a buffer to one of the CoreTSE_AHB receive
descriptors. The receive buffer specified as a parameter receives one single
packet. The receive buffer is handed back to the application via a call to the
receive callback function, assigned through a call to `TSE_set_rx_callback()`.
`TSE_receive_pkt()` needs to be called again pointing to the same buffer if more
packets are to be received into this same buffer after the packet is processed
by the application.

`TSE_receive_pkt()` is non-blocking. It returns immediately and does not wait
for a packet to be received. The application needs to implement a receive
callback function to be notified that a packet is received.

The p_user_data parameter can optionally be used to point to a memory management
data structure managed by the application.

Note: This function is applicable only for CoreTSE_AHB.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_tse
<div id="Functions$TSE_receive_pkt$description$parameters$this_tse" data-type="text" data-name="this_tse">

The this_tse parameter is a pointer to the tse_instance_t structure holding all
data regarding the CoreTSE_AHB instance controlled through this function call.


</div>

#### rx_pkt_buffer
<div id="Functions$TSE_receive_pkt$description$parameters$rx_pkt_buffer" data-type="text" data-name="rx_pkt_buffer">

The rx_pkt_buffer parameter is a pointer to a memory buffer. It points to the
memory that is assigned to one of the CoreTSE_AHB's receive descriptors. It must
point to a buffer large enough to contain the largest possible packet.


</div>

#### p_user_data
<div id="Functions$TSE_receive_pkt$description$parameters$p_user_data" data-type="text" data-name="p_user_data">

The p_user_data parameter is intended to help the application manage memory. Its
usage is left up to the application. The CoreTSE driver does not make use of
this pointer. The CoreTSE driver passes this pointer back to the application as
part of the call to the application receive callback function to help the
application associate the received packet with the memory it allocated before
the call to `TSE_receive_pkt()`.


</div>

<a name="return"></a>
### Return
<div id="Functions$TSE_receive_pkt$description$return" data-type="text">

This function returns 1 on successfully assigning the buffer to a receive
descriptor. It returns 0 otherwise.


</div>

##### Example
<div id="Functions$TSE_receive_pkt$description$example$Example1" data-type="text" data-name="Example">
The example below demonstrates the use of `TSE_receive_pkt()` to handle packet
reception using two receive buffers. The `init()` function calls
`TSE_set_rx_callback()` to register the `rx_callback()` receive callback
function with the CoreTSE driver. `TSE_receive_pkt()` is then called twice to
assign rx_buffer_1 and rx_buffer_2 to CoreTSE descriptors for packet reception.
The rx_callback function is called by the CoreTSE driver once a packet is
received into one of the receive buffers. The `rx_callback()` function calls the
`process_rx_packet()` application function to process the received packet then
calls `TSE_receive_pkt()` to reallocate the receive buffer to receive another
packet. The `rx_callback()` function is called every time a packet is received
to process the received packet and reallocate rx_buffer for packet reception.

Note: The use of the p_user_data parameter to handle the buffer reassignment to
the CoreTSE_AHB as part of the `rx_callback()` function. This is a simplistic
use of p_user_data. It is more likely that p_user_data would be useful to keep
track of a pointer to a TCP/IP stack packet container data structure dynamically
allocated. In this more complex use case, the first parameter of
`TSE_receive_pkt()` would point to the actual receive buffer and the second
parameter would point to a data structure used to free the receive buffer memory
once the packet has been consumed by the TCP/IP stack.


</div>

<div id="Functions$TSE_receive_pkt$description$example$Example1" data-type="code" data-name="Example">


```
    uint8_t rx_buffer_1[TSE_MAX_RX_BUF_SIZE];
    uint8_t rx_buffer_2[TSE_MAX_RX_BUF_SIZE];
    tse_instance_t g_tse;

    void rx_callback
    (
        uint8_t * p_rx_packet,
        uint32_t pckt_length,
        void * p_user_data
    )
    {
        process_rx_packet(p_rx_packet, pckt_length);
        TSE_receive_pkt((&g_tse, (uint8_t *)p_user_data, p_user_data);
    }

    void init(void)
    {
        TSE_set_rx_callback((&g_tse, rx_callback);
        TSE_receive_pkt(rx_buffer_1, (void *)rx_buffer_1);
        TSE_receive_pkt(rx_buffer_2, (void *)rx_buffer_2);
    }
```

</div>


 -------------------------------- 
<a name="tsegetlinkstatus"></a>
## TSE_get_link_status
<a name="prototype"></a>
### Prototype 

<div id="Functions$TSE_get_link_status$prototype" data-type="code">

    uint8_t
    TSE_get_link_status
    (
        tse_instance_t * this_tse,
        tse_speed_t * speed,
        uint8_t * fullduplex
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$TSE_get_link_status$description" data-type="text">

`TSE_get_link_status()` retrieves the status of the link from the Ethernet PHY.
It returns the current state of the Ethernet link. The speed and duplex mode of
the link are also returned via the two pointers passed as parameters - if the
link is up.

This function also adjusts the CoreTSE's internal configuration if some of the
link characteristics have changed since the previous call to this function. As
such, it is recommended to call this function periodically, so that the driver
can automatically adapt to changes in the network link status.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_tse
<div id="Functions$TSE_get_link_status$description$parameters$this_tse" data-type="text" data-name="this_tse">

The this_tse parameter is a pointer to the tse_instance_t structure holding all
data regarding the CoreTSE instance controlled through this function call.


</div>

#### speed
<div id="Functions$TSE_get_link_status$description$parameters$speed" data-type="text" data-name="speed">

The speed parameter is a pointer to a variable of type tse_speed_t where the
current link speed is stored if the link is up. This variable is not updated if
the link is down. This parameter can be set to zero if the caller does not need
to find out the link speed.


</div>

#### fullduplex
<div id="Functions$TSE_get_link_status$description$parameters$fullduplex" data-type="text" data-name="fullduplex">

The full-duplex parameter is a pointer to an unsigned character where the
current link duplex mode is stored if the link is up. This variable is not
updated if the link is down.


</div>

<a name="return"></a>
### Return
<div id="Functions$TSE_get_link_status$description$return" data-type="text">

This function returns 1 if the link is up. It returns 0 if the link is down.


</div>

##### Example
<div id="Functions$TSE_get_link_status$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$TSE_get_link_status$description$example$Example1" data-type="code" data-name="Example">


```
    uint8_t link_up;
    tse_speed_t speed;
    uint8_t full_duplex
    tse_instance_t g_tse;

    link_up = TSE_get_link_status(&g_tse, &speed, &full_duplex);
```

</div>


 -------------------------------- 
<a name="tsereadstat"></a>
## TSE_read_stat
<a name="prototype"></a>
### Prototype 

<div id="Functions$TSE_read_stat$prototype" data-type="code">

    uint32_t
    TSE_read_stat
    (
        tse_instance_t * this_tse,
        tse_stat_t stat
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$TSE_read_stat$description" data-type="text">

`TSE_read_stat()` reads the transmit and receive statistics of the CoreTSE. This
function can be used to read one of seventeen receiver statistics, twenty
transmitter statistics and seven frame-type statistics as defined in the
tse_stat_t enumeration.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_tse
<div id="Functions$TSE_read_stat$description$parameters$this_tse" data-type="text" data-name="this_tse">

The this_tse parameter is a pointer to the tse_instance_t structure holding all
data regarding the CoreTSE instance controlled through this function call.


</div>

#### stat
<div id="Functions$TSE_read_stat$description$parameters$stat" data-type="text" data-name="stat">

This parameter of type tse_stat_t identifies the statistic that is read. Refer
to the tse_stat_t type definition to see the allowed values.


</div>

<a name="return"></a>
### Return
<div id="Functions$TSE_read_stat$description$return" data-type="text">

This function returns the value of the requested statistic.


</div>

##### Example
<div id="Functions$TSE_read_stat$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$TSE_read_stat$description$example$Example1" data-type="code" data-name="Example">


```
    uint32_t tx_pkts_cnt = 0;
    tse_instance_t g_tse;

    tx_pkts_cnt = TSE_read_stat(&g_tse, TSE_TX_PKT_CNT);
```

</div>


 -------------------------------- 
<a name="tseclearstatistics"></a>
## TSE_clear_statistics
<a name="prototype"></a>
### Prototype 

<div id="Functions$TSE_clear_statistics$prototype" data-type="code">

    void
    TSE_clear_statistics
    (
        tse_instance_t * instance
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$TSE_clear_statistics$description" data-type="text">

`TSE_clear_statistics()` clears all forty-four statistics counter registers.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_tse
<div id="Functions$TSE_clear_statistics$description$parameters$this_tse" data-type="text" data-name="this_tse">

The this_tse parameter is a pointer to the tse_instance_t structure holding all
data regarding the CoreTSE instance controlled through this function call.


</div>

<a name="return"></a>
### Return
<div id="Functions$TSE_clear_statistics$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$TSE_clear_statistics$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$TSE_clear_statistics$description$example$Example1" data-type="code" data-name="Example">


```
    uint32_t tx_pkts_cnt = 0;

    tx_pkts_cnt = TSE_clear_statistics(&g_tse);
```

</div>


 -------------------------------- 
<a name="tsereadphyreg"></a>
## TSE_read_phy_reg
<a name="prototype"></a>
### Prototype 

<div id="Functions$TSE_read_phy_reg$prototype" data-type="code">

    uint16_t
    TSE_read_phy_reg
    (
        tse_instance_t * this_tse,
        uint8_t phyaddr,
        uint8_t regaddr
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$TSE_read_phy_reg$description" data-type="text">

`TSE_read_phy_reg()` reads the Ethernet PHY register specified as a parameter.
It uses the MII management interface to communicate with the Ethernet PHY. This
function is part of the Ethernet PHY drivers provided alongside the CoreTSE
driver. You only need to use this function if writing your own Ethernet PHY
driver.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_tse
<div id="Functions$TSE_read_phy_reg$description$parameters$this_tse" data-type="text" data-name="this_tse">

The this_tse parameter is a pointer to the tse_instance_t structure holding all
data regarding the CoreTSE instance controlled through this function call.


</div>

#### phyaddr
<div id="Functions$TSE_read_phy_reg$description$parameters$phyaddr" data-type="text" data-name="phyaddr">

The phyaddr parameter is the 5-bit address of the Ethernet PHY on the MII
management interface. This address is typically defined through Ethernet PHY
hardware configuration signals. Please refer to the Ethernet PHY's datasheet for
details of how this address is assigned.


</div>

#### regaddr
<div id="Functions$TSE_read_phy_reg$description$parameters$regaddr" data-type="text" data-name="regaddr">

The regaddr parameter is the address of the Ethernet register that is read. This
address is the offset of the register within the Ethernet PHY's register map.


</div>

<a name="return"></a>
### Return
<div id="Functions$TSE_read_phy_reg$description$return" data-type="text">

This function returns the 16-bit value of the requested register.


</div>

##### Example
<div id="Functions$TSE_read_phy_reg$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$TSE_read_phy_reg$description$example$Example1" data-type="code" data-name="Example">


```
    #include -phy.h-
    tse_instance_t g_tse;

    uint16_t read_phy_status(uint8_t phy_addr)
    {
        uint16_t phy_status = TSE_read_phy_reg(&g_tse, phy_addr , MII_BMSR);
        return phy_status;
    }
```

</div>


 -------------------------------- 
<a name="tsewritephyreg"></a>
## TSE_write_phy_reg
<a name="prototype"></a>
### Prototype 

<div id="Functions$TSE_write_phy_reg$prototype" data-type="code">

    void
    TSE_write_phy_reg
    (
        tse_instance_t * this_tse,
        uint8_t phyaddr,
        uint8_t regaddr,
        uint16_t regval
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$TSE_write_phy_reg$description" data-type="text">

`TSE_write_phy_reg()` writes a 16-bit value to the specified Ethernet PHY
register. It uses the MII management interface to communicate with the Ethernet
PHY. This function is part of the Ethernet PHY drivers provided alongside the
CoreTSE driver. You only need to use this function if writing your own Ethernet
PHY driver.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_tse
<div id="Functions$TSE_write_phy_reg$description$parameters$this_tse" data-type="text" data-name="this_tse">

The this_tse parameter is a pointer to the tse_instance_t structure holding all
data regarding the CoreTSE instance controlled through this function call.


</div>

#### phyaddr
<div id="Functions$TSE_write_phy_reg$description$parameters$phyaddr" data-type="text" data-name="phyaddr">

The phyaddr parameter is the 5-bit address of the Ethernet PHY on the MII
management interface. This address is typically defined through Ethernet PHY
hardware configuration signals. Please refer to the Ethernet PHY's datasheet for
details of how this address is assigned.


</div>

#### regaddr
<div id="Functions$TSE_write_phy_reg$description$parameters$regaddr" data-type="text" data-name="regaddr">

The regaddr parameter is the address of the Ethernet register that is written.
This address is the offset of the register within the Ethernet PHY's register
map.


</div>

#### regval
<div id="Functions$TSE_write_phy_reg$description$parameters$regval" data-type="text" data-name="regval">

The regval parameter is the 16-bit value that is written into the specified PHY
register.


</div>

<a name="return"></a>
### Return
<div id="Functions$TSE_write_phy_reg$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$TSE_write_phy_reg$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$TSE_write_phy_reg$description$example$Example1" data-type="code" data-name="Example">


```
    #include "phy.h"
    tse_instance_t g_tse;

    void rest_sgmii_phy(void)
    {
        TSE_write_phy_reg(&g_tse, SGMII_PHY_ADDR, MII_BMCR, 0x8000);
    }
```

</div>


 -------------------------------- 
<a name="tseisr"></a>
## TSE_isr
<a name="prototype"></a>
### Prototype 

<div id="Functions$TSE_isr$prototype" data-type="code">

    void
    TSE_isr
    (
        tse_instance_t * this_tse
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$TSE_isr$description" data-type="text">

`TSE_isr()` is the top-level interrupt handler function for the CoreTSE driver.
You must call `TSE_isr()` from the system-level (CoreInterrupt and NVIC level)
interrupt handler assigned to the interrupt triggered by the CoreTSE INTR
signal. Your system-level interrupt handler must also clear the system-level
interrupt triggered by the CoreTSE INTR signal before returning, to prevent a
re-assertion of the same interrupt.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_tse
<div id="Functions$TSE_isr$description$parameters$this_tse" data-type="text" data-name="this_tse">

The this_tse parameter is a pointer to the tse_instance_t structure holding all
data regarding the CoreTSE instance controlled through this function call.


</div>

<a name="return"></a>
### Return
<div id="Functions$TSE_isr$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$TSE_isr$description$example$Example1" data-type="text" data-name="Example">

</div>

<div id="Functions$TSE_isr$description$example$Example1" data-type="code" data-name="Example">


```
    tse_instance_t g_tse;

    void FabricIrq0_IRQHandler
    (
        void
    )
    {
        TSE_isr(&g_tse);
    }
```

</div>


 -------------------------------- 
<a name="tsesetaddressfilter"></a>
## TSE_set_address_filter
<a name="prototype"></a>
### Prototype 

<div id="Functions$TSE_set_address_filter$prototype" data-type="code">

    void
    TSE_set_address_filter
    (
        tse_instance_t * this_tse,
        const uint8_t * mac_addresses,
        uint32_t nb_addresses
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$TSE_set_address_filter$description" data-type="text">

`TSE_set_address_filter()` implements the frame filtering functionality of the
driver. This function specifies the list of destination MAC addresses of
received frames that are passed to the MAC. This function takes an array of MAC
addresses as a parameter and generates the correct hash table for that list of
addresses. It also manages the setting of the Broadcast-DA, Multicast-
DA/Unicast-DA hardware IP configuration bits based on the value of the MAC
addresses passed as a parameter. For example, if the list contains one or more
unicast addresses then unicast hash match filtering is enabled. Likewise,
multicast hash match filtering is enabled if the list contains one or more
multicast addresses. It enables broadcast filtering if the broadcast address is
included in the allowed MAC addresses list passed as a parameter. It also
enables passing frames addressed to the local MAC address (perfect unicast
match) if the local MAC address is included in the allowed MAC addresses list.

Note: The address filtering choice can also be selected using the
`TSE_cfg_struct_def_init()` function. The configuration value returned by this
function can be modified and then passed on to the `TSE_init()` function. If the
`TSE_set_address_filter()` function is used, the original filter setting chosen
at the initialization time gets overwritten.

Note: Each MAC address consists of six octets and must be placed in the buffer
starting with the first (most significant) octet of the MAC address.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### this_tse
<div id="Functions$TSE_set_address_filter$description$parameters$this_tse" data-type="text" data-name="this_tse">

The this_tse parameter is a pointer to the tse_instance_t structure holding all
data regarding the CoreTSE instance controlled through this function call.


</div>

#### mac_addresses
<div id="Functions$TSE_set_address_filter$description$parameters$mac_addresses" data-type="text" data-name="mac_addresses">

The mac_addresses parameter is a pointer to the buffer containing the MAC
addresses that are used to generate the MAC address hash table.


</div>

#### nb_addresses
<div id="Functions$TSE_set_address_filter$description$parameters$nb_addresses" data-type="text" data-name="nb_addresses">

The nb_addresses parameter specifies the number of MAC addresses being passed in
the buffer pointed by the mac_addresses buffer pointer.


</div>

<a name="return"></a>
### Return
<div id="Functions$TSE_set_address_filter$description$return" data-type="text">

This function does not return a value.


</div>

##### Example
<div id="Functions$TSE_set_address_filter$description$example$Example1" data-type="text" data-name="Example">
This example demonstrates the use of the `TSE_set_address_filter()` function to
handle frame filtering. The list of MAC addresses passed to
`TSE_set_address_filter()` function includes unicast, multicast, local base
station and broadcast MAC addresses. The `TSE_set_address_filter()` function
sets up the hash-unicast, hash-multicast, broadcast and perfect unicast match
(local base station address) filtering to pass frames with these destination
addresses to the MAC.


</div>

<div id="Functions$TSE_set_address_filter$description$example$Example1" data-type="code" data-name="Example">


```
    #define TSEMAC_BASE  0x30000000
    tse_instance_t g_tse;
    tse_cfg_t g_tse_config;

    uint8_t mac_data[4][6] = {{0x10, 0x10, 0x10, 0x10, 0x10, 0x10},
                            {0x43, 0x40, 0x40, 0x40, 0x40, 0x43},
                            {0xC0, 0xB1, 0x3C, 0x60, 0x60, 0x60},
                            {0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF}};

    g_tse_config.phy_addr = TSE_PHY_ADDRESS_AUTO_DETECT;
    g_tse_config.speed_duplex_select = TSE_ANEG_100M_FD;
    g_tse_config.mac_addr[0] = 0xC0;
    g_tse_config.mac_addr[1] = 0xB1;
    g_tse_config.mac_addr[2] = 0x3C;
    g_tse_config.mac_addr[3] = 0x60;
    g_tse_config.mac_addr[4] = 0x60;
    g_tse_config.mac_addr[5] = 0x60;

    TSE_init(&g_tse, TSEMAC_BASE, &g_tse_config);

    TSE_set_address_filter(&g_tse, mac_data[0], 4);
```

</div>


 -------------------------------- 
