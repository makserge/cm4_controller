HASS install on N100 ZX-01 Mini PC 1TB NVME 2242 SSD

Thanks for tips to @kvazis (https://github.com/kvazis/training/tree/master/lessons/articles/2023_intel_supervised)

1. Download Debian 12 ISO (debian-12.4.0-amd64-DVD-1.iso) from https://cdimage.debian.org/debian-cd/current/amd64/iso-dvd/
2. Flash this ISO using Balena Etcher to USB stick
3. Boot from USB stick and install Debian server to whole disk (select SSH server only in software selection step)
4. Remove USB stick and boot from SSD
5. Login via SSH with created user credentials (non root)

ssh user@ip

6. Change user to root

su

7. Check Debian repos 

nano /etc/apt/sources.list

and comment out

#deb cdrom:[Debian GNU/Linux 12.4.0 _Bookworm_ - Official amd64 DVD Binary-1 with firmware 20231210-17:57]/ bookworm main non-free-firmware

8. Update all

apt update && apt upgrade -y && apt autoremove -y

9. Install all related pakets

apt-get install -y jq wget curl udisks2 apparmor-utils libglib2.0-bin network-manager dbus systemd-journal-remote systemd-resolved bluez

10. Disable ipv6

nano /etc/sysctl.conf

add to the end

net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1

11. Apply changes

sysctl -p
reboot

12. Update DNS config

nano /etc/systemd/resolved.conf

add

DNS=8.8.8.8 1.1.1.1

13. Restart DNS

sudo ln -s /usr/bin/resolvectl /usr/bin/systemd-resolve

systemctl restart systemd-resolved

14. Install Docker

curl -fsSL get.docker.com | sh

15. Install OS Agent from repo

https://github.com/home-assistant/os-agent/releases

wget https://github.com/home-assistant/os-agent/releases/download/1.6.0/os-agent_1.6.0_linux_x86_64.deb 
dpkg -i os-agent_1.6.0_linux_x86_64.deb

16. Install Home Assisistant Supervised from repo 
https://github.com/home-assistant/supervised-installer/releases

wget https://github.com/home-assistant/supervised-installer/releases/download/1.5.0/homeassistant-supervised.deb
dpkg -i homeassistant-supervised.deb

17. Setup HA

17.1 In the browser of your Computer, within a few minutes you will be able to reach your new Home Assistant on homeassistant.local:8123

17.2. Wait for about 10 minutes for preparing

17.3 Press "Create my smart home" and create admin account

17.4. Choose country and language, press "Next", press "Next", press "Finish"

17.5 Enable advanced mode

Open http://homeassistant.local:8123/profile and toggle "Advanced mode" to On

18. MQTT Mosquitto broker

18.1 Open http://homeassistant.local:8123/config/users and add new user for mosquitto

18.2 Open http://homeassistant.local:8123/hassio/store and find "Mosquitto"

18.3 Press "Install" and wait until it installed

18.4 Enable watchdog

18.5 Tap "Configuration", switch to YAML and paste config

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

18.6 Click "Save", then tap "Info", then click "Start"

19. Install Samba Add-on

19.1 Open http://homeassistant.local:8123/hassio/store and find "Samba"

19.2 Click on "Samba share" then "Install"

19.3 Tap "Configuration" tab and fill username / password then press "Save"

19.4 Tap "Info" tab and click "Start"

20. Install HACS Add-on

20.1 Open http://homeassistant.local:8123/hassio/store and find "Terminal & SSH", select it and press "Install", then "Start"

20.2 Press "Open Web UI" and in terminal paste

wget -O - https://get.hacs.xyz | bash -

20.3 Restart HA

ha core restart

20.4 Open "Settings", then "Devices & Services", then "Add integration"

20.5 Search for "HACS" and select it.

20.6 Check all checkboxes and press "Submit"

20.7 Open provided link in new browser tab and enter Device activation key taken from this popup

20.8 Press "Continue" then "Authorize hacs"

20.9 Press "Finish" in popup

20.10 Open http://homeassistant.local:8123/hassio/addon/core_ssh/info , then Press "Open Web UI" and in terminal paste

ha core restart

21. Install FTP

21.1 Open "Settings", then "Add-ons", then "Add-on Store", choose "FTP" and click "Install"

21.2 Press "Install" and wait until it installed

21.3 Enable watchdog

21.4 Tap "Configuration", switch to YAML and paste config (with different username and password)

port: 3721
data_port: 20
banner: Welcome to the Hass.io FTP service.
pasv: true
pasv_min_port: 30000
pasv_max_port: 30010
pasv_address: ""
ssl: false
certfile: fullchain.pem
keyfile: privkey.pem
implicit_ssl: false
max_clients: 10
users:
  - username: hassio
    password: changeme
    allow_chmod: false
    allow_download: true
    allow_upload: true
    allow_dirlist: true
    addons: false
    backup: false
    config: false
    media: true
    share: true
    ssl: false

21.5 Press "Save" and then "Start"

22. Install MPD

22.1 Open https://community.home-assistant.io/t/mpd-music-player-daemon-addon/320446

22.2 Press "Add repository to my HomeAssistant", open link in HomeAssistant and add repository, then press "Close"

22.3 Press two times back in brrowser to return to https://community.home-assistant.io/t/mpd-music-player-daemon-addon/320446

22.4 Press "Install on my HomeAssistant", open link in HomeAssistant, then press "Install"

22.5 Press "Start"
