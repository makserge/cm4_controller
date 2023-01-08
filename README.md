# HASS.io install on CM4 with Waveshare CM4-IO-BASE-B, 1TB NVME 2242 SSD and PVA030C05L-P01 3010 fan 5V

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

4. Download (https://www.raspberrypi.com/software/) and start Raspberry Pi Imager
5. Select “Raspberry PI OS (32 bit)” 
6. Choose "RPI-MSD-0001" storage 
7. Press "Write" and wait until flashing complete (about 15 minutes)
8. Close Raspberry Pi Imager
9. Enable USB

9.1 Shutdown Compute module
9.2 Start usbboot

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

9.3 Open "boot" disk and open config.txt

Add to the end

[cm4]
# Enable host mode on the 2711 built-in XHCI USB controller.
# This line should be removed if the legacy DWC2 controller is required
# (e.g. for USB device mode) or if USB support is not required.
otg_mode=1

9.4 Unmout "boot" disk

10. Disconnect USB type-c cable from computer
11. Switch BOOT to OFF on IO board
12. Attach an Ethernet cable for network and HDMI for monitor.
13. Attach the power cable.
14. Wait until Raspberry PI OS boot complete
15. Make initial OS setup (timezone, user etc.)
16. Flash HASS 

16.1 From menu open "Accessories", then "Image"
16.2 Select OS: "Other specific-purpose OS”, then "Home assistants and home automation", then "Home assistant", then "Home assistant OS 9.4 (RPI 4/400)"  
16.3 Choose NVME storage 
16.4. Press "Write" and wait until flashing complete (about 3 minutes)

17. Enable USB and external Wi-Fi antenna
17.1 Mount partition to some folder

mkdir boot

sudo mount -t msdos /dev/nvme0n1p1 boot

17.2 Edit config.txt 

cd boot
nano config.txt

Add to the end

[cm4]
# Enable host mode on the 2711 built-in XHCI USB controller.
# This line should be removed if the legacy DWC2 controller is required
# (e.g. for USB device mode) or if USB support is not required.
otg_mode=1
# Switch to external antenna.
dtparam=ant2

18. Switch to boot from NVME drive

Reference: https://github.com/raspberrypi/usbboot

18.1 Update boot order

cd usbboot
nano recovery/boot.conf

replace 
BOOT_ORDER=0xf25641

to 
BOOT_ORDER=0xf25416

18.2 Update the pieeprom.bin file with the new settings

cd recovery
./update-pieeprom.sh

18.3 Connect your CM4 in USB boot mode and run

../rpiboot -d .

After a few seconds it should successfully update the EEPROM and the new boot order should be ready. 
The activity LED should start blinking rapidly once complete.

18.4 Shutdown Compute module
18.5 Switch BOOT to OFF on IO board
18.6 Connect USB type-c cable both to IO board and computer

19. Enable SSH on host

19.1 Create public key

ssh-keygen -t ed25519 -C "your_email@example.com"
cp hass.pub authorized_keys

19.2 Copy an authorized_keys file to the root of the USB flash drive with label CONFIG and FAT, ext4, or NTFS filesystem. 
19.3 Insert flash drive and keyboard to compute module system and switch power on
19.4 In CM4 console run

host shutdown 

19.5 Remove flash drive and keyboard and switch power on and wait until system boot up
19.6 Add ssh key

chmod 600 name_of_privatekey
ssh-add name_of_privatekey

19.7 Check SSH key

ssh-add -l

and make sure that key is here

19.8 Login to SSH 

ssh root@homeassistant.local -p 22222


20. Setup HA 

20.1 In the browser of your Computer, within a few minutes you will be able to reach your new Home Assistant on 
homeassistant.local:8123
20.2. Wait for about 5 minutes for preparing
20.3 Create administrator account
20.4. Choose country and language, press "Next", press "Next", press "Finish"


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
