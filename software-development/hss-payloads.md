# Hart Software Services Payloads

- [HSS payloads](#payload-info)
- [Role of the HSS Payload Generator](#info)
- [Downloading the HSS Payload Generator](#download)
	- [Pre-built executables](#executables)
- [Generating payloads](#payloads)
	- [Generating a payload with a Linux image](#linux-payload)
	- [Configuring a bare metal application to be a payload](#config-bare-metal-payload)
	- [Generating a payload with a bare metal application](#bare-metal-payload)

This document describes:

- What a HSS payload is.
- The function of the HSS Payload Generator in the software flow for PolarFire SoC.
- How to build the HSS Payload Generator.
- How to generate a payload for PolarFire SoC using the HSS Payload Generator.

## HSS payloads <a name="payload-info"></a>

A HSS payload is an image containing a header and one or more binary files that have been merged together to form the payload. An image contains:

- A header (containing a file identifier, data sizes and offsets, image information, hart specific configuration).
- A table of initialized boot chunks (code and data).
- A table of BSS and zero init chunks.
- The data itself broken into chunks.

Payloads are stored in non-volatile off chip memory (e.g., eMMC) and loaded on boot. This allows applications larger than on-chip storage (eNVM) to be executed by the PolarFire SoC MSS. Payloads are typically used to load a large executable from eMMC or SD card to DDR memory. When loading a payload, the HSS will copy the payload from non-volatile storage to DDR and then copy this data to the memory location(s) that were specified when the payload was generated.

## Role of the HSS Payload Generator <a name="info"></a>

The HSS Payload Generator creates a payload that will be copied from non-volatile memory to volatile memory by the HSS on system boot. It merges and compresses binary files and headers describing where the payload should be copied into a single HSS payload binary file.

## Downloading the HSS Payload Generator <a name="download"></a>

The HSS Payload Generator source code is bundled as a tool with the Hart Software Services, it's source code is available from the HSS repository on GitHub [here](https://github.com/polarfire-soc/hart-software-services/tree/master/tools/hss-payload-generator).

### Pre-built executables <a name="executables"></a>

#### Windows

A pre-compiled Windows executable for the HSS Payload Generator can be found [here](#LINK).

#### Linux

A pre-compiled Linux executable for the HSS Payload Generator can be found [here](#LINK).

## Generating payloads <a name="payloads"></a>

The HSS Payload Generator uses a configuration yaml file and command line arguments to configure the payload being built.

To generate a payload the following example command could be used:

	./hss-payload-generator -c test/config.yaml output.bin

- "-c" specifies the configuration file being passed.
-	"test/config.yaml" specifies that in this case the configuration file is in a sub-folder called "test" and called "config.yaml".
- "output.bin" specifies the name of the output payload file.

### Generating a payload with a Linux image <a name="linux-payload"></a>

The HSS Payload Generator is automatically cloned and run as part of the build process when Linux is built using the PolarFire SoC Buildroot SDK or the PolarFire SoC Yocto BSP

### Configuring a bare metal application to be a payload <a name="config-bare-metal-payload"></a>

A YouTube video is available showing the configuration of a bare metal application as a payload:

[![Watch the video](https://img.youtube.com/vi/5Q3GZVD72GQ/0.jpg)](https://www.youtube.com/watch?v=5Q3GZVD72GQ&list=PL9B4edd-p2ahGOmnvJvFLvSID3N3-rJC6&index=2)

The main steps involved are:
- In the "mss_sw_config.h" file:
	1. Ensure the "MPFS_HAL_FIRST_HART" define is set to a value greater than 0:

		- 0: Sets the E51 as the main hart of the system - as this is a payload the E51 will be running the HSS.
		- 1: Sets U54_1 as the main hart of the system.
		- 2: Sets U54_2 as the main hart of the system.
		- 3: Sets U54_3 as the main hart of the system.
		- 4: Sets U54_4 as the main hart of the system.

	2. Disable the "MPFS_HAL_HW_CONFIG" define - this define enables the configuration of MSS I/O MUXs and DDR training in the HAL, as this is a payload this configuration will already have been completed by the HSS.

- Configure linker scripts appropriately - the default linker script should be adjusted if a bare metal application is going to target the LIM. This is due to the fact that that the HSS uses a region of the LIM starting at 0x8000000 and the default bare metal linker scripts also target 0x8000000. To solve this, the map file for the HSS can be consulted to determine the end address of the region used by the HSS. In practice a LIM start address of 0x8030000 for the bare metal application leaves sufficient space for the HSS.

### Generating a payload with a bare metal application <a name="bare-metal-payload"></a>

Once the application has been built, the resulting ELF file can be used to generate the HSS payload. An example configuration yaml file is shown below:

	set-name: 'PolarFire-SoC-HSS::bare-metal-image'
	hart-entry-points: {u54_1: '0x8030000', u54_2: '0x8030000', u54_3: '0x8030000', u54_4: '0x8030000'}
	payloads:
		bare-metal-app.elf: {exec-addr: '0x8030000', owner-hart: u54_1, secondary-hart: u54_2, secondary-hart: u54_3, secondary-hart: u54_4 priv-mode: prv_m}

- **set-name**: this command is used to define a name for the payload image.
- **hart-entry-points**: this is the address each hart will start executing from once the payload has been loaded into memory by the HSS.
- **payloads**: this command specifies the executables to be included in this image.
	- **bare-metal-app.elf**: this is the path and name of an executable (paths can be given relative to the hss-payload-generator executable).
	- **exec-addr**: this is the memory location this code should be loaded into.
	- **owner-hart**: this is the main hart that will be executing this payload.
	- **secondary-hart**: this can optionally be used to specify other harts associated with this payload.
	- **priv_mode**: this specifies the privilege mode this payload will execute in, available options are:
		- priv_u - user mode.
		- priv_s - supervisor mode.
		- priv_m - machine mode.

This payload could then be generated by entering:

	./hss-payload-generator -c config.yaml output.bin

The "bare-metal-app.elf" and "confg.yaml" files are in the same directory as the "hss-payload-generator" executable in this example, these files can be stored in any local directory and relative or full paths can be provided to the HSS Payload Generator. The payload will be generated as a file called "output.bin" in the same directory as the HSS Payload Generator, this file can also be generated in a separate folder by providing a full or relative path.