# Asymmetric Multiprocessing (AMP)

This page provides a brief introduction to Asymmetric Multiprocessing (AMP) concepts, including instructions on how to run multiple operating systems or bare metal applications simultaneously on PolarFire SoC.

- [Asymmetric Multiprocessing (AMP)](#asymmetric-multiprocessing-amp)
  - [Introduction](#introduction)
  - [AMP on PolarFire SoC](#amp-on-polarfire-soc)
    - [AMP Boot Flow](#amp-boot-flow)
      - [AMP Early Boot Flow](#amp-early-boot-flow)
      - [AMP Late Boot Flow](#amp-late-boot-flow)
    - [Inter-Hart Communication (IHC)](#inter-hart-communication-ihc)
  - [AMP Configurations](#amp-configurations)
    - [Linux + FreeRTOS/Bare Metal(BM) Configuration](#linux--freertos--baremetal-configuration)
    - [Building the Linux + FreeRTOS/BM Demo](#building-the-linux--freertos--bm-demo)
      - [Build Linux + FreeRTOS/BM AMP Demo using Yocto](#build-linux--freertos--bm-amp-demo-using-yocto)
      - [Build Linux + FreeRTOS/BM AMP Demo using Buildroot](#build-linux--freertos--bm-amp-demo-using-buildroot)
    - [Running the Linux + FreeRTOS/BM AMP Demo on the Icicle Kit](#running-the-linux--freertos--bm-amp-demo-on-the-icicle-kit)
    - [RTOS/BM + RTOS/BM AMP Configuration](#rtosbm--rtosbm-amp-configuration)

<a name="introduction"></a>

## Introduction

Asymmetric Multiprocessing (AMP) is a type of multi-core software architecture that allows multiple operating systems or software contexts to run simultaneously and independently of each other.

In an AMP system, it is possible to allocate hardware resources to a specific software context. These hardware resources include processor cores, peripherals and physical memory regions.

<a name="amp-on-polarfire-soc"></a>

## AMP on PolarFire SoC

PolarFire SoC contains a CPU Core Complex with a single E51 core and four U54 application cores, where each of the cores has one hardware thread or hart.

PolarFire SoC can be configured to run up to two independent software contexts. Each software context can have its own operating system, memory regions and hardware resources assigned.

The supported PolarFire SoC AMP software architecture is described below:

- 1x E51 monitor core which is dedicated to running the Hart Software Services (HSS)

- 4x U54 application cores which can be distributed between two independent software contexts

The [Hart Software Services](https://mi-v-ecosystem.github.io/redirects/repo-hart-software-services) (HSS) is a superloop monitor software running on the E51 processor. One of the many functions of the HSS is to act as a first-stage bootloader as part of the boot service. Please refer to the  [AMP boot flow](#amp-boot-flow) section for further information on the boot process.

<a name="amp-boot-flow"></a>

### AMP Boot Flow

The hart software services (HSS) makes use of a HSS payload as part of the boot service.

A HSS payload is an image containing a header and one or more binary files that have been merged together to form the payload.

In an AMP configuration, the HSS payload should contain two binary files, one for each of the supported software contexts. For example, one of these binary files could be an ELF file containing a bare metal application or RTOS, and the other one a U-boot binary to load Linux OS.

<a name="amp-early-boot-flow"></a>

### AMP Early Boot Flow

By default, the HSS will copy the payload from non-volatile storage to DDR and then copy the binaries to the memory location(s) that were specified when the payload was generated.

The HSS payload header describes which harts are associated with each payload as well as the address that each hart should start executing from once the payload has been loaded into memory by the HSS.

![amp-early-boot-flow](./images/amp/amp-early-boot-flow.png)

For further information on [HSS payloads](https://mi-v-ecosystem.github.io/redirects/software-development_hss-payloads), please refer to the HSS payload documentation page.

<a name="amp-late-boot-flow"></a>

### AMP Late Attach Boot Flow

This mode can be used to control the Life Cycle Management (LCM) of the remote AMP context from the master context using the Remoteproc framework. This allows:

- Controlling the remote processor execution (start, stop, etc.)
- Loading the ELF firmware in the remote processor memory
- Parsing a firmware resource table to set associated resources such as inter-hart communication using RPMsg framework.

It is worth mentioning that Remoteproc functionality is currently supported by Linux (master context) to control the life cycle of the remote context (Bare Metal/FreeRTOS).

![amp-late-boot-flow](./images/amp/amp-late-boot-flow.png)

In this boot flow, the HSS payload generator YAML configuration file has the skip_autoboot flag set to true in the RTOS/BM entry.
This will indicate the HSS to do the basic setup of the AMP context (OpenSBI domain registration, PMP setup, etc.). However, it won't load the ELF file to memory or start the application execution. The harts within the context will remain idle until the application (either a FreeRTOS or BM application) gets loaded and started by another entity, such as remoteproc from a Linux master context.

More details on this framework are available in the remoteproc framework documentation [page](https://mi-v-ecosystem.github.io/redirects/asymmetric-multiprocessing_remoteproc).

<a name="inter-hart-communication-ihc"></a>

### Inter-Hart Communication (IHC)

Some AMP applications may require software contexts to be able to communicate and send messages between them. For example, a RTOS or bare metal-based application communicating with a Linux context, or two RTOS or bare metal applications sending messages between them.

PolarFire SoC supports the Remote Processor Messaging (RPMsg) framework, which allows a 'master' software context to communicate with a 'remote' software context using a built-in API.

For more information on RPMsg protocol, please refer to the [PolarFire SoC RPMsg documentation](rpmsg.md) page.

<a name="amp-configurations"></a>

## AMP Configurations

PolarFire SoC supports several AMP configurations including Linux + RTOS, Linux + Bare Metal, RTOS + Bare Metal or Bare Metal + Bare Metal.

Each of the configurations described above have different use cases and should be chosen based on application requirements such as real-time response, security and safety requirements.

One of the most common approaches is to use a general purpose operating system such as Linux with a real-time operating system (RTOS) in order to have real-time constraints applications handled by the RTOS while having connectivity or UI applications running from the Linux context.

<a name="linux--freertos--baremetal-configuration"></a>

### Linux + FreeRTOS/Bare Metal Configuration

The Linux and FreeRTOS/Bare Metal AMP configuration consists of a Linux OS running in one context and a FreeRTOS application running on the second context, or running Linux in one context and a bare metal application on the second context.

The PolarFire SoC [Yocto](https://mi-v-ecosystem.github.io/redirects/repo-meta-polarfire-soc-yocto-bsp) and [Buildroot](https://mi-v-ecosystem.github.io/redirects/repo-polarfire-soc-buildroot-sdk) build environments provide an Icicle Kit AMP machine which can be used to build a Linux + FreeRTOS/BM AMP configuration demo.

The demo runs Linux on harts 1-2-3 and a FreeRTOS or Bare metal AMP application on hart 4. The demo contains several applications to send/receive messages between Linux and FreeRTOS using the RPMsg framework.

The image below shows a diagram of the Linux and FreeRTOS/BM AMP configuration demo:

![linux_freertos_bm_config](./images/amp/linux-freertos-bm-amp.png)

The table below describes the hardware resources assignment used in this demo:

|          | Linux (Context A)   | FreeRTOS/BM (Context B) |
|----------|---------------------|----------------------|
| Harts    | U54_1, U54_2, U54_3 | U54_4                |
| PDMA     | ✓                   | -                    |
| RTC      | ✓                   | -                    |
| USB      | ✓                   | -                    |
| eMMC/SD  | ✓                   | -                    |
| Serial   | MMUART 1-2-4        | MMUART 3             |
| Ethernet | ✓                   | -                    |
| PCIE     | ✓                   | -                    |
| GPIO_2   | ✓                   |                      |
| I2C1     | ✓                   | -                    |
| LSRAM    | ✓                   | -                    |
| DMA(FIC) | ✓                   | -                    |
| CAN      | ✓                   | -                    |

The table below describes the DDR memory layout used in this demo:

|                             | Linux (Context A)                                            | FreeRTOS/BM (Context B)          |
| --------------------------- | ------------------------------------------------------------ | -------------------------------- |
| Main Memory                 | Cached: <br /><br />  0x8000_0000 (64 MiB), 0x8A00_0000 (128 MiB) and 0x10_2200_0000 (1504 MiB) <br /><br /> Non-cached: <br /><br /> 0xC400_0000 (96 MiB) and 0x14_1200_0000 (256 MiB) | Cached @ 0x91C0_0000 (1 MiB)   |
| User space mappable buffers | Cached @  0x8800_0000 (32 MiB) <br /><br /> Non-cached @ 0xC800_0000 (32MiB) <br /><br /> WCB  @ 0xD800_0000 (32MiB) | -                                |
| RPMsg vrings                | Cached @ 0x91D0_0000 (64 KiB)                              | Cached @ 0x91D0_0000 (64 KiB)  |
| RPMsg buffers               | Cached @ 0x91D1_0000 (256 KiB)                             | Cached @ 0x91D1_0000 (256 KiB) |
| Remoteproc resource table   | Cached @ 0x91D5_0000 (4 KiB)                               | Cached @ 0x91D5_0000 (4 KiB)   |

<a name="building-the-linux--freertos--bm-demo"></a>

### Building the Linux + FreeRTOS/Bare Metal Demo

Pre-requisites: Before following the steps described in this section, make sure you have the latest [Yocto](https://mi-v-ecosystem.github.io/redirects/repo-meta-polarfire-soc-yocto-bsp) or [Buildroot](https://mi-v-ecosystem.github.io/redirects/repo-polarfire-soc-buildroot-sdk) build systems configured and setup in your system. Please refer to the README of each build system for further information.

<a name="build-linux--freertos--bm-amp-demo-using-yocto"></a>
#### Build Linux + FreeRTOS/Bare Metal AMP Demo using Yocto

1. Open the `icicle-kit-es-amp.conf` machine configuration file located in the `conf` directory. Make sure that the `AMP_DEMO` variable is set to `freertos` for a Linux+FreeRTOS demo or `bm` for a Linux+Bare Metal demo:

    ```bash
    ## Set this to "freertos" for a Linux + FreeRTOS demo or "bm" for a Linux + Bare Metal demo
    AMP_DEMO = "freertos"
    ##AMP_DEMO = "bm"
    ```

2. Use the Yocto bitbake command and set the icicle-kit-es-amp MACHINE and image required:

    ```bash
    MACHINE=icicle-kit-es-amp bitbake mpfs-dev-cli
    ```

3. Copy the created Disk Image to a flash device (USB mmc flash/SD/uSD)

> Be very careful while picking /dev/sdX device! Look at dmesg, lsblk, GNOME Disks, etc. before and after plugging in your usb flash device/uSD/SD to find a proper device. Double check it to avoid overwriting any system disks/partitions!

```bash
cd yocto-dev/build
zcat tmp-glibc/deploy/images/icicle-kit-es-amp/mpfs-dev-cli-icicle-kit-es-amp.wic.gz | sudo dd of=/dev/sdX bs=4096 iflag=fullblock oflag=direct conv=fsync status=progress
```

The disk image flashed to the device in the step above contains the following partitions:

- Boot partition containing a fitImage (Linux kernel + Device Tree Blob) used by context A
- HSS boot partition with a HSS payload containing the following binaries:
  - U-boot binary used by context A
  - FreeRTOS or Bare metal ELF file used by context B
- Linux Root Filesystem for context A

<a name="build-linux--freertos--bm-amp-demo-using-buildroot"></a>

#### Build Linux + FreeRTOS/Bare Metal AMP Demo using Buildroot

1. Build the image using the icicle_amp_defconfig and set the `BR2_PACKAGE_MCHP_AMP_CONTEXT_B` variable to `mpfs-rpmsg-freertos` for a Linux+FreeRTOS demo or to `mpfs-rpmsg-bm` for a Linux+Bare Metal demo. This variable is set in the file `configs/icicle_amp_defconfig`.

    Note: If the `BR2_PACKAGE_MCHP_AMP_CONTEXT_B` variable is not set when building in AMP mode, the Linux+FreeRTOS will be set as default.

    Build the image:

    ```bash
    BR2_EXTERNAL=../buildroot-external-microchip/ make icicle_amp_defconfig
    make
    ```

2. Load the image onto the target as shown in the Linux4Microchip Buildroot External [README](https://github.com/linux4microchip/buildroot-external-microchip/blob/master/README.md).

The disk image generated by Buildroot contains the following partitions:

- HSS boot partition with a HSS payload containing the following binaries:
  - U-boot binary used by context A
  - FreeRTOS or Bare Metal ELF file used by context B
- Boot partition containing a fitImage (Linux kernel + Device Tree Blob(s)) used by context A
- A root partition containing an Initramfs (cpio archive)

<a name="running-the-linux--freertos--bm-amp-demo-on-the-icicle-kit"></a>

### Running the Linux + FreeRTOS/BM AMP Demo on the Icicle Kit

On connecting Icicle kit J11 to the host PC, you should see 4 COM port interfaces. To use this project, configure the COM port interface 1 and 3 as below:

- 115200 baud
- 8 data bits
- 1 stop bit
- no parity

On startup, the Linux console will show messages on COM port interface 1 and the FreeRTOS or Bare Metal application will display a menu on COM port interface 3.

The menu displayed on the FreeRTOS/BM COM port allows the user to run several RPMsg applications included as part of the demo.

For more information on how to use the RPMsg applications, please refer to the [RPMsg documentation](rpmsg.md) page.

<a name="rtosbm--rtosbm-amp-configuration"></a>

### RTOS/BM + RTOS/BM AMP Configuration

The HSS payload generator can be used to create custom AMP configurations including RTOS + RTOS, RTOS + Bare metal or Bare metal + Bare metal.

The `mpfs-rpmsg-freertos` and `mpfs-rpmsg-bm` SoftConsole projects included within the [PolarFire SoC AMP examples](https://mi-v-ecosystem.github.io/redirects/repo-polarfire-soc-amp-examples) repository allow a FreeRTOS + BM, FreeRTOS + FreeRTOS or Bare Metal + Bare Metal AMP configuration demo to be built with RPMsg communication.

For more information on how to build these demos, please refer to the [RPMsg on FreeRTOS/Bare Metal](rpmsg.md#rpmsg-rtos-intro) section in the PolarFire SoC RPMsg documentation page.
