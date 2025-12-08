# PolarFire SoC: Adding a New Customer-Specific Service to HSS

- [Overview](#overview)
  - [Services as Independent State Machines](#services-as-independent-state-machines)

- [Service Directory Structure](#service-directory-structure)

- [Service Implementation](#service-implementation)
  - [State Machine Definition](#state-machine-definition)
  - [State Handler](#state-handlers)
  - [Service Registration](#service-registration)

- [Build System Integration](#build-system-integration)
  - [Makefile Changes](#makefile-changes)
  - [Kconfig Changes](#kconfig-changes)

- [Global Registration](#global-registration)
  - [application/hart0/hss_registry.c](#application_hart0_hss_registry_c)
  - [application/hart0/hss_state_machine.c](#application_hart0_hss_state_machine_c)

- [Using Triggers in a State Machine](#using-triggers-in-a-state-machine)
  - [Available Trigger Events](#available-trigger-events)
  - [Trigger API Functions](#trigger-api-functions)
  - [Typical Trigger Usage Pattern in a State Machine](#typical-trigger-usage-pattern-in-a-state-machine)
  - [Trigger Best Practices](#trigger-best-practices)
  - [Trigger Example: Coordinating Service Startup](#trigger-example--coordinating-service-startup)

- [Service Testing and Debugging](#service-testing-and-debugging)

- [Summary](#summary)

This document provides a brief overview of adding a new customer-specific service to the HSS.

Please refer to the [PolarFire SoC Microprocessor Subsystem (MSS) User Guide](https://ww1.microchip.com/downloads/aemDocuments/documents/FPGA/ProductDocuments/ReferenceManuals/PolarFire_SoC_FPGA_MSS_Technical_Reference_Manual_VC.pdf) for a detailed description of PolarFire SoC.

<a name="overview"></a>

## Overview

The Hart Software Services (HSS) framework is designed to be modular, allowing new services to
be added as independent state machines. Each service typically consists of its own source
files, a state machine definition, and optional configuration and build system hooks. This
guide walks through the process of adding a new, customer-specific service.

<a name="services-as-independent-state-machines"></a>

### Services as Independent State Machines

The HSS is designed as a bare-metal superloop of independent service state machines, synchronized
via event triggers, with a focus on its suitability for a simple, fault-tolerant monitor processor.

1. Simplicity and Predictability

   Superloop architecture (single-threaded, no preemption) is easy to understand, debug, and verify.
   Each service is a state machine: logic is explicit, transitions are clear, and side effects are
   localized.
   No dynamic memory allocation or complex OS features, reducing the risk of memory leaks,
   fragmentation, or priority inversion.

2. Deterministic Behavior

   The main loop visits each service in a fixed order, making timing and execution paths
   predictable. This is important for fault-tolerant and safety-critical systems.
   There are no concurrency hazards. Since only one service runs at a time, there are no race conditions
   or deadlocks.

3. Loose Coupling via Triggers

   Event triggers (atomic flags) allow services to synchronize without direct dependencies or tight
   coupling.
   Services can be added, removed, or modified with minimal impact on others, supporting modularity
   and maintainability.

4. Resource Efficiency

   Low overhead, as there is no context switching, stacks, or scheduler.
   This makes the HSS Suitable for resource-constrained monitor processor like the E51 when running
   from L2-Scratchpad.

5. Fault Isolation

   It is easy to add watchdogs or health checks for each service.
   State transitions are performed through common code, and the transitions are output to the
   HSS console when the Kconfig option `CONFIG_DEBUG_LOG_STATE_TRANSITIONS` is enabled.

<a name="service-directory-structure"></a>

## Service Directory Structure

Create a new directory under `services/` for your service, e.g., `services/myservice/`. Place
your main service source file(s) here, e.g., `myservice_service.c`, and any headers or support files.

**Example:**

```text
services/
  myservice/
    myservice_service.c
    myservice_service.h
    Makefile
    Kconfig
```

<a name="service-implementation"></a>

## Service Implementation

<a name="state-machine-definition"></a>

### State Machine Definition

Each service is implemented as a state machine. Define your states and handlers in your main
service file, following the pattern:

```c
enum MyServiceStates {
    MYSERVICE_INITIALIZATION,
    MYSERVICE_ACTIVE,
    MYSERVICE_NUM_STATES = MYSERVICE_ACTIVE + 1
};

static const struct StateDesc myservice_state_descs[] = {
    { (const stateType_t)MYSERVICE_INITIALIZATION, "init", NULL, NULL, &myservice_init_handler },
    { (const stateType_t)MYSERVICE_ACTIVE, "active", NULL, NULL, &myservice_active_handler },
};

struct StateMachine myservice_service = {
    .state = (stateType_t)MYSERVICE_INITIALIZATION,
    .prevState = (stateType_t)SM_INVALID_STATE,
    .numStates = (const uint32_t)MYSERVICE_NUM_STATES,
    .pMachineName = "myservice_service",
    .startTime = 0u,
    .lastExecutionTime = 0u,
    .executionCount = 0u,
    .pStateDescs = myservice_state_descs,
    .debugFlag = true,
    .priority = 0u,
    .pInstanceData = NULL
};
```

<a name="state-handlers"></a>

### State Handlers

Implement the handler functions for each state:

```c
static void myservice_init_handler(struct StateMachine * const pMyMachine) {
    // Initialization logic
    if (/* ready to transition */) {
        pMyMachine->state = MYSERVICE_ACTIVE;
    }
}

static void myservice_active_handler(struct StateMachine * const pMyMachine) {
    // Main service logic
}
```

<a name="service-registration"></a>

### Service Registration

Declare your service as an external symbol in a header (e.g., `myservice_service.h`):

```c
extern struct StateMachine myservice_service;
```

<a name="build-system-integration"></a>

## Build System Integration

<a name="makefile-changes"></a>

### Makefile Changes

Add a `Makefile` in your service directory:

```makefile
# services/myservice/Makefile
obj-y += myservice_service.o
```

Then, in `services/Makefile`:

```makefile
include services/myservice/Makefile
```

<a name="kconfig-changes"></a>

### Kconfig Changes

Add a `Kconfig` in your service directory:

```kconfig
config CONFIG_SERVICE_MYSERVICE
    bool "Enable MyService"
    default n
```

Then, in `services/Kconfig`:

```kconfig
source "services/myservice/Kconfig"
```

<a name="global-registration"></a>

## Global Registration

<a name="application_hart0_hss_register_c"></a>

### `application/hart0/hss_registry.c`

Register your service in the global state machine table:

```c
#if IS_ENABLED(CONFIG_SERVICE_MYSERVICE)
#include "myservice_service.h"
#endif

struct StateMachine * const pGlobalStateMachines[] = {
#if IS_ENABLED(CONFIG_SERVICE_MYSERVICE)
    &myservice_service,
#endif
    // ... other services ...
};
```

<a name="application_hart0_hss_state_machine_c"></a>

### `application/hart0/hss_state_machine.c`

No changes are typically required here for new services, as the state machine engine iterates
over the `pGlobalStateMachines` array. However, reviewing this file can be useful to gain a
good understanding of how the state machines are executed in practice.

<a name="using-triggers-in-a-state-machine"></a>

## Using Triggers in a State Machine

The HSS trigger mechanism provides a lightweight way for different services (state machines) to
signal and wait for key events. Triggers are commonly used in state machines to coordinate
transitions, enforce dependencies, and synchronize service startup or shutdown with system-wide
events.

Triggers are implemented as atomic flags, each associated with a specific event (enumerated in
`enum HSS_Event`). Services can *notify* (set), *wait for* (check), or *clear* these triggers
as needed.

<a name="available-trigger-events"></a>

### Available Trigger Events

The following events are defined (see `hss_trigger.h`):

```c
enum HSS_Event {
    EVENT_OPENSBI_INITIALIZED,
    EVENT_IPI_INITIALIZED,
    EVENT_DDR_TRAINED,
    EVENT_STARTUP_COMPLETE,
    EVENT_USBDMSC_REQUESTED,
    EVENT_POST_BOOT,
    EVENT_BOOT_STARTED,
    EVENT_BOOT_COMPLETE,
    EVENT_HART_STATE_CHANGED,
    EVENT_HEALTHMON,
};
```

<a name="trigger-api-functions"></a>

### Trigger API Functions

- `void HSS_Trigger_Notify(enum HSS_Event event);`
  Set (notify) the trigger for the specified event.

- `bool HSS_Trigger_IsNotified(enum HSS_Event event);`
  Check if the trigger for the specified event has been set.

- `void HSS_Trigger_Clear(enum HSS_Event event);`
  Clear (reset) the trigger for the specified event.

<a name="typical-trigger-usage-pattern-in-a-state-machine"></a>

### Typical Trigger Usage Pattern in a State Machine

#### 1. **Waiting for an Event Before Transitioning**

In your state handler, use `HSS_Trigger_IsNotified()` to check if a prerequisite event has
occurred before advancing to the next state.

**Example (from healthmon service):**

```c
static void myservice_init_handler(struct StateMachine * const pMyMachine) {
    if (HSS_Trigger_IsNotified(EVENT_DDR_TRAINED) &&
        HSS_Trigger_IsNotified(EVENT_STARTUP_COMPLETE) &&
        HSS_Trigger_IsNotified(EVENT_POST_BOOT)) {
        pMyMachine->state = MYSERVICE_ACTIVE;
    }
}
```

This ensures that the service only becomes active after DDR training, startup, and post-boot
events have all occurred.

#### 2. **Notifying Other Services of an Event**

When your service completes a significant action, notify other services by setting a trigger:

```c
HSS_Trigger_Notify(EVENT_MY_CUSTOM_EVENT);
```

Other services can then wait for this event using `HSS_Trigger_IsNotified()`.

#### 3. **Clearing a Trigger**

If you need to reset a trigger (e.g., after handling an event), use:

```c
HSS_Trigger_Clear(EVENT_MY_CUSTOM_EVENT);
```

<a name="trigger-best-practices"></a>

### Trigger Best Practices

- Use triggers for cross-service synchronization. Triggers are ideal for signaling readiness, completion,
  or error states between services.
- Check all required triggers before transitioning. In your state machine, only advance when all necessary
  conditions are met.
- Avoid busy-waiting. Structure your state handlers to check triggers once per superloop iteration, not in a
  tight loop.
- Document custom events. If you add new events to `enum HSS_Event`, document their purpose and usage.

<a name="trigger-example--coordinating-service-startup"></a>

### Trigger Example: Coordinating Service Startup

Suppose your service must wait for both DDR training and system startup to complete before proceeding:

```c
static void myservice_init_handler(struct StateMachine * const pMyMachine) {
    if (HSS_Trigger_IsNotified(EVENT_DDR_TRAINED) &&
        HSS_Trigger_IsNotified(EVENT_STARTUP_COMPLETE)) {
        // Safe to proceed
        pMyMachine->state = MYSERVICE_ACTIVE;
    }
}
```

If your service needs to notify others when it is ready:

```c
HSS_Trigger_Notify(EVENT_MY_SERVICE_READY);
```

The HSS trigger mechanism is a simple, robust way to coordinate state transitions and
dependencies between services. Use `HSS_Trigger_Notify()` to signal events,
`HSS_Trigger_IsNotified()` to wait for them, and `HSS_Trigger_Clear()` to reset as needed. This
pattern ensures reliable, maintainable synchronization across the HSS framework.

<a name="service-testing-and-debugging"></a>

## Service Testing and Debugging

- Build and flash the HSS with your new service enabled.
- Use debug prints (`mHSS_DEBUG_PRINTF`) in your handlers to trace state transitions and logic.
- Use the configuration system (menuconfig or Kconfig) to enable/disable your service for
  different builds.

<a name="summary"></a>

## Summary

To add a new customer-specific service to HSS:

1. Create a new service directory.
2. Define your state machine and handlers.
3. Integrate with the build system via Makefile and Kconfig.
4. Register your service in `hss_registry.c`.
5. Add triggers to synchronise state transitions with other services etc. if necessary.
6. Test and debug as needed.

This modular approach will ensure that your service is maintainable and configurable in a
manner consistent with the HSS architecture.
