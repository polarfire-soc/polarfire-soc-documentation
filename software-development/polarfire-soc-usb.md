# Working with PolarFire SoC Linux USB driver

- [Jumper setting](#jumper-setting)
- [Working with a USB Thumb Drive](#usb-thumb)
	- [Mounting the Thumb Drive](#usb-device-mount)
	- [Accessing thumb drive](#usb-device-access)
	- [Copy/Verify Files to/from the Thumb Drive](#usb-device-operations)
	- [Disconnect the Thumb Drive](#usb-device-umount)
- [Working with Webcamera](#usb-webcam)
     - [Accessing Webcam](#usb-webcam-access)
- [USB Specification](#usb-spec)
- [Known Issue](#issues)

The USB functionality is available with yocto and buildroot from the Microchip Public GIT server under polarfire-soc. 

GIT repository:

[https://github.com/polarfire-soc/polarfire-soc-buildroot-sdk#Loading-the-Image-onto-the-Target](https://github.com/polarfire-soc/meta-polarfire-soc-yocto-bsp)

[https://github.com/polarfire-soc/meta-polarfire-soc-yocto-bsp](https://github.com/polarfire-soc/meta-polarfire-soc-yocto-bsp)

For more information about git, please refer to Using Git.

## Jumper setting <a name="jumper-setting"></a>
Make sure the jumpers J15 and J17 are closed to use PolarFire SoC as USB Host

## Thumb Drive <div id="usb-thumb"/>
After Linux has boot up on ICICLE kit, plug the thumb drive into the USB connector J16. The thumb drive will be enumerated, 
a message shown below can be seen on the console when you do “dmesg | tail -10”
 - [   35.112927] usb 1-1: new high-speed USB device number 2 using musb-hdrc
 - [   35.294671] usb-storage 1-1:1.0: USB Mass Storage device detected
 - [   35.295826] scsi host0: usb-storage 1-1:1.0
 - [   36.364385] scsi 0:0:0:0: Direct-Access     SanDisk  Cruzer Blade     1.26 PQ: 0 ANSI: 6
 - [   36.371826] sd 0:0:0:0: [sda] 7821312 512-byte logical blocks: (4.00 GB/3.73 GiB)
 - [   36.372789] sd 0:0:0:0: [sda] Write Protect is off
 - [   36.373084] sd 0:0:0:0: [sda] Mode Sense: 43 00 00 00
 - [   36.373804] sd 0:0:0:0: [sda] Write cache: disabled, read cache: enabled, doesn't support DPO or FUA
 - [   36.461102]  sda: sda1 sda2
 - [   36.467142] sd 0:0:0:0: [sda] Attached SCSI removable disk
	

### Mounting the Thumb Drive  <div id="usb-device-mount"/>
The thumb driver should be mounted to Linux (on ICICLE kit) as follow:

1. Create a driectory such as usbmsc inside media.

		mkdir usbmsc
2. Mount the drive.

		mount -t vfat /dev/sdaX /media/usbmsc

### Accessing thumb drive  <div id="usb-device-access"/>
	Go to the Thumb Drive.
	Change the current directory to thumb drive.
		cd /media/usbmsc

    List the content of the thumb drive:
	
		ls

### Copy/Verify Files to/from the Thumb Drive <div id="usb-device-operations"/>

		cp /home/dummy.txt /media/usbmsc/
		
		umount /media/usbmsc
		
		mount -t vfat /dev/sdaX /media/usbmsc
		
		cp /media/usbmsc/dummy.txt /home/dummy1.txt
		
		diff /home/dummy.txt /home/dummy2.txt

   The diff command should show nothing (copy successful).
   
   Once sure of the drive identifier, use the above command to mount the thumb drive, replacing the X as appropriate:
   
### Disconnect the Thumb Drive  <div id="usb-device-umount"/>

   Isssue the umount command to disconnect thumb drive.
   
		umount /media/usbmsc
   
## Working with Webcamera  <div id="usb-webcam"/>
The following packages are added in Yocto buildroot-sdk

	 - yavta \ 
     - libuvc \
     - gstd \
     - gstreamer1.0-plugins-good \
     - v4l-utils \

### Accesing Webcam <div id="usb-webcam-access"/>
Use the below command to capture Image from Webcam

	v4l2-ctl --device /dev/video0 --set-fmt-video=width=640,height=480,pixelformat=MJPG --stream-mmap=3 --stream-count=100 --stream-to=stream.vid

## USB Specification: <div id="usb-spec"/>
1. The MUSB code is configured to 5 transmit endpoints (TX EP) and 5 receive endpoints (RX EP),including control endpoint (EP0).
2. The USB FIFO size is fixed to 8Kbytes which is shared by all endpoints.
3. Support for high bandwidth isochronous (ISO) pipe is enabled endpoints for one endpoint i.e endpoint 4.

## Known Issue:  <div id="issues"/>
1. VBUS_ERROR in a_idle error.

   VBUS_ERROR occurs when you boot the linux and then plug in the webcam(usb device).
   Solution:
   Rather than plugging the webcam in directly, first plug in a powered hub into the Icicle board, and then connect webcam into the hub.
   
   To connect high power device, its is better to use external powered USB hub in between USB device and ICICLE kit USB port.
   
2. If the VBUS_ERROR is displayed and LED turns off, then reboot is needed to get USB port working.
   