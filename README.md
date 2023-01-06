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
