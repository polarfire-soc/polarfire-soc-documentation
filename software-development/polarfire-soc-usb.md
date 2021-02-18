# Working with PolarFire SoC Linux USB driver

- [Jumper setting](#jumper-setting)
- [Working with a USB Flash Drive](#usb-flash)
	- [Mounting the Flash Drive](#usb-device-mount)
	- [Accessing Flash drive](#usb-device-access)
	- [Verify the Flash Drive](#usb-device-operations)
	- [Disconnect the Flash Drive](#usb-device-umount)
- [Working with a Webcam](#usb-webcam)
     - [Accessing a webcam](#usb-webcam-access)
- [MSS USB hardware block configurations](#usb-spec)
- [Known Issue](#issues)

The USB functionality is available with [yocto](https://github.com/polarfire-soc/meta-polarfire-soc-yocto-bsp) and [buildroot](https://github.com/polarfire-soc/polarfire-soc-buildroot-sdk) from the Microchip Public GIT server under polarfire-soc. 

## Jumper setting <a name="jumper-setting"></a>
Make sure the jumpers J15 and J17 are closed to use PolarFire SoC as USB Host.

The default jumper configuration shown here requires J15 and J17 to be open to act as a USB device. In this case PolarFire SoC will act as a USB host which requires J15 and J17 to be closed.

To program the eMMC again, re-opening the jumpers J15 and J17.

## Flash Drive <div id="usb-flash"/>
After Linux has booted up on the Icicle Kit, connect a USB flash drive into the micro USB connector J16. The flash drive will then be enumerated,, 
a message shown below can be seen on the console when you do “dmesg | tail -10”.

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
	

### Mounting the Flash Drive  <div id="usb-device-mount"/>
The flash drive can be mounted in Linux using the following method

1. Create a driectory such as usbmsc inside media.

		mkdir /media/usbmsc
		
2. Mount the drive.

		mount -t vfat /dev/sdaX /media/usbmsc
		   
	Once sure of the drive identifier, use the above command to mount the flash drive, replacing the X as appropriate.

### Accessing Flash drive  <div id="usb-device-access"/>
	Change the current directory to Flash drive.
		cd /media/usbmsc

    List the content of the Flash drive:
	
		ls

### Verify the Flash Drive <div id="usb-device-operations"/>
To verify the Flash drive, copy a file to Flash drive. Read the file from Flash drive and compare with original file to determine the content is same/correct.
		cp /home/dummy.txt /media/usbmsc/
		
		umount /media/usbmsc
		
		mount -t vfat /dev/sd* /media/usbmsc
		
		cp /media/usbmsc/dummy.txt /home/dummy1.txt
		
		diff /home/dummy.txt /home/dummy2.txt

   The diff command should show nothing (means copy successfully).
   
   "*" -> Change the value after determining where the Flash drive mount.
  
### Disconnect the Flash Drive  <div id="usb-device-umount"/>

Issue the umount command to un mount the flash drive so it can safely be disconnected.
   
		umount /media/usbmsc
   
## Working with a Webcam  <div id="usb-webcam"/>
To work with USB video devices, such as webcams, the following packages need to be included in the yocto or buildroot build systems.

	 - yavta \ 
     - libuvc \
     - gstd \
     - gstreamer1.0-plugins-good \
     - v4l-utils \

### Accessing a webcam <div id="usb-webcam-access"/>
Use the command below to capture an image from a webcam

	v4l2-ctl --device /dev/video0 --set-fmt-video=width=640,height=480,pixelformat=MJPG --stream-mmap=3 --stream-count=100 --stream-to=stream.vid

## MSS USB hardware block configurations: <div id="usb-spec"/>
1. The MUSB code is configured to 5 transmit endpoints (TX EP) and 5 receive endpoints (RX EP),including control endpoint (EP0).
2. The USB FIFO size is fixed to 8Kbytes which is shared by all endpoints.
3. Support for high bandwidth isochronous (ISO) pipe is enabled endpoints for one endpoint i.e endpoint 4.

## Known Issues:  <div id="issues"/>
1. VBUS_ERROR in a_idle error.

   A VBUS_ERROR occurs when you boot Linux and then connect a USB device (e.g a webcam) directly to J16.
   Solution:
   Rather than connecting the webcam directly, connect a powered hub to J16 on the Icicle Kit, and then connect the webcam to the hub.
   
   To connect high power USB device, its is better to use an externally powered USB hub in between the USB device and the Icicle Kit.
   
2.  If a VBUS_ERROR is displayed and LEDs turns off, a board power cycle will be needed to get the USB port working.
   