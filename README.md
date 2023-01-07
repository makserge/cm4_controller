# HASS.io install on CM4 with Waveshare CM4-IO-BASE-B and PVA030C05L-P01 3010 fan 5V

Reference to official installation guide:
https://www.home-assistant.io/installation/raspberrypi

0. Connect fan to IO board:
(Pin numeration from top to bottom)

1 - yellow
2 - blue
3 - red
4 - black

All steps are done on macOS.

1. Switch BOOT to ON on IO board
2. Connect USB type-c cable both to IO board and computer

3. Install usbboot 

3.1 Clone the usbboot repository

git clone --depth=1 https://github.com/raspberrypi/usbboot

3.2 Install libusb and pkg-config

brew install pkgconfig libusb

3.3 Build using make

cd usbboot
make

3.4 Run the binary

sudo ./rpiboot


RPIBOOT: build-date Jan  6 2023 version 20221215~105525 e4819de4
Waiting for BCM2835/6/7/2711...
Loading embedded: bootcode4.bin
Sending bootcode.bin
Successful read 4 bytes 
Waiting for BCM2835/6/7/2711...
Loading embedded: bootcode4.bin
Second stage boot server
Cannot open file config.txt
Cannot open file pieeprom.sig
Loading embedded: start4.elf
File read: start4.elf
Cannot open file fixup4.dat
Second stage boot server done

4. Download (https://www.balena.io/etcher) and start Balena Etcher
5. Select “Flash from URL” 

6. Insert 
https://github.com/home-assistant/operating-system/releases/download/9.4/haos_rpi4-64-9.4.img.xz

to "Use Image URL" field

7. Press "Ok" and wait until download will be complete
8. Click “Select target”
9. Select "Compute Module" and press "Select"
10. Press "Flash" and wait until flashing complete (about 10 minutes)
11. Close Balena Etcher
12. Disconnect USB type-c cable from computer
13. Switch BOOT to OFF on IO board
14. Attach an Ethernet cable for network.
15. Attach the power cable.
16. In the browser of your Computer, within a few minutes you will be able to reach your new Home Assistant on 
homeassistant.local:8123
17. Wait for about 5 minutes for preparing
18. Create administrator account
19. Choose country and language, press "Next", press "Next", press "Finish"

20. Enable USB

20.1 Shutdown Compute module
20.2 Switch BOOT to ON on IO board
20.3 Connect USB type-c cable both to IO board and computer
20.4 Start usbboot

sudo ./rpiboot

RPIBOOT: build-date Jan  6 2023 version 20221215~105525 e4819de4
Waiting for BCM2835/6/7/2711...
Loading embedded: bootcode4.bin
Sending bootcode.bin
Successful read 4 bytes 
Waiting for BCM2835/6/7/2711...
 Loading embedded: bootcode4.bin
Second stage boot server
Cannot open file config.txt
Cannot open file pieeprom.sig
Loading embedded: start4.elf
File read: start4.elf
Cannot open file fixup4.dat
Second stage boot server done

20.5 Find first partition on Compute module

diskutil list

/dev/disk0 (internal, physical):
...

/dev/disk1 (synthesized):
...

/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *31.3 GB    disk2
   1:         Microsoft Reserved                         33.6 MB    disk2s1
   2:           Linux Filesystem                         25.2 MB    disk2s2
   3:           Linux Filesystem                         268.4 MB   disk2s3
   4:           Linux Filesystem                         25.2 MB    disk2s4
   5:           Linux Filesystem                         268.4 MB   disk2s5
   6:           Linux Filesystem                         8.4 MB     disk2s6
   7:           Linux Filesystem                         100.7 MB   disk2s7
   8:           Linux Filesystem                         30.5 GB    disk2s8

20.6 Mount partition to some folder

mkdir boot

sudo mount -t msdos /dev/disk2s1 boot

20.7 Edit config.txt 

cd boot
nano config.txt

Add to the end

[cm4]
# Enable host mode on the 2711 built-in XHCI USB controller.
# This line should be removed if the legacy DWC2 controller is required
# (e.g. for USB device mode) or if USB support is not required.
otg_mode=1

20.8 Unmount boot

cd ..
sudo umount boot

20.9. Disconnect USB type-c cable from computer
20.10. Switch BOOT to OFF on IO board


