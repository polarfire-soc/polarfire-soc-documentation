# PolarFire SoC Boot Modes Fundamentals

This section describes the 4 boot modes of the PolarFire SoC Coreplex. These boot modes control the start-up of the Coreplex harts under the System Controller's control.

- [Boot mode 0](./boot-mode-0/boot-mode-0-fundamentals.md) is used by blank devices or when debugging embedded software where code should not be executed on power on
- [Boot mode 1](./boot-mode-1/boot-mode-1-fundamentals.md) is used where the Microprocessor Subsystem (MSS) harts start executing non-secured code from eNVM on power up
- [Boot mode 2](./boot-mode-2/boot-mode-2-fundamentals.md) is intended for implementing user-defined secure boot authentication
- [Boot mode 3](./boot-mode-3/boot-mode-3-fundamentals.md) implements Microchip supplied factory secure boot authentication of the eNVM content
