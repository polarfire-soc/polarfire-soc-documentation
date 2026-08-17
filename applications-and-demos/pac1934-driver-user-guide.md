# PAC1934 Bare Metal Driver User Guide

## Introduction

PAC1934 is a four-channel power monitor that measures bus voltage and the
voltage developed across a sense resistor. The PAC1934 driver in the PolarFire®
SoC platform repository uses the MSS I2C driver to read the monitor and prints
converted voltage and current measurements through an MSS MMUART instance.

The primary example for this driver is the MPFS HAL power-saving demo in the
[PolarFire® SoC Bare Metal Examples repository](https://mi-v-ecosystem.github.io/redirects/repo-polarfire-soc-bare-metal-examples),
at `driver-examples/mss/mpfs-hal/mpfs-hal-power-saving-demo`. The demo uses
PAC1934 measurements to compare the board current before and after applying
PolarFire SoC power-saving features.

The driver source is in the
[PolarFire® SoC platform repository](https://mi-v-ecosystem.github.io/redirects/repo-platform)
at the following path:

```text
drivers/off_chip/pac1934/
|-- pac1934.c
|-- pac1934.h
`-- pac1934_regs.h
```

## Supported Hardware

PAC1934 driver supports Icicle Kit and Video Kit boards. PAC1934 accesses 7-bit
I2C address `0x10` through the low MSS I2C1 instance, `g_mss_i2c1_lo`. The four
channels at address `0x10` monitor the following rails:

| Channel | Driver output | Monitored rail |
|:--|:--|:--|
| 1 | `VDD_REG` / `IVDD_REG` | VDD core supply |
| 2 | `VDDA25` / `IVDDA25` | 2.5V analog supply |
| 3 | `VDD25` / `IVDD25` | 2.5V supply |
| 4 | `VDDA_REG` / `IVDDA_REG` | VDDA analog supply |

PAC1934 assumes a 32V full-scale bus voltage and a 10A full-scale current. The
current calculation corresponds to the 10 mΩ sense resistors used on the
supported kits. When porting the driver to another board, verify the I2C
instance, device address, channel-to-rail mapping and sense resistor values.

For board-level details, see the
[Icicle Kit schematics](https://ww1.microchip.com/downloads/aemDocuments/documents/FPGA/ProductDocuments/ReferenceManuals/mpfs-icicle-kit-es-schematics.pdf)
and the
[Video Kit schematics](https://ww1.microchip.com/downloads/aemDocuments/documents/FPGA/ProductDocuments/ReferenceManuals/PolarFire_SoC_SEV_KIT_Schematics.pdf).
For device details, see the
[PAC1934 product page and data sheet](https://www.microchip.com/en-us/product/PAC1934).

## Hardware and Software Dependencies

Before using the driver, ensure that:

- MSS I2C1 is enabled and routed to PAC1934. The base designs for the
  Icicle Kit and Video Kit route MSS I2C1 to the monitor. Designs that repurpose
  I2C1 for another interface make PAC1934 inaccessible.
- The application enables the clock and releases reset for MSS I2C1.
- The application supplies and initializes an MSS MMUART instance for output.
- The PAC1934 source files and the MSS I2C, MSS MMUART, MPFS HAL and PLIC
  sources are included in the build.
- Interrupts and the MPFS HAL system tick are configured on U54_1, which owns
  the driver in the power-saving demo. The system tick services the MSS I2C
  transaction timeout.

The driver declares the following application-owned symbol:

```c
extern mss_uart_instance_t *g_uart;
```

Define `g_uart` once in the application and point it to an initialized MMUART
instance. The driver sends measurements and error messages to this instance.

The driver uses global state, static transaction buffers and one fixed I2C
instance. Use it from one hart and do not make concurrent PAC1934 calls. The
driver also initializes PLIC and enables the I2C1 main, alert and suspend
interrupts during the probe, so call the probe before configuring other PLIC
users that must retain their settings.

The supplied `SysTick_Handler_h1_IRQHandler()` services `g_mss_i2c1_lo` for
U54_1. If another hart owns the driver, update that hart's system tick handler
to call `MSS_I2C_system_tick()` for `g_mss_i2c1_lo`. Do not add another strong
definition of a system tick handler already defined by the driver.

## Driver Operation

The normal call sequence is:

1. Enable and configure the application MMUART and MSS I2C1 clocks.
2. Configure the system tick on U54_1, or adapt the timeout handler when another
   hart owns the driver.
3. Call `pac1934_sensor_probe()`.
4. If the probe succeeds, call `pac1934_draw_vb()` and
   `pac1934_draw_isense()` as required.

### Probe and Initialization

`pac1934_sensor_probe()` performs the following operations:

- Enables the FIC3 clock and fabric access path.
- Initializes the PLIC and enables the MSS I2C1 interrupts.
- Initializes `g_mss_i2c1_lo` as a master with serial address `0x21` and clock
  divider `MSS_I2C_PCLK_DIV_192`.
- Reads the PAC1934 product, manufacturer and revision registers.
- Marks the sensor available only when the returned identifiers match the
  expected PAC1934 values.

The function returns `0` when the probe succeeds and `-1` when it fails. Always
check this return value before requesting measurements.

### Voltage Measurement

`pac1934_draw_vb()` sends a refresh command, reads the four averaged VBUS
registers, converts each unsigned 16-bit result to volts and prints the result
through `g_uart`. The conversion is:

```text
voltage (V) = raw VBUS average * 32 / 65535
```

### Current Measurement

`pac1934_draw_isense()` sends a refresh command, reads the four averaged VSENSE
registers, converts each unsigned 16-bit result to milliamperes and prints the
result through `g_uart`. The conversion is:

```text
current (mA) = raw VSENSE average * 10000 / 65535
```

This conversion is specific to the 10 mΩ sense resistors on the supported kits.
Update the scaling factor before using a different sense resistance.

The public API prints measurements directly. It does not return raw samples,
calculated power, or accumulated energy to the application.

### Error Handling

I2C transactions use a 3000 ms software timeout. If any channel read fails, the
driver:

- stops the current measurement operation;
- marks the sensor unavailable; and
- prints `PAC1934 I2C read failed` once to avoid repeatedly flooding UART.

Subsequent draw calls return without accessing the sensor. After correcting the
I2C routing or hardware condition, call `pac1934_sensor_probe()` again. A
successful probe restores measurements and clears the one-time error state.

## Public API

Include the driver header as follows:

```c
#include "drivers/off_chip/pac1934/pac1934.h"
```

| Function | Description | Return value |
|:--|:--|:--|
| `pac1934_sensor_probe()` | Initializes I2C and verifies the PAC1934 identity | `0` on success, `-1` on failure |
| `pac1934_draw_vb()` | Reads and prints the four averaged bus voltages | None |
| `pac1934_draw_isense()` | Reads and prints the four averaged currents | None |

## Integration Example

The following excerpt shows the required initialization when the driver is
owned by U54_1 and prints through MMUART1. Adapt the peripheral ownership and
UART selection to the application and MSS configuration.

```c
#include "mpfs_hal/mss_hal.h"
#include "drivers/mss/mss_mmuart/mss_uart.h"
#include "drivers/off_chip/pac1934/pac1934.h"

mss_uart_instance_t *g_uart = &g_mss_uart1_lo;

void u54_1(void)
{
    (void)mss_config_clk_rst(MSS_PERIPH_MMUART1, 1U, PERIPHERAL_ON);
    (void)mss_config_clk_rst(MSS_PERIPH_I2C1, 1U, PERIPHERAL_ON);

    MSS_UART_init(g_uart,
                  MSS_UART_115200_BAUD,
                  MSS_UART_DATA_8_BITS |
                  MSS_UART_NO_PARITY |
                  MSS_UART_ONE_STOP_BIT);

    (void)SysTick_Config();

    if (0 == pac1934_sensor_probe())
    {
        pac1934_draw_vb();
        pac1934_draw_isense();
    }
    else
    {
        MSS_UART_polled_tx_string(
            g_uart,
            (const uint8_t *)"PAC1934 probe failed\r\n");
    }

    while (1U)
    {
        ;
    }
}
```

The calls are synchronous and include a sensor refresh delay. Call them from
normal application context rather than an interrupt handler or a
latency-sensitive section.

## Running the Power-Saving Demo

The power-saving demo provides a ready-to-use integration and is the recommended
starting point:

```text
polarfire-soc-bare-metal-examples/
`-- driver-examples/mss/mpfs-hal/mpfs-hal-power-saving-demo/
```

To run it:

1. Import `mpfs-hal-power-saving-demo` into SoftConsole.
2. Select `Icicle-Kit-eNVM-Release` or `Video-Kit-eNVM-Release`, as appropriate
   for the target board and build the project.
3. Connect a serial terminal to the project's UART0 interface using 115200 baud,
   8 data bits, 1 stop bit, no parity and no flow control.
4. Run `mpfs-hal-power-saving-demo hw all-harts debug.launch`.
5. At the power-saving menu, enter `c` to display the PAC1934 voltage and
   current values.
6. Apply a power-saving option, return to a menu and enter `c` again to compare
   the measurements.

In the demo, the E51 menu sets a monitor request and U54_1 performs the PAC1934
transactions. A successful request prints four voltage readings followed by
four current readings for the rails listed in [Supported hardware](#supported-hardware).

For a complete build configuration, launch instructions, supported boards and
power-saving menu descriptions, refer to `README.md` in the demo directory of
the [PolarFire® SoC Bare Metal Examples repository](https://mi-v-ecosystem.github.io/redirects/repo-polarfire-soc-bare-metal-examples).

## Troubleshooting

### The Probe Returns `-1`

- Confirm that the board is powered and that PAC1934 is accessible at
  address `0x10`.
- Confirm that MSS I2C1 is enabled and routed to the board monitor.
- Confirm that the application enabled the MSS I2C1 clock for the hart using
  the driver.
- Check that another I2C master does not control the same bus.
- Use the base design for the Icicle Kit or Video Kit. Confirm that a customized
  design has not routed MSS I2C1 away from PAC1934.

### `PAC1934 I2C read failed` is printed

A transaction failed after a successful probe. Correct the I2C or hardware
condition and call `pac1934_sensor_probe()` again before requesting another
measurement.

### No Measurement Text Appears

- Verify that `g_uart` is globally defined and points to the initialized UART.
- Verify that the serial terminal uses the same UART and the 115200 8-N-1
  settings used by the application.
- Check the return value from `pac1934_sensor_probe()`.

### Current Values are Incorrect on a Custom Board

The supplied conversion assumes a 10 mΩ sense resistor. Change the
current scaling for the sense resistor fitted to the custom board and update
the displayed channel names to match its rail connections.
