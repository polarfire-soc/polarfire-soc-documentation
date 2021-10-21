# Mi-V Inter-hart Communication (IHC) Subsystem

## Table of Contents
- [Introduction](#ihc-intro)
- [Configuration](#ihc-config)
    - [Channel Allocation](#ihc-alloc)
    - [Layout](#ihc-layout)
- [Associated Software](#ihc-software)
    - [Hart Software Services (HSS)](#ihc-hss)
    - [Operating Systems](#ihc-os)

## Introduction <a name="ihc-intro"></a>

The Inter-hart Communication (IHC) subsystem can be used to exchange data between harts in PolarFire SoC. It provides the ability to communicate and coordinate between harts through a non-blocking interrupt signaling mechanism. 

Internally, the IHC consists in multiple interconnected instances of the following components:

- **Inter-hart communication channels (IHCC)**
- **Inter-hart communication interrupt aggregators (IHCIA)**

![ihc-overview](images/ihc-overview.png)
### Inter-hart Communication Channel (IHCC)

The Icicle Kit Reference Design's AMP subsystem provides up to ten inter-hart communication channels (IHCC) of which:

- Four are dedicated for communication between the monitor hart (E51) and applications harts (U54's) using the Hart Software Services (HSS) 
- One is assigned for communication between two software contexts at operating system level using the RPMsg protocol

The six remaining channels are not used by the software. These free channels could be used to extend the inter-hart communication if required.

Each IHC channel is divided into two unidirectional subchannels that provide a signaling mechanism between a "sender" and a "receiver" hart. 

Each subchannel consists of:

- Up to four 32-bit "message out" write-only registers that can be used to send data to the receiving hart.

- Up to four 32-bit "message in" read-only registers that can be used to read a message sent by the sender hart

> The "message in" and "message out" registers could be used to send/receive small amounts of data such as a message ID, a numerical index, a custom control data structure, or a pointer to another location in memory (DDR, LIM, etc.)

- Two associated interrupts:
    - message present interrupt: set when a message posted by the sending processor is available to be consumed on the receiving hart

    - acknowledge interrupt: set when a message posted by the sending processor has been retrieved by the receiving hart

![ihcc-zoom](images/ihcc-zoom.png)

> Note: The purpose of this IHC is to provide a signaling mechanism between harts. Therefore, the actual data to be shared between software contexts should be located in a shared memory area which is not part of the IHC. Some examples of shared memory areas include DDR or LIM.

### Inter-hart Communication Interrupt Aggregator (IHCIA)

The Inter-hart Communication Interrupt Aggregator (IHCIA) component has two main purposes:

- Manages interrupts from several channels in order to group them on a hart-level basis

- Provides a mechanism to quickly identify the source IHC channel that sent an interrupt to a particular hart


![ihcia-zoomin](images/ihcia-zoom.png)

The IHC subsystem contains five Inter-hart communication interrupt aggregators (IHCIA's), one for each of the five harts available in PolarFire SoC.

## IHC Subsystem Configuration <a name="ihc-config"></a>

This section provides an overview of the default IHC configuration provided in the [Icicle Kit reference design](https://github.com/polarfire-soc/icicle-kit-reference-design).

### Channel Assingment <a name="ihc-alloc"></a>

The table below provides a list of all IHCC's available on PolarFire SoC, as well as the software associated with each of these channels:

| Channel Name     | Software framework     |
|-------------     |------------------------|
| E51<->U54_1      | HSS                    |
| E51<->U54_2      | HSS                    |
| E51<->U54_3      | HSS                    |
| E51<->U54_4      | HSS                    |
| U54_1<->U54_4    | RPMsg (Context A <-> B)|
| U54_1<->U54_3    | Unused                 |
| U54_1<->U54_2    | Unused                 |
| U54_2<->U54_3    | Unused                 |
| U54_3<->U54_4    | Unused                 |
| U54_2<->U54_4    | Unused                 |

Channels E51<->U54_1, E51<->U54_2, E51<->U54_3 and E51<->U54_4 are used to communicate the monitor hart (E51) with the application harts using the Hart Software Services. For more information on the IHC usage by the HSS, please refer to the [Associated Software](#ihc-hss) section within this page.

The U54_1 <-> U54_4 channel is assigned for context A to context B communication using the RPMsg protocol.

The above implies that the AMP configuration used should have at least U54_1 assigned to context A and U54_4 assigned to context B.

### Layout <a name="ihc-layout"></a>

This image below provides a simplified diagram representing the connection between the channels (IHCCs) and the IHCIAs which conform the IHC.

![ihc-basic-view](images/ihc-animation.gif)

## Associated Software <a name="ihc-software"></a>

### Operating Systems <a name="ihc-os"></a>

The IHC is handled by Linux through the Linux mailbox framework and by RTOS/bare metal applications through the bare metal [Mi-V IHC driver]().

The IHC is used by the RPMsg framework to:

a) notify a 'remote' software context that a message has been posted in a ring buffer (vring) located in shared memory

b) send/receive the ring buffer (vring) ID where the sender context posted a message

For more information on the RPMsg framework, please refer to the [RPMsg](https://github.com/polarfire-soc/polarfire-soc-documentation/blob/master/asymmetric-multiprocessing/rpmsg.md) documentation page

#### Context A to Context B Communication 

![context-a-b](images/ihc-rpmsg-a-b.png)


#### Context B to Context A Communication 

![context-b-a](images/ihc-rpmsg-b-a.png)


## Further Reading

For further information on the IHC, including its API please refer to the [IHC user guide](https://bitbucket.microchip.com/projects/FPGA_BU_SOFT_IP_ES/repos/coreipc-internal/browse/.md-dev/md-out/core-ihcm-user-guide.md?at=refs%2Fheads%2Fwip).

