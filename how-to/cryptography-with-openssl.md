# Microchip® PolarFire® SoC - Crypto hardware acceleration

- [Introduction](#introduction)
- [Hardware Support](#hardware-support)
- [Prerequisites](#prerequisites)
  - [Hart Software Services Requirements](#hart-software-services-requirements)
  - [Linux Requirements](#linux-requirements)
  - [cryptodev-linux](#cryptodev-linux)
  - [af_alg](#af_alg)
- [Applications](#applications)
  - [OpenSSL](#openssl)
    - [OpenSSL Engine](#openssl-engine)
    - [AES-ECB 256-bit Encryption](#aes-ecb-256-bit-encryption)
    - [AES-ECB 256-bit Decryption](#aes-ecb-256-bit-decryption)
    - [AES-CBC 256-bit Encryption](#aes-cbc-256-bit-encryption)
    - [AES-CBC 256-bit Decryption](#aes-cbc-256-bit-decryption)
- [Benchmarks](#benchmarks)

<a name="introduction"></a>

## Introduction

This document describes how to use crypto hardware acceleration on PolarFire SoC using OpenSSL.
While PolarFire SoC supports rich set of [cryptography services](https://www.microchip.com/content/dam/mchp/documents/FPGA/ProductDocuments/UserGuides/Microchip_PolarFire_FPGA_and_PolarFire_SoC_FPGA_Security_User_Guide_VA%20(2).pdf), currently only the AES services are supported in software.

<a name="hardware-support"></a>

## Hardware Support

Before attempting to use hardware crypto acceleration, make sure you have a board with a PolarFire SoC device with the suffix 'S', which provides a dedicated cryptoprocessor, such as the PolarFire SoC Video Kit (MPFS250-VIDEO-KIT). The Icicle Kit with Engineering Sample silicon (denoted as 'ES') does not support crypto hardware acceleration.

<a name="prerequisites"></a>

## Prerequisites

The use of user crypto hardware acceleration, two software components are needed and implemented as:

- A low level device driver in Hart Software Services(HSS).
- A linux kernel driver.

In order to use the user crypto Linux kernel driver from a user space application, a third party kernel module is needed to expose the accelerator to user space cryptodev-linux and AF_ALG (which supports fewer algorithms) are the two such interfaces.

<a name="hart-software-services-requirements"></a>

### Hart Software Services Requirements

To enable support for the crypto services, the following Hart Software Services (HSS) Kconfig options are required. These defconfig options are enabled in the HSS for the PolarFire SoC Video Kit starting from release v2024.02 and onwards.

```c
CONFIG_USE_USER_CRYPTO=y
CONFIG_SERVICE_OPENSBI=y
CONFIG_SERVICE_OPENSBI_CRYPTO=y
```

<a name="linux-requirements"></a>

### Linux Requirements

The crypto driver and subsystem must be enabled:

```c
CONFIG_CRYPTO_DEV_POLARFIRE_SOC=y
CONFIG_CRYPTO_USER=y
CONFIG_CRYPTO_USER_API=y
CONFIG_CRYPTO_USER_API_SKCIPHER=y
```

These defconfig options are enabled in the Linux for the PolarFire SoC Video Kit starting from release v2024.02 and onwards.

<a name="cryptodev-linux"></a>

### cryptodev-linux

[Cryptodev-linux](http://cryptodev-linux.org/) is a device that allows access to Linux kernel cryptographic drivers; thus allowing user-space applications to take advantage of hardware accelerators. Cryptodev-linux is implemented as a standalone module that requires no dependencies other than a stock linux kernel. Its API is compatible with OpenBSD's cryptodev user-space API (/dev/crypto).

<a name="af_alg"></a>

### af_alg

The AF_ALG interface uses sockets to allow access to the kernel crypto algorithms, so there is no /dev/crypto interface associated with it.

<a name="applications"></a>

## Applications

If cryptodev is built and installed in the root filesystem, after login in, enter the following command to load the cryptodev module.

```c
root@mpfs-video-kit:~# modprobe cryptodev
[   26.639569] cryptodev: loading out-of-tree module taints kernel.
[   26.665105] cryptodev: driver 1.12 loaded.
```

When the module has loaded, a device node at /dev/crypto will appear. If not, the hardware acceleration cannot be used in user space program.

<a name="openssl"></a>

### OpenSSL

OpenSSL is an open source toolkit implementing the Secure Socket Layer (SSL) and Transport Layer Security protocols as well as a full-strength general purpose cryptography library.

It can be used with cryptodev-linux, which creates the link between OpenSSL and cryptography hardware drivers.

<a name="openssl-engine"></a>

#### OpenSSL Engine

Set the devcrypto engine with OpenSSL.

```c
root@mpfs-video-kit:~# openssl engine devcrypto
(devcrypto) /dev/crypto engine
```

<a name="aes-ecb-256-bit-encryption"></a>

#### AES-ECB 256-bit Encryption

```c
root@mpfs-video-kit:~# echo "020000300040050" >plain.txt
```

```c
root@mpfs-video-kit:~# openssl enc -engine devcrypto -aes-256-ecb -e -K 1d85a181b54cde51f0e098095b2962fdc93b51fe9b88602b3f54130bf76a5bd9 -in plain.txt -out cipher.txt
```

<a name="aes-ecb-256-bit-decryption"></a>

#### AES-ECB 256-bit Decryption

```c
root@mpfs-video-kit:~# openssl enc -engine devcrypto -aes-256-ecb -d -K 1d85a181b54cde51f0e098095b2962fdc93b51fe9b88602b3f54130bf76a5bd9 -in cipher.txt -out dec.txt
```

<a name="aes-cbc-256-bit-encryption"></a>

#### AES-CBC 256-bit Encryption

```c
root@mpfs-video-kit:~# openssl enc -engine devcrypto -aes-256-cbc -e -K 1d85a181b54cde51f0e098095b2962fdc93b51fe9b88602b3f54130bf76a5bd9 -iv 00000000000000000000000000000000 -in plain.txt -out cipher.txt -nopad
```

<a name="aes-cbc-256-bit-decryption"></a>

#### AES-CBC 256-bit Decryption

```c
root@mpfs-video-kit:~# openssl enc -engine devcrypto -aes-256-cbc -d -K 1d85a181b54cde51f0e098095b2962fdc93b51fe9b88602b3f54130bf76a5bd9 -iv 00000000000000000000000000000000 -in cipher.txt -out dec.txt -nopad
```

Note: Use `-nopad` for AES-CBC mode encryption and decryption.

<a name="benchmarks"></a>

## Benchmarks

OpenSSL can perform benchmarks when passed the "speed" parameter. Other parameters used for the benches are:

- evp: algorithm name - AES: aes-128-cbc, aes-192-cbc and aes-256-cbc.
- elapsed: performances are calculated in taking into account time consumed in user space and kernel space. Without this parameter only user space time is used.

In the speed test, a series of performance tests are done to check the performance of the symmetric operation. The following is described in the OpenSSL test execution:

```c
root@mpfs-video-kit:~# openssl speed -evp aes-128-cbc -engine devcrypto -elapsed
Engine "devcrypto" set.
You have chosen to measure elapsed time instead of user CPU time.
Doing AES-128-CBC for 3s on 16 size blocks: 1066896 AES-128-CBC's in 3.00s
Doing AES-128-CBC for 3s on 64 size blocks: 320699 AES-128-CBC's in 3.00s
Doing AES-128-CBC for 3s on 256 size blocks: 84747 AES-128-CBC's in 3.00s
Doing AES-128-CBC for 3s on 1024 size blocks: 21491 AES-128-CBC's in 3.00s
Doing AES-128-CBC for 3s on 8192 size blocks: 2697 AES-128-CBC's in 3.00s
Doing AES-128-CBC for 3s on 16384 size blocks: 1347 AES-128-CBC's in 3.00s
version: 3.0.5
built on: Tue Jul  5 08:57:04 2022 UTC
options: bn(64,64)
compiler: riscv64-oe-linux-gcc    -fstack-protector-strong  -O2 -D_FORTIFY_SOURCE=2 -Wformat -Wformat-security -Werror=format-security --sysroot=recipe-sysroot -O2 -pipe -g -feliminate-unused-debug-types -fmacro-prefix-map=                      -fdebug-prefix-map=                      -fdebug-prefix-map=                      -fdebug-prefix-map=  -DOPENSSL_USE_NODELETE -DOPENSSL_PIC -DOPENSSL_BUILDING_OPENSSL -DNDEBUG
CPUINFO: N/A
The 'numbers' are in 1000s of bytes per second processed.
type             16 bytes     64 bytes    256 bytes   1024 bytes   8192 bytes  16384 bytes
AES-128-CBC       5690.11k     6841.58k     7231.74k     7335.59k     7364.61k     7356.42k
root@mpfs-video-kit:~#
```

Additional ciphers that could be benchmarked: `aes-192-cbc`, `aes-256-cbc`, `aes-128-ecb`, `aes-192-ecb`, `aes-256-ecb`, `aes-128-ctr`, `aes-192-ctr`, `aes-256-ctr`.
