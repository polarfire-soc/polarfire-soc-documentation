# Secure Boot - Code Signing

2021-11-17

## Introduction

The HSS uses a digital signature to verify if the intended image in its boot payload has been modified after it was signed. The flow for the digital signature process is illustrated below.

The Hash Function used is SHA-384.

The Signature Generation and Signature Verification Functions use the Elliptic Curve SECP 384r1 (aka NIST P-384).

To implement these, the HSS can perform the functions either using the accelerated user crypto core\*, or completely in software (albeit with an additional time penalty on boot).

For its software implementation, the HSS incorporates the third party library [libecc](https://github.com/ANSSI-FR/libecc).

\* Note: not all Microchip PolarFire SoC parts include the user crypto core.

## Digital Signature Process in Action

![Signature Generation](images/Secure_Boot--Signature_Generation.png)

Signature generation involves creating a message digest (via hash function) of the payload.bin and signing this using an Elliptic Curve Crypto library.

![Signature Verification](images/Secure_Boot--Signature_Verification.png)

Signature verification involves checking that the signature matches the binary, and then verifying the message digest against the payload.bin.

## Boot Image Generation Flow

The are three steps to the boot image generation:

* Generate an X.509 ASN.1 PEM-format private key and a matching X.509 ASN.1 DER-format public key;
* Use the payload generator to sign the `payload.bin` binary with an X.509 ASN.1 PEM-format private key;
* Use a version of the HSS which includes support for image signing, and has been built with the matching X.509 ASN.1 DER-format public key.

### Key Generation Flow

The easiest way to generate a public/private key pair is to use the OpenSSL library, available for Windows and Linux from [openssl.org](https://www.openssl.org/). OpenSSL is a robust, commercial-grade, fully-featured toolkit for general-purpose cryptography and secure communication.

OpenSSL can be compiled from source, installed via third-party community contributed binaries for Windows etc. (available from [openssl's website](https://www.openssl.org/community/binaries.html) or installed via a Linux distribution package manager (e..g., for Ubuntu, `apt install openssl`).

With OpenSSL installed, generate a private key, and a public key from the private key, as follows:

```shell
$ openssl ecparam -genkey -name secp384r1 -param_enc named_curve -out x509-ec-secp384r1-private.pem
$ openssl ec -in x509-ec-secp384r1-private.pem -pubout -out x509-ec-secp384r1-public.der -outform DER
```

There are helper scripts in `tools/secure-boot` in the HSS (`gen_keys.sh` for Linux / `gen_keys.bat` for Windows) to generate these keys using OpenSSL.

### Payload Generator

The `hss-payload-generator` tool allows the specification of an X.509 ASN.1 PEM private key for image signing, e.g.:

```shell
$ ./hss-payload-generator -c test/config.yaml payload.bin -p /path/to/private.pem
```

It calculates a SHA-384 message digest for the entire payload.bin image, and signs it using ECDSA SECP384r1.

### HSS Configuration

Once the keys are generated, the HSS needs to be compiled with support for code signing enabled and with the public-key embedded in it. To achieve this, several Kconfig options must be enabled, and paths to the public key (in DER binary format) provided:

```text
CONFIG_CRYPTO_SIGNING=y
CONFIG_CRYPTO_SIGNING_KEY_PUBLIC="tools/secure-boot/x509-ec-secp384r1-public.der"
CONFIG_CRYPTO_LIBECC=y
CONFIG_CRYPTO_SHA384=y
```

Then, build the HSS as normal.  Program the payload.bin image to the boot device (SPI flash, eMMC or SD-Card) as normal. When the HSS reads the payload image, it will check the digital signature to ensure it has not been tampered with before proceeding to boot it.

If the signature check fails, the HSS will refuse to boot the image.

## Appendix: NIST P-384 Details

The HSS uses Elliptic Curve SECP384r1 (aka NIST P-384).  This is the elliptic curve that the NSA recommends everyone use until post-quantum methods have been standardized.

It provides 192-bits of security.

The equation of the P-384 curve is:

$$y^2 = x^3 + ax + b$$

and it is defined in [FIPS PUB 186-4], section D.2.4.

Only uncompressed keys are supported.

Public keys are twice the field size in bytes plus 1. For SECP384r1, the field size is 384 bits, or 48 bytes, so the key size is 96+1 bytes.

The uncompressed key format consists of a 0x04 (DER OCTET STRING tag), plus the concatenation of the public point's x-coordinate as a big-endian binary string, and the public point's y-coordinate as a big-endian binary string.

The dumpasn1 tool is able to inspect the format of the public key:

```text
$ dumpasn1 x509-ec-secp384r1-public.der

  0 118: SEQUENCE {
  2  16:   SEQUENCE {
  4   7:     OBJECT IDENTIFIER ecPublicKey (1 2 840 10045 2 1) (ANSI X9.62 public key type)
 13   5:     OBJECT IDENTIFIER secp384r1 (1 3 132 0 34) (SECG (Certicom) named elliptic curve)
       :     }
 20  98:   BIT STRING
       :     04 xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx
       :     xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx
       :     xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx
       :     xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx
       :     xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx
       :     xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx
       :     xx
       :   }

```

This means that the first 24 bytes of the key are constant, indicating an EC SECP384r1 key, and the last 96 bytes are the X and Y portions of the public point itself.

## Useful Links and References

* lapo.it has an [online ASN.1 JavaScript decoder](https://lapo.it/asn1js/) that can decode PEM public keys.
* [FIPS PUB 186-4] Digital Signature Standard (DSS), July 2013, available from [nist.gov](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf)
* [ANSI X9.62] Public Key Cryptography for the Financial Services Industry: the Elliptic Curve Digital Signature Algorithm (ECDSA), November 2005, - available [here](https://standards.globalspec.com/std/1955141/ANSI%20X9.62)
* [IEEE 1363-2000] IEEE Standard Specifications for Public-Key Cryptography, January 2000, available [here](https://standards.ieee.org/standard/1363-2000.html)
