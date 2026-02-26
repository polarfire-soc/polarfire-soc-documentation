# Microchip® PolarFire® SoC - Crypto hardware acceleration

- [Introduction](#introduction)
- [Hardware Support](#hardware-support)
- [Prerequisites](#prerequisites)
  - [Hart Software Services Requirements](#hart-software-services-requirements)
  - [Linux Kernel Requirements](#linux-requirements)
  - [User Space Interface](#user-space-interface)
- [Applications](#applications)
  - [Libkcapi](#Libkcapi)
    - [OpenSSL](#openssl)
    - [AES-CBC 128-bit Encryption](#aes-cbc-128-bit-encryption)
    - [AES-CBC 128-bit Decryption](#aes-cbc-128-bit-decryption)
- [Benchmarks](#benchmarks)

<a name="introduction"></a>

## Introduction

This document describes how to use crypto hardware acceleration on PolarFire SoC.

While PolarFire SoC supports rich set of [cryptography services](https://www.microchip.com/content/dam/mchp/documents/FPGA/ProductDocuments/UserGuides/Microchip_PolarFire_FPGA_and_PolarFire_SoC_FPGA_Security_User_Guide_VA%20(2).pdf), currently only the AES services are supported in software.

<a name="hardware-support"></a>

## Hardware Support

Before attempting to use hardware crypto acceleration, make sure you have a board with a PolarFire SoC device with the suffix 'S', which provides a dedicated cryptoprocessor, such as the PolarFire SoC Video Kit (MPFS250-VIDEO-KIT). The Icicle Kit with Production and Engineering Sample silicon (denoted as 'ES') does not support crypto hardware acceleration.

<a name="prerequisites"></a>

## Prerequisites

To use user crypto hardware acceleration, the following software components are required:

- A low level device driver in Hart Software Services(HSS).
- A linux kernel driver.
- A userspace interface and application to interact with the Linux Kernel Crypto API

<a name="hart-software-services-requirements"></a>

### Hart Software Services Requirements

To enable support for the crypto services, the following Hart Software Services (HSS) Kconfig options are required. These defconfig options are enabled
by default in the HSS for the PolarFire SoC Video Kit starting from release v2024.02 and onwards.

```c
CONFIG_USE_USER_CRYPTO=y
CONFIG_SERVICE_OPENSBI=y
CONFIG_SERVICE_OPENSBI_CRYPTO=y
```

<a name="linux-requirements"></a>

### Linux Kernel Requirements

To use crypto hardware acceleration, ensure the following kernel configuration options are set:

```c
CONFIG_CRYPTO=y
CONFIG_CRYPTO_CBC=y
CONFIG_CRYPTO_USER=y
CONFIG_CRYPTO_USER_API=y
CONFIG_CRYPTO_USER_API_SKCIPHER=y
CONFIG_CRYPTO_DEV_POLARFIRE_SOC=y
```

These defconfig options are enabled in the Linux for the PolarFire SoC Video Kit starting from release v2024.02 and onwards.

To view the available crypto skciphers in the kernel, use the following command:

```sh
cat /proc/crypto
```

A sample output is shown below, listing both generic and hardware-accelerated cipher implementations. Hardware-accelerated drivers provided by Microchip are indicated by the microchip- prefix and have higher priority values (e.g., 300):

```text
name         : ctr(aes)
driver       : ctr(aes-generic)
...
name         : ctr(aes)
driver       : microchip-ctr-aes
priority     : 300
...
name         : cfb(aes)
driver       : microchip-cfb-aes
priority     : 300
...
name         : ofb(aes)
driver       : microchip-ofb-aes
priority     : 300
...
name         : cbc(aes)
driver       : microchip-cbc-aes
priority     : 300
...
name         : ecb(aes)
driver       : microchip-ecb-aes
priority     : 300
...
```

<a name="user-space-interface"></a>

### User Space Interface

User-space applications access the cryptoprocessor via the Linux Kernel Crypto API using the AF_ALG interface.

AF_ALG is a Linux kernel mechanism that exposes cryptographic algorithms to user space through special sockets,
enabling applications to perform encryption and decryption using kernel-supported implementations.

This document focuses on `libkcapi`, the officially supported user-space library for AF_ALG as well as using
openSSL with AF_ALG engine.

<a name="applications"></a>

## Applications

<a name="libkcapi"></a>

### Libkcapi

Starting with release v2026.04, the kcapi library and the official libkcapi test applications are
included in the root filesystem of Microchip Yocto Project images (mchp-base-image).

The following example demonstrates how to encrypt a file using libkcapi:

1. Create an input file:

    ```sh
    echo -n "0123456789abcdef" > /tmp/input.bin
    ```

    The command above writes 16 bytes of data to /tmp/input.bin.

2. Create a key file:

    ```sh
    echo -n -e "\x00\x11\x22\x33\x44\x55\x66\x77\x88\x99\xaa\xbb\xcc\xdd\xee\xff" > /tmp/key.bin
    ```

    This command writes a 16-byte AES key to /tmp/key.bin.

3. Encrypt the file using `kcapi-enc`:

    ```sh
    kcapi-enc -c "cbc(aes)" --encrypt --infile /tmp/input.bin --outfile /tmp/out.bin --iv 000102030405060708090a0b0c0d0e0f --keyfd 3 3</tmp/key.bin --verbose
    ```

    Explanation of options:

    `-c "cbc(aes)"`: Specifies AES in CBC mode.

    `--encrypt`: Sets the operation to encryption.

    `--infile`:  Input file to encrypt.

    `--outfile`: Output file for encrypted data.

    `--iv 000102030405060708090a0b0c0d0e0f`:
    The IV ensures identical plaintext blocks produce different ciphertext.
    Adds randomness and security to encryption.
    Must be 16 bytes for AES.

    `--keyfd 3 3</tmp/key.bin`:
    Allows securely providing the encryption key using a file descriptor.
    `3</tmp/key.bin` opens `/tmp/key.bin` as FD 3.

    `--verbose`: Enables detailed output for debugging.

4. Decrypting a file with libkcapi

To decrypt a file that was encrypted using kcapi-enc, you use a similar command, specifying the --decrypt option and providing the same key and IV used for encryption

```sh
kcapi-enc -c "cbc(aes)" --decrypt --infile /tmp/out.bin --outfile /tmp/decrypted.bin --iv 000102030405060708090a0b0c0d0e0f --keyfd 3 3</tmp/key.bin --verbose
```

  To decrypt, use the same key and IV as for encryption, and specify the encrypted file as input. The output will be the original plaintext if the key and IV are correct.

<a name="openssl"></a>

### OpenSSL

OpenSSL is an open source toolkit implementing the Secure Socket Layer (SSL) and Transport Layer Security protocols as well as a full-strength general purpose cryptography library.

It can be used with afalg, which creates the link between OpenSSL and cryptography hardware drivers.

To test if OpenSSL detects the AF_ALG engine:

```sh
openssl engine -t -c afalg
```

Example output:

```text
(afalg) AFALG engine support
 [AES-128-CBC, AES-192-CBC, AES-256-CBC]
     [ available ]
```

<a name="aes-cbc-128-bit-encryption"></a>

#### AES-CBC 128-bit Encryption

1. Create an input file:

    ```c
    echo "020000300040050" >plain.txt
    ```

2. Encrypt the file using OpenSSL with AF_ALG:

    ```c
    openssl enc -engine afalg -aes-128-cbc -e -K 1d85a181b54cde51f0e098095b2962f0 -iv 00000000000000000000000000000000 -in plain.txt -out cipher.txt -nopad
    ```

    `--engine afalg`: Use the AF_ALG engine for hardware acceleration.
    `--aes-128-cbc`: Use AES-128 in CBC mode.
    `--e`: Encrypt mode.
    `--K`: 128-bit (32 hex characters) encryption key.
    `--iv`: 16-byte (32 hex characters) initialization vector.
    `--in plain.txt`: Input plaintext file.
    `--out cipher.txt`: Output encrypted file.
    `--nopad`: Disables padding (ensure your input length is a multiple of the block size).

<a name="aes-cbc-128-bit-decryption"></a>

#### AES-CBC 128-bit Decryption

To decrypt the file:

```c
openssl enc -engine afalg -aes-128-cbc -d -K 1d85a181b54cde51f0e098095b2962f0 -iv 00000000000000000000000000000000 -in cipher.txt -out dec.txt -nopad
```

- -d: Decrypt mode.

- All other options should match those used during encryption.

<a name="benchmarks"></a>

## Benchmarks

OpenSSL can perform benchmarks when passed the "speed" parameter. Other parameters used for the benches are:

- evp: algorithm name - AES: aes-128-cbc, aes-192-cbc and aes-256-cbc.
- elapsed: performances are calculated in taking into account time consumed in user space and kernel space. Without this parameter only user space time is used.

In the speed test, a series of performance tests are done to check the performance of the symmetric operation. The following is described in the OpenSSL test execution:

Example: Hardware-Accelerated (AF_ALG Engine) vs. Software Implementation

1. Hardware-Accelerated (AF_ALG Engine):

    ```sh
    openssl speed -evp aes-256-cbc -engine afalg -elapsed
    Engine "afalg" set.
    You have chosen to measure elapsed time instead of user CPU time.
    Doing AES-256-CBC ops for 3s on 16 size blocks: 17945 AES-256-CBC ops in 3.00s
    Doing AES-256-CBC ops for 3s on 64 size blocks: 17964 AES-256-CBC ops in 3.00s
    Doing AES-256-CBC ops for 3s on 256 size blocks: 16861 AES-256-CBC ops in 3.00s
    Doing AES-256-CBC ops for 3s on 1024 size blocks: 13719 AES-256-CBC ops in
    3.00s
    Doing AES-256-CBC ops for 3s on 8192 size blocks: 5350 AES-256-CBC ops in 3.00s
    Doing AES-256-CBC ops for 3s on 16384 size blocks: 3159 AES-256-CBC ops in
    3.00s
    version: 3.2.4
    built on: Tue Feb 11 14:38:30 2025 UTC
    options: bn(64,64)
    compiler: riscv64-mchp-linux-gcc     -fstack-protector-strong  -O2
    -D_FORTIFY_SOURCE=2 -Wformat -WformaG
    CPUINFO: N/A
    The 'numbers' are in 1000s of bytes per second processed.
    type             16 bytes     64 bytes    256 bytes   1024 bytes   8192 bytes
    16384 bytes
    AES-256-CBC         95.71k      383.23k     1438.81k     4682.75k    14609.07k 17252.35k
    ```

2. Software Implementation:

    ```sh
    openssl speed -evp aes-256-cbc -elapsed

    root@mpfs-video-kit:~# openssl speed -evp aes-256-cbc -elapsed
    You have chosen to measure elapsed time instead of user CPU time.
    Doing AES-256-CBC ops for 3s on 16 size blocks: 1071013 AES-256-CBC ops in
    3.00s
    Doing AES-256-CBC ops for 3s on 64 size blocks: 310661 AES-256-CBC ops in 3.00s
    Doing AES-256-CBC ops for 3s on 256 size blocks: 81070 AES-256-CBC ops in 3.00s
    Doing AES-256-CBC ops for 3s on 1024 size blocks: 20495 AES-256-CBC ops in
    3.00s
    Doing AES-256-CBC ops for 3s on 8192 size blocks: 2569 AES-256-CBC ops in 3.00s
    Doing AES-256-CBC ops for 3s on 16384 size blocks: 1283 AES-256-CBC ops in
    3.01s
    version: 3.2.4
    built on: Tue Feb 11 14:38:30 2025 UTC
    options: bn(64,64)
    compiler: riscv64-mchp-linux-gcc     -fstack-protector-strong  -O2
    -D_FORTIFY_SOURCE=2 -Wformat -WformaG
    CPUINFO: N/A
    The 'numbers' are in 1000s of bytes per second processed.
    type             16 bytes     64 bytes    256 bytes   1024 bytes   8192 bytes 16384 bytes
    AES-256-CBC       5712.07k     6627.43k     6917.97k     6995.63k     7015.08k 6983.61k
    ```

  Note:
  Performance may vary depending on system configuration, kernel version, and whether the AF_ALG engine is properly enabled.
