# Precision Time Protocol (PTP) on PolarFire SoC GEM

This document describes the PTP support on PolarFire Soc, including supported features, limitations and
some information on basic testing.

## Table of Contents

- [Requirements](#requirements)
- [Introduction](#introduction)
  - [Modes of Operation](#modes-of-operation)
- [Missing Features, Known Issues and Limitations](#missing-features-known-issues-and-limitations)
- [Linux Requirements for PTP](#linux-requirements-for-ptp)
  - [Kernel Configuration](#kernel-configuration)
  - [Device Tree Requirements](#device-tree-requirements)
- [PTP Test Procedure](#ptp-test-procedure)
  - [Test Setup](#test-setup)
  - [Using LinuxPTP](#using-linuxptp)

<a name="requirements"></a>

## Requirements

- 2x PolarFire SoC Kits

  (e.g. Icicle Kit, BeagleV-Fire, Discovery Kit, etc) running PolarFire SoC
  Linux release v2025.10.1

- 1x standard crossover network cable

<a name="introduction"></a>

## Introduction

Precision Time Protocol (PTP), defined in the IEEE 1588 standard, is used to synchronize clocks throughout a packet-based network. PTP provides highly accurate and reliable time synchronization across local or wide area networks for various systems and applications.

PolarFire SoC contains two identical Gigabit Ethernet MACs (GEMs) integrated into the MSS, each with built-in hardware support for IEEE 1588 PTP. Each GEM block includes a Timestamping Unit (TSU). The MAC provides support for:

- Identifying PTP frames

- Extracting timestamp information from received PTP frames

- Inserting timestamp information into transmitted data frames

<a name="modes-of-operation"></a>

### Modes of Operation

The TSU clock can be controlled via the MSS or the FPGA fabric.

There are two modes of operation configurable via the MSS Configurator:

1. **Increment Mode (MSS Controlled)**
  - In this mode, the clock is configured to come from either the FPGA Fabric or an off-chip reference clock (100 MHz or 125 MHz).
  - The clock source is selected via User GPIO signals.
  - By default, PolarFire SoC reference designs are configured to use Increment Mode, and Linux is configured to use the 125 MHz reference clock.
  - Increment Mode is supported by Linux using standard PTP user space tools such as LinuxPTP.

2. **Timer Adjust Mode (FPGA Fabric Controlled)**
  - In this mode, the TSU clock is configured to come from the FPGA fabric.
  - Additional input signals from the fabric can control timer operation, allowing for discrete time adjustments.
  - Timer Adjust Mode is intended for use cases where the TSU is controlled directly by the FPGA fabric, and therefore is not supported in Linux.

For more information, please refer to the Time Stamping Unit section in the [PolarFire SoC Technical Reference Manual](https://ww1.microchip.com/downloads/aemDocuments/documents/FPGA/ProductDocuments/ReferenceManuals/PolarFire_SoC_FPGA_MSS_Technical_Reference_Manual_VC.pdf).

<a name="missing-features-known-issues-and-limitations"></a>

## Missing Features, Known Issues and Limitations

- PTP over IEEE 802.3 (Ethernet) is tested and supported.

This method uses Ethernet frames at Layer 2 to synchronize clocks across the network.

- PTP over IP (UDP/IP) is not supported.

PTP frames must be transmitted as raw Ethernet frames, not encapsulated in UDP/IP packets.

<a name="linux-requirements-for-ptp"></a>

## Linux Requirements for PTP

PolarFire SoC PTP support for the GEM (Gigabit Ethernet MAC) is available starting from the v2025.10.1 Linux release and onwards.

<a name="kernel-configuration"></a>

### Kernel Configuration

To enable hardware timestamping support in the MACB driver and its dependencies, ensure the following kernel configuration options are set:

```text
CONFIG_MACB_USE_HWSTAMP=y
CONFIG_PTP_1588_CLOCK=y
```

These options are enabled by default from release v2025.10.1 and onwards.

<a name="device-tree-requirements"></a>

### Device Tree Requirements

The TSU (Timestamping Unit) clock must be specified in the MAC node(s) within the device tree. For example, to provide a 125 MHz reference clock as the TSU clock:

```text
mac0: ethernet@20110000 {
    compatible = "microchip,mpfs-macb", "cdns,macb";
    reg = <0x0 0x20110000 0x0 0x2000>;
    #address-cells = <1>;
    #size-cells = <0>;
    interrupt-parent = <&plic>;
    interrupts = <64>, <65>, <66>, <67>, <68>, <69>;
    local-mac-address = [00 00 00 00 00 00];
    clocks = <&clkcfg CLK_MAC0>, <&clkcfg CLK_AHB>, <&refclk>;
    clock-names = "pclk", "hclk", "tsu_clk";
    resets = <&mss_top_sysreg CLK_MAC0>;
};
```

This device tree configuration is included in release v2025.10.1 and onwards.

<a name="ptp-test-procedure"></a>

## PTP Test Procedure

In the Linux ecosystem, Linux PTP is the most popular implementation of the Precision Time Protocol (PTP).

The linuxptp package is included in both Microchip Buildroot and Yocto Project layer starting from PolarFire SoC release v2025.10.1 and onwards.

<a name="test-setup"></a>

### Test Setup

- Devices Required:

Two devices are needed for PTP testing, one configured as the master and the other as the slave.

- Recommended Hardware:

For demonstration purposes, two PolarFire SoC Kits are used. However, any device with a Network Interface Card (NIC) that supports hardware timestamping can be used.

- Connection:

The two PolarFire SoC boards are connected via their respective Ethernet ports using a standard crossover network cable.

<a name="using-linux-ptp"></a>

### Using LinuxPTP

1. On the first board, run ptp4l in master mode using the default configuration (two-step mode, E2E delay request-response), and IEEE 802.3 (Ethernet) network transport:

    ```text
    ptp4l -i eth0 -m -2
    ```

2. On the second board, run ptp4l in slave mode using the default configuration (two-step mode, E2E delay request-response), and IEEE 802.3 (Ethernet) network transport:

    ```text
    ptp4l -i eth0 -m -s -2
    ```

3. After running ptp4l on both ends, the slave should display the frequency sinchronization as shown int the log below:

    ```text
    root@beaglev-fire:~# ptp4l -i eth0 -m -s -2
    ptp4l[95.636]: selected /dev/ptp0 as PTP clock
    ptp4l[95.672]: port 1 (eth0): INITIALIZING to LISTENING on INIT_COMPLETE
    ptp4l[95.673]: port 0 (/var/run/ptp4l): INITIALIZING to LISTENING on INIT_COMPLETE
    ptp4l[95.673]: port 0 (/var/run/ptp4lro): INITIALIZING to LISTENING on INIT_COMPLETE
    ptp4l[101.726]: selected local clock 0004a3.fffe.481150 as best master
    ptp4l[313.182]: port 1 (eth0): new foreign master 0004a3.fffe.92cc3e-1
    ptp4l[317.182]: selected best master clock 0004a3.fffe.92cc3e
    ptp4l[317.182]: port 1 (eth0): LISTENING to UNCALIBRATED on RS_SLAVE
    ptp4l[319.181]: master offset -360052570306 s0 freq      -0 path delay      2834
    ptp4l[320.181]: master offset -360052577296 s1 freq   -6989 path delay      2320
    ptp4l[321.181]: master offset      -2261 s2 freq   -9250 path delay      2320
    ptp4l[321.182]: port 1 (eth0): UNCALIBRATED to SLAVE on MASTER_CLOCK_SELECTED
    ptp4l[322.181]: master offset       -969 s2 freq   -8637 path delay      2314
    ptp4l[323.182]: master offset        -37 s2 freq   -7995 path delay      2314
    ptp4l[324.182]: master offset       1211 s2 freq   -6759 path delay      1525
    ptp4l[325.182]: master offset        586 s2 freq   -7020 path delay      1525
    ptp4l[326.182]: master offset        447 s2 freq   -6983 path delay      1099
    ptp4l[327.182]: master offset       -163 s2 freq   -7459 path delay      1099
    ptp4l[328.182]: master offset        173 s2 freq   -7172 path delay       901
    ptp4l[329.182]: master offset        250 s2 freq   -7043 path delay       872
    ```
