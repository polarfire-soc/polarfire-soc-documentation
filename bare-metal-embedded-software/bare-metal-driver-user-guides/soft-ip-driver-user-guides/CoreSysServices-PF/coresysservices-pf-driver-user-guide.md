<html>
 
 ------------------------------------ 

# CoreSysServices_PF Bare Metal Driver.
 
-----------------------------------------
## Table of Contents 
  - [Introduction](#introduction)

     - [Features](#features)

  - [Configuration](#configuration)

  - [Theory of Operation](#theory-of-operation)

- [Types](#types)

- [Constants](#constants)
  - [Service Execution Success and Error Status Codes](#service-execution-success-and-error-status-codes)
  - [System Service Timeout Count](#system-service-timeout-count)
  - [Secure Nvm Write Error Codes](#secure-nvm-write-error-codes)
  - [Secure Nvm Read Error Codes](#secure-nvm-read-error-codes)
  - [Digital Signature Service Error Codes](#digital-signature-service-error-codes)
  - [Digest Check Error Codes](#digest-check-error-codes)
  - [Bitstream Authentication and Iap Bitstream Authentication Return Status](#bitstream-authentication-and-iap-bitstream-authentication-return-status)
  - [Mailbox ECC Status](#mailbox-ecc-status)
  - [Digest Check Input Options](#digest-check-input-options)

- [Functions](#functions)
  - [SYS_init](#sysinit)
  - [SYS_get_serial_number](#sysgetserialnumber)
  - [SYS_get_user_code](#sysgetusercode)
  - [SYS_get_design_info](#sysgetdesigninfo)
  - [SYS_get_device_certificate](#sysgetdevicecertificate)
  - [SYS_read_digest](#sysreaddigest)
  - [SYS_query_security](#sysquerysecurity)
  - [SYS_read_debug_info](#sysreaddebuginfo)
  - [SYS_read_envm_parameter](#sysreadenvmparameter)
  - [SYS_puf_emulation_service](#syspufemulationservice)
  - [SYS_digital_signature_service](#sysdigitalsignatureservice)
  - [SYS_secure_nvm_write](#syssecurenvmwrite)
  - [SYS_secure_nvm_read](#syssecurenvmread)
  - [SYS_nonce_service](#sysnonceservice)
  - [SYS_bitstream_authenticate_service](#sysbitstreamauthenticateservice)
  - [SYS_IAP_image_authenticate_service](#sysiapimageauthenticateservice)
  - [SYS_digest_check_service](#sysdigestcheckservice)
  - [SYS_iap_service](#sysiapservice)

<div id="TitlePage" data-type="text">

# Introduction
The PolarFire® System Services (PF_SYSTEM_SERVICES) SgCore enables executing the
system services on the PolarFire and PolarFire SoC device. The system services
are the system controller actions initiated by the System Controller's System
Service Interface (SSI). The PolarFire System Services "SgCore" provides a
method to initiate these system services. The PF_SYSTEM_SERVICES interacts with
the system controller on SSI and Mailbox interface to initiate system services,
exchange data required for that services, and to know the successful completion
or error status.

The PF_SYSTEM_SERVICES provides an APB interface for controlling the registers
functions for controlling the PF_SYSTEM_SERVICES as part of a bare metal system
register implemented within it. This software driver provides a set of where no
part of an operating system but the implementation of the adaptation layer
operating system is available. This driver is adapted for use in between this
driver and the operating system's driver model is outside the scope of this
driver.

## Features
The CoreSysServices_PF driver provides the following features:

  - Executing device and design information services
  - Executing design services
  - Executing data security services
  - Executing Fabric services

The CoreSysServices_PF driver is provided as C source code.

# Configuration
The application software should configure the CoreSysServices_PF driver through
calling the SYS_init() function. Only one instance of PF_SYSTEM_SERVICES SgCore
is supported. No additional configuration files are required to use the driver.
If using this driver on RT PolarFire device FPGA, define RT_DEVICE_FAMILY macro
in application.

# Theory of Operation
The CoreSysServices_PF driver provides access to the PolarFire system services.
These system services are grouped into the following categories:

Device and Design Information Service

  - Serial Number Service
  - USERCODE Service
  - Design Info Service
  - Device Certificate Services
  - Read Digests
  - Query Security
  - Read Debug Info
  - Read eNVM param

Design Services

  - Bitstream authentication service
  - IAP bitstream authentication service

Data Security Services

  - Digital Signature Service
  - Secure NVM (SNVM) Functions
  - PUF Emulation Service
  - Nonce Service

Fabric Services

  - Digest Check Service
  - In Application programming(IAP)/Auto-Update service

Initialization and Configuration

The CoreSysServices_PF driver is initialized by calling the SYS_init() function.
The SYS_init() function must be called before calling any other
CoreSysServices_PF driver functions.

Device and Design Information Services

The CoreSysServices_PF driver is used to read information about the device and
the design using the following functions:

  - SYS_get_serial_number()
  - SYS_get_user_code()
  - SYS_get_design_info()
  - SYS_get_device_certificate()
  - SYS_read_digest()
  - SYS_query_security()
  - SYS_read_debug_info()

Design Authentication Services

The CoreSysServices_PF driver is used to execute design services using the
following functions:

  - SYS_bitstream_authenticate_service()
  - SYS_IAP_image_authenticate_service()

Data Security Services

The CoreSysServices_PF driver is used to execute data security services using
the following functions:

  - SYS_digital_signature_service()
  - SYS_secure_nvm_write()
  - SYS_secure_nvm_read()
  - SYS_puf_emulation_service ()
  - SYS_nonce_service ()

Executing Fabric Services

The CoreSysServices_PF driver is used to execute fabric services using the
following functions:

  - SYS_digest_check_service()
  - SYS_iap_service()

All the service execution functions return the 8-bit status, which is returned
by the system controller on executing the given service. A '0' value indicates
successful execution of that service. A non-zero value indicates error. The
error codes for each service are different. See individual function description
to know the exact meanings of the error codes for each service.

The function descriptions in this file mainly focus on the details required by
the user to use the APIs provided by this driver to execute the services. To
know the complete details of the system services, see the PolarFire FPGA and
PolarFire SoC FPGA System Services <a href="https://onlinedocs.microchip.com/pr/
GUID-1409CF11-8EF9-4C24-A94E-70979A688632-en-US-3/index.html">document</a>

</div>


# Types

 ---------------- 

# Constants

 ---------------- 
<div id="Constants$SYS_SUCCESS$description" data-type="text">

<a name="syssuccess"></a>
## Service Execution Success and Error Status Codes
The following status codes are the return values from the system service
functions. For any service, a return value '0' indicates that the service was
executed successfully. A non-zero return value indicates that the service was
not executed successfully. For all the services, the return value represents the
status code returned by the system controller for the respective service, except
the values SYS_PARAM_ERR, SS_USER_BUSY_TIMEOUT, and SS_USER_RDVLD_TIMEOUT. These
three values indicate the error conditions detected by this driver and they do
not overlap with the status code returned by the system controller for any of
the system service.

SYS_SUCCESS  
System service executed successfully

SYS_PARAM_ERR  
System service cannot be executed as one or more parameters are
not as expected by this driver. No read/write access is performed with the IP.

SS_USER_BUSY_TIMEOUT  
The System service request is initiated and the driver
timed-out while waiting for the system service to complete. The System Service
completion is indicated by de-assertion of the SS_USER_BUSY bit by the IP.

SS_USER_RDVLD_TIMEOUT  
The System service request is initiated and the driver
timed-out while waiting for SS_USER_RDVLD bit, which indicates availability of
data to be read from the mailbox, to become active.

</div>

<div id="Constants$SS_TIMEOUT_COUNT$description" data-type="text">

<a name="sstimeoutcount"></a>
## System Service Timeout Count
The SS_TIMEOUT_COUNT value is used by the driver as a timeout count while
waiting for either the SS_USER_BUSY or SS_USER_RDVLD. This empirical value is
sufficiently large so that the operations are falsely timeout in the normal
circumstance. It is provided as a way to provide more debug information to the
application in case there are some unforeseen issues. You may change this value
for your need based on your system design.

</div>

<div id="Constants$SNVM_WRITE_INVALID_SNVMADDR$description" data-type="text">

<a name="snvmwriteinvalidsnvmaddr"></a>
## Secure Nvm Write Error Codes
SNVM_WRITE_INVALID_SNVMADDR  
Illegal page address

SNVM_WRITE_FAILURE  
PNVM program/verify failed

SNVM_WRITE_SYSTEM_ERROR  
PUF or storage failure

SNVM_WRITE_NOT_PERMITTED  
Write is not permitted

</div>

<div id="Constants$SNVM_READ_INVALID_SNVMADDR$description" data-type="text">

<a name="snvmreadinvalidsnvmaddr"></a>
## Secure Nvm Read Error Codes
SNVM_READ_INVALID_SNVMADDR  
Illegal page address

SNVM_READ_AUTHENTICATION_FAILURE  
Storage corrupt or incorrect USK

SNVM_READ_SYSTEM_ERROR  
PUF or storage failure

</div>

<div id="Constants$DIGITAL_SIGNATURE_FEK_FAILURE_ERROR$description" data-type="text">

<a name="digitalsignaturefekfailureerror"></a>
## Digital Signature Service Error Codes
DIGITAL_SIGNATURE_FEK_FAILURE_ERROR  
Error retrieving FEK

DIGITAL_SIGNATURE_DRBG_ERROR  
Failed to generate nonce

DIGITAL_SIGNATURE_ECDSA_ERROR  
ECDSA failed

</div>

<div id="Constants$DIGEST_CHECK_FABRICERR$description" data-type="text">

<a name="digestcheckfabricerr"></a>
## Digest Check Error Codes
NOTE: When these error occur, the DIGEST tamper flag is triggered.

DIGEST_CHECK_FABRICERR  
Fabric digest check error

DIGEST_CHECK_CCERR  
UFS Fabric Configuration (CC) segment digest check error

DIGEST_CHECK_SNVMERR  
ROM digest in SNVM segment digest check error

DIGEST_CHECK_ULERR  
UFS UL segment digest check error

DIGEST_CHECK_UK0ERR  
UKDIGEST0 in User Key segment digest check error

DIGEST_CHECK_UK1ERR  
UKDIGEST1 in User Key segment digest check error

DIGEST_CHECK_UK2ERR  
UKDIGEST2 in User Key segment (UPK1) digest check error

DIGEST_CHECK_UK3ERR  
UKDIGEST3 in User Key segment (UK1) digest check error

DIGEST_CHECK_UK4ERR  
UKDIGEST4 in User Key segment (DPK) digest check error

DIGEST_CHECK_UK5ERR  
UKDIGEST5 in User Key segment (UPK2) digest check error

DIGEST_CHECK_UK6ERR  
UKDIGEST6 in User Key segment (UK2) digest check error

DIGEST_CHECK_UPERR  
UFS Permanent Lock (UPERM) segment digest check error

DIGEST_CHECK_SYSERR  
M3 ROM, Factory and Factory Key Segments digest check
error

</div>

<div id="Constants$BSTREAM_AUTH_CHAINING_MISMATCH_ERR$description" data-type="text">

<a name="bstreamauthchainingmismatcherr"></a>
## Bitstream Authentication and Iap Bitstream Authentication Return Status
BSTREAM_AUTH_CHAINING_MISMATCH_ERR  
Validator or hash chaining mismatch.
Incorrectly constructed bitstream or wrong key used.

BSTREAM_AUTH_UNEXPECTED_DATA_ERR  
Unexpected data received. Additional data
received after end of EOB component.

BSTREAM_AUTH_INVALID_ENCRY_KEY_ERR  
Invalid/corrupt encryption key. The
requested key mode is disabled or the key could not be read/reconstructed.

BSTREAM_AUTH_INVALID_HEADER_ERR  
Invalid component header

BSTREAM_AUTH_BACK_LEVEL_NOT_SATISFIED_ERR  
Back level not satisfied

BSTREAM_AUTH_ILLEGAL_BITSTREAM_MODE_ERR  
Illegal bitstream mode. Requested
bitstream mode is disabled by user security.

BSTREAM_AUTH_DNS_BINDING_MISMATCH_ERR  
DSN binding mismatch

BSTREAM_AUTH_ILLEGAL_COMPONENT_SEQUENCE_ERR  
Illegal component sequence

BSTREAM_AUTH_INSUFF_DEVICE_CAPAB_ERR  
Insufficient device capabilities

BSTREAM_AUTH_INCORRECT_DEVICEID_ERR  
Incorrect DEVICEID

BSTREAM_AUTH_PROTOCOL_VERSION_ERR  
Unsupported bitstream protocol version
(regeneration required)

BSTREAM_AUTH_VERIFY_ERR  
Verify not permitted on this bitstream

BSTREAM_AUTH_INVALID_DEV_CERT_ERR  
Invalid Device Certificate. Device SCAC is
invalid or not present.

BSTREAM_AUTH_INVALID_DIB_ERR  
Invalid DIB

BSTREAM_AUTH_SPI_NOT_MASTER_ERR  
Device not in SPI Master Mode. Error may occur
only when bitstream is executed through IAP mode.

BSTREAM_AUTH_AUTOIAP_NO_VALID_IMAGE_ERR  
No valid images found. Error may occur
when bitstream is executed through Auto Update mode. Occurs when no valid image
pointers are found.

BSTREAM_AUTH_INDEXIAP_NO_VALID_IMAGE_ERR  
No valid images found. Error may
occur when bitstream is executed through IAP mode via Index Mode. Occurs when No
valid image pointers are found.

BSTREAM_AUTH_NEWER_DESIGN_VERSION_ERR  
Programmed design version is newer than
AutoUpdate image found. Error may occur when bitstream is executed through Auto
Update mode.

BSTREAM_AUTH_INVALID_IMAGE_ERR  
Selected image was invalid and no recovery was
performed due to valid design in device. Error may occur only when bitstream is
executed through Auto Update or IAP mode (This error is here for completeness
but only can be observed by running the READ_DEBUG_INFO instruction and looking
at IAP Error code field).

BSTREAM_AUTH_IMAGE_PROGRAM_FAILED_ERR  
Selected and Recovery image failed to
program. Error may occur only when bitstream is executed through Auto Update or
IAP mode (This error is here for completeness but only can be observed by
running the READ_DEBUG_INFO instruction and looking at IAP Error code field).

BSTREAM_AUTH_ABORT_ERR  
Abort. Non-bitstream instruction executed during
bitstream loading.

BSTREAM_AUTH_NVMVERIFY_ERR  
Fabric/UFS verification failed (min or weak limit)

BSTREAM_AUTH_PROTECTED_ERR  
Device security prevented modification of non-
volatile memory

BSTREAM_AUTH_NOTENA  
Programming mode not enabled

BSTREAM_AUTH_PNVMVERIFY  
pNVM verify operation failed

BSTREAM_AUTH_SYSTEM  
System hardware error (PUF or DRBG)

BSTREAM_AUTH_BADCOMPONENT  
An internal error was detected in a component
payload

BSTREAM_AUTH_HVPROGERR  
HV programming subsystem failure (pump failure)

BSTREAM_AUTH_HVSTATE  
HV programming subsystem in unexpected state (internal
error)

</div>

<div id="Constants$SYS_MBOX_ECC_NO_ERROR_MASK$description" data-type="text">

<a name="sysmboxeccnoerrormask"></a>
## Mailbox ECC Status
Provides ECC status when the mailbox is read. The values are as follows:  
00:
No ECC errors detected, data is correct.  
01: Exactly one bit error occurred
and has been corrected.  
10: Exactly two bits error occurred and no correction
performed.  
11: Reserved.

</div>

<div id="Constants$DIGEST_CHECK_FABRIC$description" data-type="text">

<a name="digestcheckfabric"></a>
## Digest Check Input Options
DIGEST_CHECK_FABRIC  
Carry out digest check on Fabric

DIGEST_CHECK_CC  
Carry out digest check on UFS Fabric Configuration (CC)
segment

DIGEST_CHECK_SNVM  
Carry out digest check on ROM digest in SNVM segment

DIGEST_CHECK_UL  
Carry out digest check on UFS UL segment

DIGEST_CHECK_UKDIGEST0  
Carry out digest check on UKDIGEST0 in User Key segment

DIGEST_CHECK_UKDIGEST1  
Carry out digest check on UKDIGEST1 in User Key segment

DIGEST_CHECK_UKDIGEST2  
Carry out digest check on UKDIGEST2 in User Key segment
(UPK1)

DIGEST_CHECK_UKDIGEST3  
Carry out digest check on UKDIGEST3 in User Key segment
(UK1)

DIGEST_CHECK_UKDIGEST4  
Carry out digest check on UKDIGEST4 in User Key segment
(DPK)

DIGEST_CHECK_UKDIGEST5  
Carry out digest check on UKDIGEST5 in User Key segment
(UPK2)

DIGEST_CHECK_UKDIGEST6  
Carry out digest check on UKDIGEST6 in User Key segment
(UK2)

DIGEST_CHECK_UPERM  
Carry out digest check on UFS Permanent lock (UPERM)
segment

DIGEST_CHECK_SYS  
Carry out digest check on Factory and Factory Key Segments

</div>


# Functions

 ---------------- 
<a name="sysinit"></a>
## SYS_init
<a name="prototype"></a>
### Prototype 

<div id="Functions$SYS_init$prototype" data-type="code">

    void
    SYS_init
    (
        uint32_t base_addr
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$SYS_init$description" data-type="text">

The function SYS_init() is used to initialize the internal data structures of
this driver. Currently this function is empty.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### base_addr
<div id="Functions$SYS_init$description$parameters$base_addr" data-type="text" data-name="base_addr">

The base_addr parameter specifies the base address of the PF_System_services
core.


</div>

<a name="return"></a>
### Return
<div id="Functions$SYS_init$description$return" data-type="text">

This function does not return a value.


</div>


 -------------------------------- 
<a name="sysgetserialnumber"></a>
## SYS_get_serial_number
<a name="prototype"></a>
### Prototype 

<div id="Functions$SYS_get_serial_number$prototype" data-type="code">

    uint8_t
    SYS_get_serial_number
    (
        const uint8_t * p_serial_number,
        uint16_t mb_offset
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$SYS_get_serial_number$description" data-type="text">

The function SYS_get_serial_number() is used to execute "serial number" system
service.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### p_serial_number
<div id="Functions$SYS_get_serial_number$description$parameters$p_serial_number" data-type="text" data-name="p_serial_number">

The p_serial_number parameter is a pointer to a buffer in which the data
returned by system controller is copied.


</div>

#### mb_offset
<div id="Functions$SYS_get_serial_number$description$parameters$mb_offset" data-type="text" data-name="mb_offset">

The mb_offset parameter specifies the offset from the start of Mailbox where the
data related to this service is available. Note that all accesses to the mailbox
are of word length (4 bytes). Value '10' of this parameter means that the data
access area for this service starts from 11th word (offset 10) in the Mailbox.


</div>

<a name="return"></a>
### Return
<div id="Functions$SYS_get_serial_number$description$return" data-type="text">

This function returns the status code returned by the system controller for this
service. A '0' status code means that the service was executed successfully.


</div>


 -------------------------------- 
<a name="sysgetusercode"></a>
## SYS_get_user_code
<a name="prototype"></a>
### Prototype 

<div id="Functions$SYS_get_user_code$prototype" data-type="code">

    uint8_t
    SYS_get_user_code
    (
        const uint8_t * p_user_code,
        uint16_t mb_offset
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$SYS_get_user_code$description" data-type="text">

The function SYS_get_user_code() is used to execute "USERCODE" system service.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### p_user_code
<div id="Functions$SYS_get_user_code$description$parameters$p_user_code" data-type="text" data-name="p_user_code">

The p_user_code parameter is a pointer to a buffer in which the data returned by
system controller is copied.


</div>

#### mb_offset
<div id="Functions$SYS_get_user_code$description$parameters$mb_offset" data-type="text" data-name="mb_offset">

The mb_offset parameter specifies the offset from the start of Mailbox where the
data related to this service is available. Note that all accesses to the mailbox
are of word length (4 bytes). Value '10' of this parameter means that the data
access area for this service starts from 11th word (offset 10) in the Mailbox.


</div>

<a name="return"></a>
### Return
<div id="Functions$SYS_get_user_code$description$return" data-type="text">

This function returns the status code returned by the system controller for this
service. A '0' status code means that the service was executed successfully.


</div>


 -------------------------------- 
<a name="sysgetdesigninfo"></a>
## SYS_get_design_info
<a name="prototype"></a>
### Prototype 

<div id="Functions$SYS_get_design_info$prototype" data-type="code">

    uint8_t
    SYS_get_design_info
    (
        const uint8_t * p_design_info,
        uint16_t mb_offset
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$SYS_get_design_info$description" data-type="text">

The function SYS_get_design_info() is used to execute "Get Design Info" system
service.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### p_design_info
<div id="Functions$SYS_get_design_info$description$parameters$p_design_info" data-type="text" data-name="p_design_info">

The p_design_info parameter is a pointer to a buffer in which the data returned
by system controller is copied. Total size of debug information is 36 bytes. The
data from the system controller includes the 256-bit user-defined design ID,
16-bit design version, and 16-bit design back level.


</div>

#### mb_offset
<div id="Functions$SYS_get_design_info$description$parameters$mb_offset" data-type="text" data-name="mb_offset">

The mb_offset parameter specifies the offset from the start of Mailbox where the
data related to this service is available. Note that all accesses to the mailbox
are of word length (4 bytes). Value '10' of this parameter means that the data
access area for this service starts from 11th word (offset 10) in the Mailbox.


</div>

<a name="return"></a>
### Return
<div id="Functions$SYS_get_design_info$description$return" data-type="text">

This function returns the status code returned by the system controller for this
service. A '0' status code means that the service was executed successfully.


</div>


 -------------------------------- 
<a name="sysgetdevicecertificate"></a>
## SYS_get_device_certificate
<a name="prototype"></a>
### Prototype 

<div id="Functions$SYS_get_device_certificate$prototype" data-type="code">

    uint8_t
    SYS_get_device_certificate
    (
        const uint8_t * p_device_certificate,
        uint16_t mb_offset
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$SYS_get_device_certificate$description" data-type="text">

The function SYS_get_device_certificate() is used to execute "Get Device
Certificate" system service.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### p_device_certificate
<div id="Functions$SYS_get_device_certificate$description$parameters$p_device_certificate" data-type="text" data-name="p_device_certificate">

The p_device_certificate parameter is a pointer to a buffer in which the data
returned by the system controller is copied.


</div>

#### mb_offset
<div id="Functions$SYS_get_device_certificate$description$parameters$mb_offset" data-type="text" data-name="mb_offset">

The mb_offset parameter specifies the offset from the start of Mailbox where the
data related to this service is available. Note that all accesses to the mailbox
are of word length (4 bytes). Value '10' of this parameter means that the data
access area for this service starts from 11th word (offset 10) in the Mailbox.


</div>

<a name="return"></a>
### Return
<div id="Functions$SYS_get_device_certificate$description$return" data-type="text">

This function returns the status code returned by the system controller for this
service. A '0' status code means that the service was executed successfully.


</div>


 -------------------------------- 
<a name="sysreaddigest"></a>
## SYS_read_digest
<a name="prototype"></a>
### Prototype 

<div id="Functions$SYS_read_digest$prototype" data-type="code">

    uint8_t
    SYS_read_digest
    (
        const uint8_t * p_digest,
        uint16_t mb_offset
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$SYS_read_digest$description" data-type="text">

The function SYS_read_digest() is used to execute "Read Digest" system service.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### p_digest
<div id="Functions$SYS_read_digest$description$parameters$p_digest" data-type="text" data-name="p_digest">

The p_digest parameter is a pointer to a buffer in which the data returned by
system controller is copied.


</div>

#### mb_offset
<div id="Functions$SYS_read_digest$description$parameters$mb_offset" data-type="text" data-name="mb_offset">

The mb_offset parameter specifies the offset from the start of Mailbox where the
data related to this service is available. Note that all accesses to the mailbox
are of word length (4 bytes). Value '10' of this parameter means that the data
access area for this service starts from 11th word (offset 10) in the Mailbox.


</div>

<a name="return"></a>
### Return
<div id="Functions$SYS_read_digest$description$return" data-type="text">

This function returns the status code returned by the system controller for this
service. A '0' status code means that the service was executed successfully.


</div>


 -------------------------------- 
<a name="sysquerysecurity"></a>
## SYS_query_security
<a name="prototype"></a>
### Prototype 

<div id="Functions$SYS_query_security$prototype" data-type="code">

    uint8_t
    SYS_query_security
    (
        uint8_t * p_security_locks,
        uint16_t mb_offset
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$SYS_query_security$description" data-type="text">

The function SYS_query_security() is used to execute "Query Security" system
service.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### p_security_locks
<div id="Functions$SYS_query_security$description$parameters$p_security_locks" data-type="text" data-name="p_security_locks">

The p_security_locks parameter is a pointer to a buffer in which the data
returned by system controller is copied.


</div>

#### mb_offset
<div id="Functions$SYS_query_security$description$parameters$mb_offset" data-type="text" data-name="mb_offset">

The mb_offset parameter specifies the offset from the start of Mailbox where the
data related to this service is available. Note that all accesses to the mailbox
are of word length (4 bytes). Value '10' of this parameter means that the data
access area for this service starts from 11th word (offset 10) in the Mailbox.


</div>

<a name="return"></a>
### Return
<div id="Functions$SYS_query_security$description$return" data-type="text">

This function returns the status code returned by the system controller for this
service. A '0' status code means that the service was executed successfully.


</div>


 -------------------------------- 
<a name="sysreaddebuginfo"></a>
## SYS_read_debug_info
<a name="prototype"></a>
### Prototype 

<div id="Functions$SYS_read_debug_info$prototype" data-type="code">

    uint8_t
    SYS_read_debug_info
    (
        const uint8_t * p_debug_info,
        uint16_t mb_offset
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$SYS_read_debug_info$description" data-type="text">

The function SYS_read_debug_info() is used to execute "Read Debug info" system
service.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### p_debug_info
<div id="Functions$SYS_read_debug_info$description$parameters$p_debug_info" data-type="text" data-name="p_debug_info">

The p_debug_info parameter is a pointer to a buffer in which the data returned
by system controller is copied.


</div>

#### mb_offset
<div id="Functions$SYS_read_debug_info$description$parameters$mb_offset" data-type="text" data-name="mb_offset">

The mb_offset parameter specifies the offset from the start of Mailbox where the
data related to this service is available. Note that all accesses to the mailbox
are of word length (4 bytes). Value '10' of this parameter means that the data
access area for this service starts from 11th word (offset 10) in the Mailbox.


</div>

<a name="return"></a>
### Return
<div id="Functions$SYS_read_debug_info$description$return" data-type="text">

This function returns the status code returned by the system controller for this
service. A '0' status code means that the service was executed successfully.


</div>


 -------------------------------- 
<a name="sysreadenvmparameter"></a>
## SYS_read_envm_parameter
<a name="prototype"></a>
### Prototype 

<div id="Functions$SYS_read_envm_parameter$prototype" data-type="code">

    uint8_t
    SYS_read_envm_parameter
    (
        uint8_t * p_envm_param,
        uint16_t mb_offset
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$SYS_read_envm_parameter$description" data-type="text">

The function SYS_read_envm_parameter() is used to retrieve all parameters needed
for the eNVM operation and programming.

NOTE: This service is available only on PolarFire SoC Platform. This service is
not yet supported by PF_SYSTEM_SERVICES 3.0.100.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### p_envm_param
<div id="Functions$SYS_read_envm_parameter$description$parameters$p_envm_param" data-type="text" data-name="p_envm_param">

The p_envm_param parameter is a pointer to a buffer in which the data returned
by system controller is copied. This buffer stores all the eNVM parameters.


</div>

#### mb_offset
<div id="Functions$SYS_read_envm_parameter$description$parameters$mb_offset" data-type="text" data-name="mb_offset">

The mb_offset parameter specifies the offset from the start of Mailbox where the
data related to this service is available. Note that all accesses to the mailbox
are of word length (4 bytes). Value '10' of this parameter means that the data
access area for this service starts from 11th word (offset 10) in the Mailbox.


</div>

<a name="return"></a>
### Return
<div id="Functions$SYS_read_envm_parameter$description$return" data-type="text">

The SYS_read_envm_parameter service will return zero if the service executed
successfully, otherwise, it will return one indicating error.


</div>


 -------------------------------- 
<a name="syspufemulationservice"></a>
## SYS_puf_emulation_service
<a name="prototype"></a>
### Prototype 

<div id="Functions$SYS_puf_emulation_service$prototype" data-type="code">

    uint8_t
    SYS_puf_emulation_service
    (
        const uint8_t * p_challenge,
        uint8_t op_type,
        uint8_t * p_response,
        uint16_t mb_offset
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$SYS_puf_emulation_service$description" data-type="text">

The function SYS_puf_emulation_service() is used to authenticate a device.

The SYS_puf_emulation_service() function accept a challenge comprising a 8-bit
optype and 128-bit challenge and return a 256-bit response unique to the given
challenge and the device.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### p_challenge
<div id="Functions$SYS_puf_emulation_service$description$parameters$p_challenge" data-type="text" data-name="p_challenge">

The p_challenge parameter specifies the 128-bit challenge to generate the
256-bits unique response.


</div>

#### op_type
<div id="Functions$SYS_puf_emulation_service$description$parameters$op_type" data-type="text" data-name="op_type">

The op_type parameter specifies the operational parameter to generate the
256-bits unique response.


</div>

#### p_response
<div id="Functions$SYS_puf_emulation_service$description$parameters$p_response" data-type="text" data-name="p_response">

The p_response parameter is a pointer to a buffer where the data returned which
is the response by system controller is copied.


</div>

#### mb_offset
<div id="Functions$SYS_puf_emulation_service$description$parameters$mb_offset" data-type="text" data-name="mb_offset">

The mb_offset parameter specifies the offset from the start of Mailbox where the
data related to this service is available. Note that all accesses to the mailbox
are of word length (4 bytes). Value '10' of this parameter means that the data
access area for this service starts from 11th word (offset 10) in the Mailbox.


</div>

<a name="return"></a>
### Return
<div id="Functions$SYS_puf_emulation_service$description$return" data-type="text">

The SYS_puf_emulation_service function will return zero if the service executed
successfully, otherwise, it will return one indicating error.


</div>


 -------------------------------- 
<a name="sysdigitalsignatureservice"></a>
## SYS_digital_signature_service
<a name="prototype"></a>
### Prototype 

<div id="Functions$SYS_digital_signature_service$prototype" data-type="code">

    uint8_t
    SYS_digital_signature_service
    (
        const uint8_t * p_hash,
        uint8_t format,
        uint8_t * p_response,
        uint16_t mb_offset
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$SYS_digital_signature_service$description" data-type="text">

The SYS_digital_signature_service() function is used to generate P-384 ECDSA
signature based on SHA384 hash value.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### p_hash
<div id="Functions$SYS_digital_signature_service$description$parameters$p_hash" data-type="text" data-name="p_hash">

The p_hash parameter is a pointer to the buffer which contain the 48 bytes
SHA384 Hash value (input value).


</div>

#### format
<div id="Functions$SYS_digital_signature_service$description$parameters$format" data-type="text" data-name="format">

The format parameter specifies the output format of generated SIGNATURE field.
The different types of output signature formats are as follow:

  - DIGITAL_SIGNATURE_RAW_FORMAT
  - DIGITAL_SIGNATURE_DER_FORMAT


</div>

#### p_response
<div id="Functions$SYS_digital_signature_service$description$parameters$p_response" data-type="text" data-name="p_response">

The p_response parameter is a pointer to a buffer that contains the generated
ECDSA signature. The field may be 96 bytes or 104 bytes depending upon the
output format.


</div>

#### mb_offset
<div id="Functions$SYS_digital_signature_service$description$parameters$mb_offset" data-type="text" data-name="mb_offset">

The mb_offset parameter specifies the offset from the start of Mailbox where the
data related to this service is available. Note that all accesses to the mailbox
are of word length (4 bytes). Value '10' of this parameter means that the data
access area for this service starts from 11th word (offset 10) in the Mailbox.


</div>

<a name="return"></a>
### Return
<div id="Functions$SYS_digital_signature_service$description$return" data-type="text">

The SYS_digital_signature_service function returns zero if the service executed
successfully, otherwise, it returns non-zero values indicating error.


</div>


 -------------------------------- 
<a name="syssecurenvmwrite"></a>
## SYS_secure_nvm_write
<a name="prototype"></a>
### Prototype 

<div id="Functions$SYS_secure_nvm_write$prototype" data-type="code">

    uint8_t
    SYS_secure_nvm_write
    (
        uint8_t format,
        uint8_t snvm_module,
        const uint8_t * p_data,
        const uint8_t * p_user_key,
        uint16_t mb_offset
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$SYS_secure_nvm_write$description" data-type="text">

The SYS_secure_nvm_write() function writes data in the sNVM region. Data gets
stored in the following format:

  - Non-authenticated plaintext
  - Authenticated plaintext
  - Authenticated ciphertext

Note: If you are executing this function with Authenticated plaintext or
Authenticated ciphertext on a device whose sNVM was never previously written to,
then the service may fail. For it to work, you must first write Authenticated
data to the sNVM using Libero along with USK client and custom security. This
flow generates the SMK. See UG0753 PolarFire FPGA Security User Guide for
further details.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### format
<div id="Functions$SYS_secure_nvm_write$description$parameters$format" data-type="text" data-name="format">

The format parameter specifies the format used to write data in sNVM region. The
different type of text formats are as follow:

  - NON_AUTHENTICATED_PLAINTEXT_FORMAT
  - AUTHENTICATED_PLAINTEXT_FORMAT
  - AUTHENTICATED_CIPHERTEXT_FORMAT


</div>

#### snvm_module
<div id="Functions$SYS_secure_nvm_write$description$parameters$snvm_module" data-type="text" data-name="snvm_module">

The snvm_module parameter specifies the the sNVM module in which the data need
to be written.


</div>

#### p_data
<div id="Functions$SYS_secure_nvm_write$description$parameters$p_data" data-type="text" data-name="p_data">

The p_data parameter is a pointer to a buffer which contains the data to be
stored in sNVM region. The data length to be written is fixed depending on the
format parameter. If NON_AUTHENTICATED_PLAINTEXT_FORMAT is selected, then you
can write 252 bytes in the sNVM module. For other two formats the data length is
236 bytes.


</div>

#### p_user_key
<div id="Functions$SYS_secure_nvm_write$description$parameters$p_user_key" data-type="text" data-name="p_user_key">

The p_user_key parameter is a pointer to a buffer which contain the 96-bit key
USK (user secret key). This user secret key will enhance the security when
authentication is used. That is, when Authenticated plaintext and Authenticated
ciphertext format is selected.


</div>

#### mb_offset
<div id="Functions$SYS_secure_nvm_write$description$parameters$mb_offset" data-type="text" data-name="mb_offset">

The mb_offset parameter specifies the offset from the start of Mailbox where the
data related to this service is available. Note that all accesses to the mailbox
are of word length (4 bytes). Value '10' of this parameter means that the data
access area for this service starts from 11th word (offset 10) in the Mailbox.


</div>

<a name="return"></a>
### Return
<div id="Functions$SYS_secure_nvm_write$description$return" data-type="text">

The SYS_digital_signature_service function returns zero if the service executed
successfully, otherwise, it returns non-zero values indicating error.


</div>


 -------------------------------- 
<a name="syssecurenvmread"></a>
## SYS_secure_nvm_read
<a name="prototype"></a>
### Prototype 

<div id="Functions$SYS_secure_nvm_read$prototype" data-type="code">

    uint8_t
    SYS_secure_nvm_read
    (
        uint8_t snvm_module,
        const uint8_t * p_user_key,
        uint8_t * p_admin,
        uint8_t * p_data,
        uint16_t data_len,
        uint16_t mb_offset
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$SYS_secure_nvm_read$description" data-type="text">

The SYS_secure_nvm_read() function is used to read data present in sNVM region.
User should provide USK key, if the data was programmed using authentication. If
the data was written in the sNVM using the authenticated plaintext or the
authenticated ciphertext service option then this service will return the valid
data only when authentication is successful. For more details, see
SYS_secure_nvm_write() function. If the data was written in the sNVM using the
authenticated plaintext or the authenticated ciphertext service option then this
service will return the valid data only when authentication is successful. For
more details, see SYS_secure_nvm_write() function and its parameter description.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### snvm_module
<div id="Functions$SYS_secure_nvm_read$description$parameters$snvm_module" data-type="text" data-name="snvm_module">

The snvm_module parameter specifies the sNVM module from which the data need to
be read.


</div>

#### p_user_key
<div id="Functions$SYS_secure_nvm_read$description$parameters$p_user_key" data-type="text" data-name="p_user_key">

The p_user_key parameter is a pointer to a buffer which contain the 96-bit key
USK (user secret key). User should provide same secret key which is previously
used for authentication while writing data in sNVM region.


</div>

#### p_admin
<div id="Functions$SYS_secure_nvm_read$description$parameters$p_admin" data-type="text" data-name="p_admin">

The p_admin parameter is a pointer to the buffer where the output page admin
data is stored. The page admin data is 4 bytes long.


</div>

#### p_data
<div id="Functions$SYS_secure_nvm_read$description$parameters$p_data" data-type="text" data-name="p_data">

The p_data parameter is a pointer to a buffer which contains the data read from
sNVM region. User should provide the buffer large enough to store the read data.


</div>

#### data_len
<div id="Functions$SYS_secure_nvm_read$description$parameters$data_len" data-type="text" data-name="data_len">

The data_len parameter specifies the number of bytes to be read from sNVM. The
application should know whether the data written in the chose sNVM module was
previously stored using Authentication or not. The data_len should be 236 bytes,
for authenticated data. For not authenticated data the data_len should be 252
bytes.


</div>

#### mb_offset
<div id="Functions$SYS_secure_nvm_read$description$parameters$mb_offset" data-type="text" data-name="mb_offset">

The mb_offset parameter specifies the offset from the start of Mailbox where the
data related to this service is available. Note that all accesses to the mailbox
are of word length (4 bytes). Value '10' of this parameter means that the data
access area for this service starts from 11th word (offset 10) in the Mailbox.


</div>

<a name="return"></a>
### Return
<div id="Functions$SYS_secure_nvm_read$description$return" data-type="text">

The SYS_digital_signature_service function returns zero if the service executed
successfully, otherwise, it returns non-zero values indicating error.


</div>


 -------------------------------- 
<a name="sysnonceservice"></a>
## SYS_nonce_service
<a name="prototype"></a>
### Prototype 

<div id="Functions$SYS_nonce_service$prototype" data-type="code">

    uint8_t
    SYS_nonce_service
    (
        const uint8_t * p_nonce,
        uint16_t mb_offset
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$SYS_nonce_service$description" data-type="text">

The function SYS_nonce_service() is used to issue "Nonce Service" system service
to the system controller.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### p_nonce
<div id="Functions$SYS_nonce_service$description$parameters$p_nonce" data-type="text" data-name="p_nonce">

The p_nonce parameter is a pointer to a buffer in which the data returned by
system controller is copied.


</div>

#### mb_offset
<div id="Functions$SYS_nonce_service$description$parameters$mb_offset" data-type="text" data-name="mb_offset">

The mb_offset parameter specifies the offset from the start of Mailbox where the
data related to this service is available. Note that all accesses to the mailbox
are of word length (4 bytes). Value '10' of this parameter means that the data
access area for this service starts from 11th word (offset 10) in the Mailbox.


</div>

<a name="return"></a>
### Return
<div id="Functions$SYS_nonce_service$description$return" data-type="text">

This function returns the status code returned by the system controller for this
service. A '0' status code means that the service was executed successfully and
a non-zero value indicates error. See the document link provided in the theory
of operation section to know more about the service and service response.


</div>


 -------------------------------- 
<a name="sysbitstreamauthenticateservice"></a>
## SYS_bitstream_authenticate_service
<a name="prototype"></a>
### Prototype 

<div id="Functions$SYS_bitstream_authenticate_service$prototype" data-type="code">

    uint8_t
    SYS_bitstream_authenticate_service
    (
        uint32_t spi_flash_address,
        uint16_t mb_offset
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$SYS_bitstream_authenticate_service$description" data-type="text">

The SYS_bitstream_authenticate_service() function is used to authenticate the
Bitstream which is located in SPI through a system service routine. Prior to
using the IAP service, it may be required to first validate the new bitstream
before committing the device to reprogramming, thus avoiding the need to invoke
recovery procedures if the bitstream is invalid.

This service is applicable to bitstreams stored in SPI Flash memory only.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### spi_flash_address
<div id="Functions$SYS_bitstream_authenticate_service$description$parameters$spi_flash_address" data-type="text" data-name="spi_flash_address">

The spi_flash_address parameter specifies the address within SPI Flash memory
where the bit-stream is stored.


</div>

#### mb_offset
<div id="Functions$SYS_bitstream_authenticate_service$description$parameters$mb_offset" data-type="text" data-name="mb_offset">

The mb_offset parameter specifies the offset from the start of Mailbox where the
data related to this service is available. Note that all accesses to the mailbox
are of word length (4 bytes). Value '10' of this parameter means that the data
access area for this service starts from 11th word (offset 10) in the Mailbox.


</div>

<a name="return"></a>
### Return
<div id="Functions$SYS_bitstream_authenticate_service$description$return" data-type="text">

The SYS_bitstream_authenticate_service function will return zero if the service
executed successfully and the non-zero response from system controller indicates
error. See the document link provided in the theory of operation section to know
more about the service and service response.


</div>


 -------------------------------- 
<a name="sysiapimageauthenticateservice"></a>
## SYS_IAP_image_authenticate_service
<a name="prototype"></a>
### Prototype 

<div id="Functions$SYS_IAP_image_authenticate_service$prototype" data-type="code">

    uint8_t
    SYS_IAP_image_authenticate_service
    (
        uint8_t spi_idx
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$SYS_IAP_image_authenticate_service$description" data-type="text">

The SYS_IAP_image_authenticate_service() function is used to authenticate the
IAP image which is located in SPI through a system service routine. The service
checks the image descriptor and the referenced bitstream and optional
initialization data. If the image is authenticated successfully, then the image
is guaranteed to be valid when used by an IAP function.

This service is applicable to bitstreams stored in SPI Flash memory only.

</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### spi_idx
<div id="Functions$SYS_IAP_image_authenticate_service$description$parameters$spi_idx" data-type="text" data-name="spi_idx">

The spi_idx parameter specifies the index in the SPI directory to be used where
the IAP bit-stream is stored.  
Note: To support recovery SPI_IDX=1 should be an
empty slot and the recovery image should be located in SPI_IDX=0. Since
SPI_IDX=1 should be an empty slot, it shouldn’t be passed into the system
service.


</div>

<a name="return"></a>
### Return
<div id="Functions$SYS_IAP_image_authenticate_service$description$return" data-type="text">

The SYS_IAP_image_authenticate_service function will return zero if the service
executed successfully the non-zero response from system controller indicates
error. Please refer to the document link provided in the theory of operation
section to know more about the service and service response.


</div>


 -------------------------------- 
<a name="sysdigestcheckservice"></a>
## SYS_digest_check_service
<a name="prototype"></a>
### Prototype 

<div id="Functions$SYS_digest_check_service$prototype" data-type="code">

    uint8_t
    SYS_digest_check_service
    (
        uint32_t options,
        uint16_t mb_offset
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$SYS_digest_check_service$description" data-type="text">

The SYS_digest_check_service() function is used to Recalculates and compares
digests of selected non-volatile memories. If the fabric digest is to be
checked, then the user design must follow all prerequisite steps for the
FlashFreeze service before invoking this service.  
This service is applicable
to bitstreams stored in SPI Flash memory only.


</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### options
<div id="Functions$SYS_digest_check_service$description$parameters$options" data-type="text" data-name="options">

The options parameter specifies the digest check options which indicate the area
on which the digest check should be performed. Below is the list of options. You
can OR these options to indicate to perform digest check on multiple segments.
Note: The options parameter is of 2 bytes when used with PF device and 4 bytes
when used with PolarFire SoC device.

| Options[i]    | Description     | 
| -----|-----|
| 0x01    | Fabric digest     | 
| 0x02    | Fabric Configuration (CC) segment     | 
| 0x04    | ROM digest in SNVM segment     | 
| 0x08    | UL segment     | 
| 0x10    | UKDIGEST0 in User Key segment     | 
| 0x20    | UKDIGEST1 in User Key segment     | 
| 0x40    | UKDIGEST2 in User Key segment (UPK1)     | 
| 0x80    | UKDIGEST3 in User Key segment (UK1)     | 
| 0x100    | UKDIGEST4 in User Key segment (DPK)     | 
| 0x200    | UKDIGEST5 in User Key segment (UPK2)     | 
| 0x400    | UKDIGEST6 in User Key segment (UK2)     | 
| 0x800    | UFS Permanent lock (UPERM) segment     | 
| 0x1000    | Factory and Factory Key Segments.     | 
| 0x2000    | UKDIGEST7 in User Key segment (HWM) (PFSoC)     | 
| 0x4000    | ENVMDIGEST (PFSoC only)     | 
| 0x8000    | UKDIGEST8 for MSS Boot Info (PFSoC only)     | 
| 0x10000    | SNVM_RW_ACCESS_MAP Digest (PFSoC only)     | 
| 0x20000    | SBIC revocation digest (PFSoC only)    | 


</div>

#### mb_offset
<div id="Functions$SYS_digest_check_service$description$parameters$mb_offset" data-type="text" data-name="mb_offset">

The mb_offset parameter specifies the offset from the start of Mailbox where the
data related to this service is available. Note that all accesses to the mailbox
are of word length (4 bytes). Value '10' of this parameter means that the data
access area for this service starts from 11th word (offset 10) in the Mailbox.


</div>

<a name="return"></a>
### Return
<div id="Functions$SYS_digest_check_service$description$return" data-type="text">

The SYS_digest_check_service function will return zero if the service executed
successfully the non-zero response from system controller indicates error.
Pleaes refer to the document link provided in the theory of operation section to
know more about the service and service response.


</div>


 -------------------------------- 
<a name="sysiapservice"></a>
## SYS_iap_service
<a name="prototype"></a>
### Prototype 

<div id="Functions$SYS_iap_service$prototype" data-type="code">

    uint8_t
    SYS_iap_service
    (
        uint8_t iap_cmd,
        uint32_t spiaddr,
        uint16_t mb_offset
    );


</div>

<a name="description"></a>
### Description

<div id="Functions$SYS_iap_service$description" data-type="text">

The SYS_iap_service() function is used to IAP service. The IAP service allows
the user to reprogram the device without the need for an external master. The
user design writes the bitstream to be programmed into a SPI Flash connected to
the SPI port. When the service is invoked, the System Controller automatically
reads the bitstream from the SPI flash and programs the device. The service
allows the image to be executed in either VERIFY or PROGRAM modes. Another
option for IAP is to perform the auto-update sequence. In this case the newest
image of the first two images in the SPI directory is chosen to be programmed.


</div>


 --------------------------- 

<a name="parameters"></a>
### Parameters
#### iap_cmd
<div id="Functions$SYS_iap_service$description$parameters$iap_cmd" data-type="text" data-name="iap_cmd">

The iap_cmd parameter specifies the specific IAP command which depends upon
VERIFY or PROGRAM modes and the SPI address method.

| iap_cmd    | Description     | 
| -----|-----|
| IAP_PROGRAM_BY_SPIIDX_CMD    | IAP program.     | 
| IAP_VERIFY_BY_SPIIDX_CMD    | Fabric Configuration (CC) segment     | 
| IAP_PROGRAM_BY_SPIADDR_CMD    | ROM digest in SNVM segment     | 
| IAP_VERIFY_BY_SPIADDR_CMD    | UL segment     | 
| IAP_AUTOUPDATE_CMD    | UKDIGEST0 in User Key segment    | 


</div>

#### spiaddr
<div id="Functions$SYS_iap_service$description$parameters$spiaddr" data-type="text" data-name="spiaddr">

The spiaddr parameter specifies either the index in the SPI directory or the SPI
address in the SPI Flash memory. Below is the list of the possible meaning of
spiaddr parameter in accordance with the iap_cmd parameter.

| iap_cmd    | spiaddr     | 
| -----|-----|
| IAP_PROGRAM_BY_SPIIDX_CMD    | Index in the SPI directory.     | 
| IAP_VERIFY_BY_SPIIDX_CMD    | Index in the SPI directory.     | 
| IAP_PROGRAM_BY_SPIADDR_CMD    | SPI address in the SPI Flash memory     | 
| IAP_VERIFY_BY_SPIADDR_CMD    | SPI address in the SPI Flash memory     | 
| IAP_AUTOUPDATE_CMD    | spiaddr is ignored as No index/address required for this command.    | 


</div>

#### mb_offset
<div id="Functions$SYS_iap_service$description$parameters$mb_offset" data-type="text" data-name="mb_offset">

The mb_offset parameter specifies the offset from the start of Mailbox where the
data related to this service is available. Note that all accesses to the mailbox
are of word length (4 bytes). Value '10' of this parameter means that the data
access area for this service starts from 11th word (offset 10) in the Mailbox.
Note: For the IAP services with command IAP_PROGRAM_BY_SPIIDX_CMD and
IAP_VERIFY_BY_SPIIDX_CMD To support recovery SPI_IDX=1 should be an empty slot
and the recovery image should be located in SPI_IDX=0. Since SPI_IDX=1 should be
an empty slot it shouldn’t be passed into the system service.


</div>

<a name="return"></a>
### Return
<div id="Functions$SYS_iap_service$description$return" data-type="text">

The SYS_iap_service function will return zero if the service executed
successfully and the non-zero response from system controller indicates error.
Please refer to the document link provided in the theory of operation section to
know more about the service and service response.


</div>


 -------------------------------- 
