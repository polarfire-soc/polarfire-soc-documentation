# Linux Power Management on PolarFire SoC

This document describes the Linux power management features available on PolarFire SoC,
including system suspend/resume and CPU frequency scaling.

## Table of Contents

- [Introduction](#introduction)
- [Requirements](#requirements)
- [System Suspend/Resume](#system-suspendresume)
- [CPU Frequency Scaling](#cpu-frequency-scaling)
  - [Kernel Configuration](#kernel-configuration)
  - [Available Governors](#available-governors)
  - [Changing the CPU Frequency Governor](#changing-the-cpu-frequency-governor)
  - [Changing the CPU Frequency](#changing-the-cpu-frequency)

<a name="introduction"></a>

## Introduction

PolarFire SoC supports several Linux power management features that allow users to reduce power
consumption in their applications. These features include:

- System suspend/resume
- CPU frequency scaling
- DDR self-refresh and clock powerdown

System suspend allows the processor complex to enter a low-power state while preserving the system
state in memory, enabling fast resume times.

CPU frequency scaling allows the operating frequency of the U54 application coreplex to be adjusted
dynamically based on workload, reducing power consumption during periods of low activity.

It can also be automatically lowered to a minimum value during suspend, saving extra power at the
expense of wake-up latency.

<a name="requirements"></a>

## Requirements

- A PolarFire SoC board (e.g. Icicle Kit)
- A serial terminal connected to the board's UART console
- PolarFire SoC Linux release v2024.06 and onwards

<a name="system-suspendresume"></a>

## System Suspend/Resume

Suspend/resume support is available on PolarFire SoC starting from Hart Software Services (HSS) release v2026.04.

It is implemented via the RISC-V Supervisor Binary Interface (SBI) SUSP extension (`SBI_EXT_SUSP`), which provides a
standard interface for Linux to request system suspend from the HSS acting as the SBI implementation.

Suspend to RAM saves the system state to RAM. When entering suspend, the DDR memory
is placed into self-refresh mode. In self-refresh, the DRAM retains its contents while consuming
minimal power by autonomously performing internal refresh cycles without requiring access from the memory controller.

To suspend the system from Linux, run the following command:

```bash
# systemctl suspend
```

The system will suspend immediately, producing output similar to the following:

```text
[  229.153724] PM: suspend entry (deep)
[  229.172614] Filesystems sync: 0.015 seconds
[  229.179342] Freezing user space processes
[  229.185136] Freezing user space processes completed (elapsed 0.001 seconds)
[  229.192216] OOM killer disabled.
[  229.195519] Freezing remaining freezable tasks
[  229.201346] Freezing remaining freezable tasks completed (elapsed 0.001 seconds)
[  229.208856] printk: Suspending console(s) (use no_console_suspend to debug)
```

To resume, trigger the configured wake-up source (e.g. the TinyCLI `resume` command via the HSS
UART console). The memory controller will exit self-refresh and normal operation will be restored
before Linux regains control, producing output similar to the following:

```text
[  229.242217] Enabling non-boot CPUs ...
[  229.243487] CPU1 is up
[  229.252325] CPU2 is up
[  229.261713] CPU3 is up
[  229.268441] macb 20112000.ethernet end0: configuring for phy/sgmii link mode
[  229.269607] ptp ptp0: PM: parent end0 should not be sleeping
[  229.269865] macb 20112000.ethernet: gem-ptp-timer ptp clock registered.
[  229.273487] macb 20110000.ethernet eth0: configuring for phy/sgmii link mode
[  229.274623] ptp ptp1: PM: parent eth0 should not be sleeping
[  229.274900] macb 20110000.ethernet: gem-ptp-timer ptp clock registered.
[  229.357182] OOM killer enabled.
[  229.360408] Restarting tasks: Starting
[  229.365250] Restarting tasks: Done
[  229.368980] random: crng reseeded on system resumption
[  229.374587] PM: suspend exit
[  229.444756] sdhci-cdns 20008000.mmc: no tuning point found
[  234.383363] macb 20112000.ethernet end0: Link is Up - 1Gbps/Full - flow control off
```

HSS versions later than v2026.04.1 support extra options via Kconfig:

- SERVICE\_OPENSBI\_SUSPEND\_CLOCK\_SCALING

This feature lowers CPU, AXI and AHB/APB clocks while Linux is suspended, and retunes
MMUART0 around the clock change. The frequencies become 75/75/75 MHz, respectively.

**The user is responsible to ensure that any peripherals can cope with the change in clock
frequency during suspend.**

- SERVICE\_OPENSBI\_SUSPEND\_SCB\_CLOCK\_SWITCH

This feature switches clk\_in\_mss to the 80 MHz SCB standby clock while Linux is suspended,
then disables the MSS PLL Q0 output. The frequencies become 10/10/10 MHz. As the SCB is an
RC oscillator, clock stability will be degraded.

- SERVICE\_OPENSBI\_SUSPEND\_DDR\_POWER\_SAVE

This feature enables the tested DDR suspend power sequence: disable DRAM clocks through the DDR
controller while in self-refresh, put DDR logic in low power, gate DDR PLL outputs, and gate the
DDRC sub-block clock until resume.

**The user is responsible to ensure that there is no activity from the fabric to the DRAM during
suspend**

<a name="cpu-frequency-scaling"></a>

## CPU Frequency Scaling

CPU frequency scaling allows the operating frequency of the U54 application
coreplex to be adjusted at runtime. This can reduce power consumption and heat dissipation during
periods of low CPU activity.

Three operating points are available: 600 MHz, 300 MHz, and 150 MHz. The available operating points
can be listed using the `cpupower` utility:

```bash
cpupower frequency-info
```

Example output:

```text
analyzing CPU 0:
  driver: cpufreq-dt
  CPUs which run at the same hardware frequency: 0 1 2 3
  CPUs which need to have their frequency coordinated by software: 0 1 2 3
  maximum transition latency: 1000 us
  hardware limits: 150 MHz - 600 MHz
  available frequency steps:  150 MHz, 300 MHz, 600 MHz
  available cpufreq governors: performance schedutil
  current policy: frequency should be within 150 MHz and 600 MHz.
                  The governor "performance" may decide which speed to use
                  within this range.
  current CPU frequency: 600 MHz (asserted by call to kernel)
  boost state support:
    Active: no
```

<a name="kernel-configuration"></a>

### Kernel Configuration

To enable CPU frequency scaling, the following kernel configuration options must be enabled:

```text
CONFIG_CPU_FREQ=y
CONFIG_CPU_FREQ_STAT=y
CONFIG_CPU_FREQ_GOV_POWERSAVE=m
CONFIG_CPU_FREQ_GOV_USERSPACE=m
CONFIG_CPU_FREQ_GOV_ONDEMAND=m
CONFIG_CPU_FREQ_GOV_CONSERVATIVE=m
CONFIG_CPU_FREQ_GOV_SCHEDUTIL=y
CONFIG_CPUFREQ_DT=y
```

These options are enabled by default in the PolarFire SoC Linux releases starting from release v2026.04 and onwards.

<a name="available-governors"></a>

### Available Governors

A CPUFreq governor defines the policy used to select the CPU operating frequency. The active
governor can be listed using the `cpupower` utility:

```bash
cpupower frequency-info -p
```

<a name="changing-the-cpu-frequency-governor"></a>

### Changing the CPU Frequency Governor

The following governors are built into the kernel by default on PolarFire SoC:

- `performance`: Always selects the highest available frequency. This is the default governor.
- `schedutil`: Dynamically adjusts frequency based on CPU scheduler utilisation statistics.

The following governors are available as loadable kernel modules:

- `powersave`: Always selects the lowest available frequency.
- `userspace`: Allows a specific frequency to be set manually from user space.
- `ondemand`: Dynamically adjusts frequency based on CPU load, scaling up quickly under load.
- `conservative`: Similar to `ondemand` but scales frequency up and down more gradually.

To load a governor module, use `modprobe`. For example:

```bash
modprobe cpufreq_ondemand
```

To change the active governor, use the `cpupower` utility. For example, to switch to the `schedutil` governor (recommended):

```bash
cpupower frequency-set -g schedutil
```

To verify the active governor:

```bash
cpupower frequency-info -p
```

<a name="changing-the-cpu-frequency"></a>

### Changing the CPU Frequency

The available CPU frequencies can be listed with:

```bash
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_frequencies
```

To set a specific frequency using the `cpupower` utility, first switch to the `userspace` governor and then set the desired frequency:

```bash
modprobe cpufreq_userspace
cpupower frequency-set -g userspace
cpupower frequency-set -f 300MHz
```

The current operating frequency can be verified with:

```bash
cpupower frequency-info -f
```
