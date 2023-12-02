# HASS.io install on CM4 with Waveshare CM4-IO-BASE-B, 1TB NVME 2242 SSD and PVA030C05L-P01 3010 fan 5V

Reference to official installation guide:
https://www.home-assistant.io/installation/raspberrypi

0.1 Connect fan to IO board:
(Pin numeration from top to bottom)

    1 - yellow
    2 - blue
    3 - red
    4 - black

0.2 Connect PCM5102A DAC to IO board (40 pin socket)

    IO BOARD       DAC
    
    1 - 5V         - VIN
    3 - GND        - SCK and GND
    12 - PCM_CLK   - BCK
    35 - PCM_FS    - LCK
    40 - PCM_DOUT  - DIN

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
    16.2 Select OS: "Other specific-purpose OS”, then "Home assistants and home automation", then "Home assistant", then "Home assistant OS 11.1 (RPI 4/400)"  
    16.3 Choose NVME storage 
    16.4. Press "Next", then "Yes", enter user password and wait until flashing complete (about 5 minutes)

18. Enable USB and external Wi-Fi antenna
    17.1 Mount partition to some folder
    
        cd
        mkdir boot
        sudo mount -t msdos /dev/nvme0n1p1 boot
    
    17.2 Edit config.txt 

        sudo su
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

    17.3 Shutdown Compute module

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

20. Enable fan, RTC and I2S

    20.1 Edit config
    
        vi /mnt/boot/config.txt
    
    add
    
        # Enable RTC
        dtparam=i2c_vc=on
        dtoverlay=i2c-rtc,pcf85063a,i2c_csi_dsi
        
        # Fan Controller
        dtoverlay=i2c-fan,emc2301,i2c_csi_dsi,midtemp=45000,maxtemp=50000
        
        #I2S DAC
        dtparam=i2s=on
        dtoverlay=hifiberry-dac
    
    Comment out
    
        #dtparam=audio=on
    
    20.2 Reboot 
    
        ha host reboot

21. Setup HA 

    21.1 In the browser of your Computer, within a few minutes you will be able to reach your new Home Assistant on 
    homeassistant.local:8123

    21.2. Wait for about 10 minutes for preparing

    21.3 Press "Create my smart home" and create admin account

    21.4. Choose country and language, press "Next", press "Next", press "Finish"

    21.5 Enable advanced mode

    Open http://homeassistant.local:8123/profile and toggle "Advanced mode" to On

22. MQTT Mosquitto broker

    22.1 Open http://homeassistant.local:8123/config/users
    and add new user for mosquitto
    
    22.2 Open http://homeassistant.local:8123/hassio/store and find "Mosquitto"

    22.3 Press "Install" and wait until it installed

    22.4 Enable watchdog

    22.5 Tap "Configuration", switch to YAML and paste config
    
        logins:
          - username: myusername-change-this
            password: mypassword-change-this
        require_certificate: false
        certfile: fullchain.pem
        keyfile: privkey.pem
        customize:
          active: false
          folder: mosquitto
        anonymous: false
    
    22.6 Click "Save", then tap "Info", then click "Start"

23. Install Samba Add-on

    23.1 Open http://homeassistant.local:8123/hassio/store and find "Samba"
  
    23.2 Click on "Samba share" then "Install"

    23.3 Tap "Configuration" tab and fill username / password then press "Save"

    23.4 Tap "Info" tab and click "Start"

24. Install HACS Add-on

    24.1 Open http://homeassistant.local:8123/hassio/store and find "Terminal & SSH", select it and press "Install", then "Start"

    24.2 Press "Open Web UI" and in terminal paste
    
        wget -O - https://get.hacs.xyz | bash -
    
    24.3 Restart HA
    
        ha core restart
    
    24.4 Open "Settings", then "Devices & Services", then "Add integration"

    24.5 Search for "HACS" and select it.

    24.6 Check all checkboxes and press "Submit"

    24.7 Open provided link in new browser tab and enter Device activation key taken from this popup

    24.8 Press "Continue" then "Authorize hacs"

    24.9 Press "Finish" in popup

    24.10 Open http://homeassistant.local:8123/hassio/addon/core_ssh/info , then Press "Open Web UI" and in terminal paste
    
        ha core restart

25. Install FTP

    25.1 Open "Settimgs", then "Add-ons", then "Add-on Store", choose "FTP" and click "Install"

    25.2 Press "Start"
    
    25.3 Tap "Configuration" tab and fill username / password then press "Save"

26. Install MPD

    26.1 Open https://community.home-assistant.io/t/mpd-music-player-daemon-addon/320446

    26.2 Press "Add repository to my HomeAssistant", then "Install on my HomeAssistant"

    26.3 Press "Start"





