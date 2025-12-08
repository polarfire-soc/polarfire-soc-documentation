# PolarFire SoC: Hart Software Services Health Monitoring Service

- [Overview](#overview)
- [State Machine Structure](#state-machine-structure)
  - [States](#states)
- [Key Components](#key-components)
  - [Monitor Definition (`struct HealthMonitor`)](#monitor-definition-(struct-healthmonitor))
  - [Runtime Status (`struct HealthMonitor_Status`)](#runtime-status-(struct-healthmonitor-status))
- [Monitoring Logic](#monitoring-logic)
  - [Check Types](#check-types)
  - [Shift and Mask](#shift-and-mask)
  - [Throttle / Rate Limiting](#throttle--rate-limiting)
  - [Trigger Callbacks](#trigger-callbacks)
- [Initialization and Dependencies](#initialization-and-dependencies)
- [Configuration and Extensibility](#configuration-and-extensibility)
- [Debugging and Statistics](#debugging-and-statistics)
- [Flow Summary](#flow-summary)
- [Guidance and Best Practices](#guidance-and-best-practices)
- [Example of BSP-specific Overrides](#example-of-bsp-specific-overrides)
- [Conclusion](#conclusion)

**Scope.** This document describes the purpose, structure, and operation of the HSS Health
Monitoring service for PolarFire SoC designs. It explains how to define health “monitors”,
what conditions they detect, and how the service reports and reacts to anomalies.

Please refer to the [PolarFire SoC Microprocessor Subsystem (MSS) User Guide](https://ww1.microchip.com/downloads/aemDocuments/documents/FPGA/ProductDocuments/ReferenceManuals/PolarFire_SoC_FPGA_MSS_Technical_Reference_Manual_VC.pdf) for a detailed description of PolarFire SoC.

<a name="overview"></a>

## Overview

The Health Monitoring service is an HSS state machine (running on the E51 monitor hart) that
periodically inspects hardware/software observable values (registers, counters, status words,
etc.) at specified memory addresses and reports anomalies based on developer‑selected rules.
Its goals are to:

- Provide lightweight, periodic checks of system health without blocking application workloads.

- Offer structured triggers (callbacks) when thresholds or change conditions are met.

- Ensure controlled verbosity via throttled log messages, reducing console noise while preserving
  signal.

The service is fully extensible: platform teams supply an array of monitor descriptors defining
*what* to read, *how* to interpret the value, and *when* to raise an alert.

<a name="state-machine-structure"></a>

## State Machine Structure

The Health Monitoring service is implemented as a two-state HSS State Machine:

```ditaa
+-----------------------+            +---------------------+
| HEALTH_INITIALIZATION | --ready--> | HEALTH_MONITORING   |
+-----------------------+            +---------------------+
```

It is registered as a global machine named `healthmon_service` and runs with debug enabled.

### States

#### 1. Initialization (`HEALTH_INITIALIZATION`)

- **Purpose:** Defer monitoring until the platform is ready.
- **Transition condition:** Advances to `HEALTH_MONITORING` only after the following HSS events are
  notified:
  - `EVENT_DDR_TRAINED`
  - `EVENT_STARTUP_COMPLETE`
  - `EVENT_POST_BOOT`

This ensures monitoring starts with memory and subsystems properly initialized.

#### 2. Monitoring (`HEALTH_MONITORING`)

- **Purpose:** Iterate over all configured monitors, evaluate their condition, and react.
- **Actions:** For each monitor:
  1. Apply rate limiting (throttle) to avoid spamming logs.
  2. Read a `uint32_t` value from the monitor’s `pAddr` (volatile read).
  3. Optionally apply right‑shift and bitmask.
  4. Evaluate the selected check type.
  5. If triggered (and the value changed since the previous trigger), log a highlighted message,
     invoke the monitor’s callback (if any), and update status (count, timestamps, last value).

<a name="key-components"></a>

## Key Components

<a name="monitor-definition-struct-healthmonitor"></a>

### Monitor Definition (`struct HealthMonitor`)

Each monitor is defined with the following fields:

```c
struct HealthMonitor {
    char const * const pName;                 // Human-readable monitor name
    uintptr_t          pAddr;                 // Address to read (uint32_t volatile *)
    enum HealthMon_CheckType checkType;       // Condition type (see Check Types)
    uint32_t           maxValue;              // Upper bound or comparison value
    uint32_t           minValue;              // Lower bound when needed
    uint8_t            shift;                 // Right shift applied before mask
    uint64_t           mask;                  // Bitmask applied after shift
    void (*triggerCallback)(uintptr_t pAddr); // Optional reaction on trigger
    uint32_t           throttleScale;         // Scale factor * 1s for log throttling
};
```

> **Note:** A weak, empty set is provided in `healthmon_monitors_weak.c`. BSPs can *override* this
with a strong definition supplying their platform‑specific monitors.

<a name="runtime-status-struct-healthmonitor-status"></a>

### Runtime Status (`struct HealthMonitor_Status`)

The service keeps per‑monitor runtime state:

```c
struct HealthMonitor_Status {
    HSSTicks_t throttle_startTime; // Last time a message was emitted
    uint32_t   lastValue;          // Last value reported when triggered
    size_t     count;              // Number of times this monitor triggered
    bool       initialized;        // Whether we've captured the first sample
};
```

These fields drive the throttle logic and change detection.

<a name="monitoring-logic"></a>

## Monitoring Logic

For each monitor, the service performs:

1. **Throttle check:** `HSS_Timer_IsElapsed(status.throttle_startTime, throttleScale * ONE_SEC)`
   If not elapsed, the monitor is temporarily skipped to rate‑limit console output.

2. **Sample read:** `value = *(uint32_t volatile *)(pAddr)`
   The volatile read prevents the compiler from eliding the access.

3. **Transform:** if `shift` is non‑zero, apply `value >>= shift`; if `mask` is non‑zero, apply
   `value &= mask`.

4. **Condition evaluation:** Based on `checkType` (below).

5. **Triggering:** If the condition evaluates true *and* `value != status.lastValue`, the service:

- Increments `status.count`.
- Logs a highlighted message including the monitor name, check type, expected value(s), and the observed value.
- Invokes `triggerCallback(pAddr)` if non‑NULL.
- Updates `status.throttle_startTime = HSS_GetTime()` and `status.lastValue = value`.
- Marks `initialized = true` if using change‑since‑last semantics.

### Check Types

The service supports the following condition primitives:

- `ABOVE_THRESHOLD` - Trigger when `value > maxValue`.
- `BELOW_THRESHOLD` - Trigger when `value < minValue`.
- `ABOVE_OR_BELOW_THRESHOLD` - Trigger when outside `[minValue, maxValue]`.
  The service refines the message to “above” or “below” depending on which side breached.
- `EQUAL_TO_VALUE` - Trigger when `value == maxValue`.
- `NOT_EQUAL_TO_VALUE` - Trigger when `value != maxValue`.
- `CHANGED_SINCE_LAST` - Trigger when a subsequent read differs from the previous sample (first
  sample sets `initialized` and does not trigger).

A fixed table of human‑readable names is used in logs (e.g., "above threshold", "changed since last
read").

<a name="shift-and-mask"></a>

### Shift and Mask

Monitors may target bitfields within a register. The service first applies a right shift, then
a bitmask:

```c
if (monitor.shift) value >>= monitor.shift;
if (monitor.mask)  value &= monitor.mask;
```

This allows checks against specific subfields without additional code.

<a name="throttle--rate-limiting"></a>

### Throttle / Rate Limiting

To minimize console spam, each monitor can set `throttleScale` (seconds). The service only emits
a new message for that monitor once the throttle interval has elapsed since the last trigger.
This applies *per monitor*, and is independent of other monitors.

<a name="trigger-callbacks"></a>

### Trigger Callbacks

Each monitor can supply a `triggerCallback(uintptr_t pAddr)` invoked after logging a trigger.
A default no‑op (`healthmon_nop_trigger`) is provided; projects can implement callbacks to:

- Notify other subsystems,
- Toggle indicators,
- Collect extended diagnostics,
- Initiate recovery actions.

> **Guardrails:** Callbacks should be quick and non‑blocking and avoid heavy work in the monitoring
loop.

<a name="initialization--dependencies"></a>

## Initialization and Dependencies

The service depends on standard HSS facilities:

- **Event gating:** Monitoring begins only after `EVENT_DDR_TRAINED`, `EVENT_STARTUP_COMPLETE`, and
  `EVENT_POST_BOOT` are notified (platform readiness).
- **Timekeeping:** `HSS_GetTime()` and `HSS_Timer_IsElapsed()` provide throttle timing.
- **Debug/logging:** `mHSS_DEBUG_PRINTF(...)` and highlight helpers indicate alerts with `LOG_ERROR`
  emphasis, then restore normal logging.

The service is compiled as part of HSS and registered under the name `healthmon_service`.

<a name="configuration--extensibility"></a>

## Configuration and Extensibility

- **Defining monitors:** Provide a strong definition of:

  ```c
  const struct HealthMonitor monitors[] = {
      // Example:
      { "DDR ECC Errors", DDR_ECC_REG,
        ABOVE_THRESHOLD, .maxValue = 0, .minValue = 0,
        .shift = 0, .mask = 0xFFFFFFFF,
        .triggerCallback = my_ddr_ecc_handler,
        .throttleScale = 5  // one log at most every 5s
      },
      // ...more monitors...
  };

  struct HealthMonitor_Status monitor_status[ARRAY_SIZE(monitors)] = {0};
  const size_t monitors_array_size = ARRAY_SIZE(monitors);
  ```

  By overriding the weak declarations in `healthmon_monitors_weak.c`, platforms tailor what is
  observed.

- **Addressing and Safety:** Ensure `pAddr` points to a valid, readable location mapped for the E51;
  use appropriate barriers if required for device registers.
- **Value semantics:** Select `shift`/`mask` and `minValue`/`maxValue` consistent with the register
  layout and units.

<a name="debugging--statistics"></a>

## Debugging and Statistics

The service exposes a convenience function:

```c
void HSS_Health_DumpStats(void);
```

It prints the following to the HSS console:

- `monitors_array_size`, a count of the number of active monitors.
- A line per monitor showing its name, condition, comparison value(s) (when applicable), and the
  trigger count to date.

Formatting is handled with `sbi_snprintf` and HSS debug macros, producing aligned output suitable
for console review.

<a name="flow-summary"></a>

## Flow Summary

### 1. Initialization

- Wait for readiness events (`DDR trained`, `startup complete`, `post‑boot`).

### 2. Monitoring

- For `i = 0 .. monitors_array_size-1`:
  - If throttle interval elapsed:
    - Read `uint32_t` at `pAddr` (volatile).
    - Apply `shift`/`mask` transforms.
    - Evaluate `checkType`.
    - If triggered and `value` changed:
      - Increment count.
      - Emit highlighted log with name, condition, expected value(s), and observed value.
      - Invoke `triggerCallback` (if any).
      - Update `throttle_startTime` and `lastValue`.

## Guidance and Best Practices

- Choose sensible throttle scales. Start with 1–5 seconds; increase for noisy signals.
- Use `CHANGED_SINCE_LAST` for indicators that are expected to vary but only warrant attention on
  transitions.
- Prefer specific bitfield checks via `shift`/`mask` to avoid false positives from unrelated bits.
- Keep callbacks lightweight. Heavy processing or blocking I/O in the monitoring loop can stall
  the service / superloop.
- Validate addresses. Ensure each `pAddr` is readable by the E51 and reflects up‑to‑date device state.
- Instrument early. Add monitors during bring‑up to catch configuration regressions (clocking, ECC,
  link status).

<a name="minimal-example-of-bsp-specific-overrides"></a>

### Example of BSP-specific Overrides

```c
// Example monitor for a hypothetical temperature register, in
//     boards/my-platform/healthmon_overrides.c
// and added as a source file into
//     boards/my-platform/Makefile
//
#define SOC_TEMP_REG   0x20100040u

static void temp_alert(uintptr_t addr) {
    (void)addr;
    // e.g., notify thermal manager or toggle an LED
}

const struct HealthMonitor monitors[] = {
    { "SoC Temperature (deg C)",
      SOC_TEMP_REG,
      ABOVE_THRESHOLD,
      .maxValue = 85,              // alert above 85 degC
      .minValue = 0,
      .shift = 0,
      .mask  = 0xFF,               // 8-bit field
      .triggerCallback = temp_alert,
      .throttleScale   = 10        // at most one alert every 10s
    },
};

struct HealthMonitor_Status monitor_status[ARRAY_SIZE(monitors)] = {0};
const size_t monitors_array_size = ARRAY_SIZE(monitors);
```

<a name="conclusion"></a>

## Conclusion

The HSS Health Monitoring service provides a modular, non‑intrusive framework for continuously
assessing platform health. With simple descriptors and built‑in throttling, developers can quickly
add targeted checks and trigger actions when conditions are met - improving observability and
resilience across PolarFire SoC deployments.
