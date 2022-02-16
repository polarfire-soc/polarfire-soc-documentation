# Working with PolarFire SoC USB Linux driver

- [Working with PolarFire SoC USB Linux driver](#working-with-polarfire-soc-usb-linux-driver)
  - [Jumper Settings](#jumper-settings)
  - [Flash Drive](#flash-drive)
    - [Mounting the Flash Drive](#mounting-the-flash-drive)
    - [Accessing the Flash Drive](#accessing-the-flash-drive)
    - [Verify the Flash Drive](#verify-the-flash-drive)
    - [Disconnect the Flash Drive](#disconnect-the-flash-drive)
  - [Working with a Webcam](#working-with-a-webcam)
    - [Accessing a webcam](#accessing-a-webcam)
  - [MSS USB hardware block configurations](#mss-usb-hardware-block-configurations)
  - [Known Issues](#known-issues)

The PolarFire SoC FPGA devices contain one instance of a Universal Serial Bus (USB) controller, which is a multipoint dual-role On-The-Go (OTG) controller. This USB controller, which is implemented as a hard ASIC block, complies with the USB 2.0 standard with OTG support.

This document describes the steps to test the USB host on the Icicle kit using a flash drive and a webcam. It also describes the required hardware configuration for the USB controller and some known issues.

The PolarFire SoC USB Linux driver is available with [yocto](https://github.com/polarfire-soc/meta-polarfire-soc-yocto-bsp) and [buildroot](https://github.com/polarfire-soc/polarfire-soc-buildroot-sdk) from the Microchip Public GIT server under polarfire-soc.

<a name="jumper-settings"></a>

## Jumper Settings

Before power up, make sure that Icicle Kit jumpers J15 and J17 are closed to use PolarFire SoC as a USB Host.

Note: The default jumper configuration for the Icicle Kit, shown [here](https://github.com/polarfire-soc/polarfire-soc-documentation/blob/master/boards/mpfs-icicle-kit-es/updating-icicle-kit/updating-icicle-kit-design-and-linux.md), requires J15 and J17 to be open to act as a USB device. To program the eMMC again, open jumpers J15 and J17.

<a name="flash-drive"></a>

## Flash Drive

After Linux has booted up on the Icicle Kit, connect a USB flash drive into the micro USB connector J16. The flash drive will then be enumerated. Log messages about the enumeration process can be retrieved using the following command:

```sh
$ dmesg | tail -10
```

Example USB enumeration log:

```text
[   35.112927] usb 1-1: new high-speed USB device number 2 using musb-hdrc
[   35.294671] usb-storage 1-1:1.0: USB Mass Storage device detected
[   35.295826] scsi host0: usb-storage 1-1:1.0
[   36.364385] scsi 0:0:0:0: Direct-Access     SanDisk  Cruzer Blade     1.26 PQ: 0 ANSI: 6
[   36.371826] sd 0:0:0:0: [sda] 7821312 512-byte logical blocks: (4.00 GB/3.73 GiB)
[   36.372789] sd 0:0:0:0: [sda] Write Protect is off
[   36.373084] sd 0:0:0:0: [sda] Mode Sense: 43 00 00 00
[   36.373804] sd 0:0:0:0: [sda] Write cache: disabled, read cache: enabled, doesn't support DPO or FUA
[   36.467142] sd 0:0:0:0: [sda] Attached SCSI removable disk
[   36.461102]  sda: sda1 sda2
```

<a name="mounting-the-flash-drive"></a>

### Mounting the Flash Drive
The flash drive can be mounted in Linux using the following method

Create a directory such as usbmsc inside media. This will be the mount point for the flash drive.

```sh
$ mkdir /media/usbmsc
```

Idenitfy the flash drive on the Icicle kit, use dmesg to check what the drive identifier for the flash drive is.

```sh
$ dmesg | egrep "sd"
```

The output should contain a line similar to one of the following lines:

```text
[    4.013197] sd 0:0:0:0: [sda] 7821312 512-byte logical blocks: (4.00 GB/3.73 GiB)
[    4.124154]  sda: sda1 sda2 sda3
[    4.137143] sd 0:0:0:0: [sda] Attached SCSI removable disk
```

sdX is the drive identifier that should be used in the following commands, where X should be replaced with the specific character from the output of the previous command.
For these examples the identifier sdX is used.

WARNING:

    The drive with the identifier `sda` is the default location for your operating system.  
    DO NOT pass this identifier to any of the commands listed here without being absolutely sure that your OS is not located here.  
    Check that the size of the card matches the dmesg output before continuing.  

Once sure of the drive identifier, use the following command to mount the flash drive to the board, replacing the X as appropriate:

```sh
$ mount -t vfat /dev/sdX /media/usbmsc
```

<a name="accessing-the-flash-drive"></a>

### Accessing the Flash Drive
Change the current directory to Flash drive.

```sh
$ cd /media/usbmsc
```

List the content of the Flash drive:

```sh  
$ ls
```

<a name="verify-the-flash-drive"></a>

### Verify the Flash Drive
To verify the Flash drive, copy a file to it, unmount the drive.

```sh
$ cp /home/dummy.txt /media/usbmsc/
$ umount /media/usbmsc
```

Remove and reconnect the flash drive to the Icicle kit. Check the flash drive identifier (as mentioned in [Mounting the Flash Drive](#usb-device-mount)).
Once sure of the drive identifier, use the below command to mount the flash drive, replacing the X as appropriate.

```sh
$ mount -t vfat /dev/sdX /media/usbmsc
```

Next, read the dummy file from the flash drive and compare the stored file with the original to determine that the content is the same/correct using diff command.

```sh
$ cp /media/usbmsc/dummy.txt /home/dummy1.txt
$ diff /home/dummy.txt /home/dummy2.txt
```

The diff command should show nothing (means copy successfully).

<a name="disconnect-the-flash-drive"></a>

### Disconnect the Flash Drive

Issue the umount command to unmount the flash drive so it can safely be removed.

```sh
$ umount /media/usbmsc
```

<a name="working-with-a-webcam"></a>

## Working with a Webcam

To work with USB video devices, such as webcams, the following packages need to be included in the yocto or buildroot build systems.

- yavta \
- libuvc \
- gstd \
- gstreamer1.0-plugins-good \
- v4l-utils \

<a name="accessing-a-webcam"></a>

### Accessing a webcam

Use the below command to capture an image from a webcam

```sh
$ v4l2-ctl --device /dev/video0 --set-fmt-video=width=640,height=480,pixelformat=MJPG --stream-mmap=3 --stream-count=100 --stream-to=stream.vid
```

<a name="mss-usb-hardware-block-configurations"></a>

## MSS USB hardware block configurations

1. The MUSB code is configured to 5 transmit endpoints (TX EP) and 5 receive endpoints (RX EP),including control endpoint (EP0).
2. The USB FIFO size is fixed to 8Kbytes which is shared by all endpoints.
3. Support for high bandwidth isochronous (ISO) pipe is enabled endpoints for one endpoint i.e endpoint 4.

<a name="known-issues"></a>

## Known Issues

1. VBUS_ERROR in a_idle error.

    A VBUS_ERROR occurs when you boot Linux and then connect a USB device (e.g a webcam) directly to J16.
    Solution:
    Rather than connecting the webcam directly, connect a powered hub to J16 on the Icicle Kit, and then connect the webcam to the hub.
    To connect high power USB device, its is better to use an externally powered USB hub in between the USB device and the Icicle Kit.

2. If a VBUS_ERROR is displayed and LEDs turns off, a board power cycle will be needed to get the USB port working.
