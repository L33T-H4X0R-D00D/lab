# Lab
Tools for building and maintaining travel lab and remote content endpoints.

## Prerequisites:
* Raspberry pi 8GB+
* SDcard of sufficient size to run RPi OS with desktop environment
* NVMe case or hat
* NVMe drive
* USB-C charger capable of running Raspberry Pi
* SDcard adapter for another computer
* Another computer on the same network that you can run your SSH application of choice

## Initial setup
Assembly and building the recovery partition.
1. Using the Raspberry Pi Imager (https://www.raspberrypi.com/software/), write the current default Raspberry Pi OS to SDcard. During customization in the Raspberry Pi Imager make certain to set the hostname to "recovery".   

2. Install the SDcard in Raspberry Pi.  

3. Install the NVMe case or hat.  

4. Boot the Raspberry Pi connected to keyboard, mouse, monitor, and network.  Once the desktop environment is up, use the Raspberry Pi Imager on the device to write the Raspberry Pi OS to the NVMe drive.  During customization in the Raspberry Pi Imager make certain to set the hostname to something other than "recovery". 

5. Change boot order through Raspi-config
```bash
sudo raspi-config
```
Navigate to 6 Advanced Options > A5 Boot Order. Select NVMe/USB Boot.

6. Reboot
```bash
sudo shutdown -r now
```
7. After your Raspberry Pi reboots confirm the hostname isn't "recovery".
```bash
hostname
```


# Container environment
8. Connect to your Raspberry Pi via SSH.

9. Install Docker. This is taken directly from the Docker website.
```bash
curl -sSL https://get.docker.com | sh
```
10. Create Docker user
```bash
sudo usermod -aG docker $USER
```
11. Enable PCIe Gen 3
```bash
echo "dtparam=pciex1_gen=3" | sudo tee -a /boot/firmware/config.txt
```
12. Reboot
```bash
sudo shutdown -r now
```
13. Once the Raspberry Pi reboots, log back in and check that the Docker group was created.  
```bash
groups
```
14. Install Portainer: port 9443.
```bash
docker run -d --name Portainer -p 8000:8000 -p 9443:9443 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data --restart always portainer/portainer-ce:latest
```



# Application containers
These applications can be installed in any combination. All their dependencies are capture within the container instantiation command. 
## Open WebUI/Olllama: 3000
Ollama studio allows you to install a variety of LLMs which can be accessed through the web interface on port 3000 via browser.
```bash
docker run -d --name OpenWebUI -p 3000:8080 -v /home/lab/ollama:/root/.ollama -v /home/lab/open-webui:/app/backend/data -e PUID=1000 -e PGID=1000 --restart=unless-stopped ghcr.io/open-webui/open-webui:ollama
```

## Jellyfin: 8096
Jellyfin is a very capable media player for movies, TV shows, and anime.  It can be accessed through the web interface on port 8096 via browser, via branded phone /TV app.
```bash
docker run -d --name Jellyfin -p 8096:8096/tcp -p 7359:7359/udp -p 1900:1900/udp  -v /home/lab/jellyfin/config:/config -v /home/lab/media/movies:/data/movies -v /home/lab/media/tv:/data/tv -v /home/lab/media/kidsmovies:/data/kidsmovies -v /home/lab/media/kidstv:/data/kidstv -v /home/lab/media/holidaymovies:/data/holidaymovies -v /home/lab/media/holidaytv:/data/holidaytv -v /home/lab/media/anime:/data/anime -v /home/lab/media/animetv:/data/animetv -v /home/lab/media/music:/data/music -e PUID=1000 -e PGID=1000 -e TZ=America/New_York --restart unless-stopped linuxserver/jellyfin
```

## Navidrome: 4533
Navidrome is a very light music manager that replicates the Spotify experience. It can be accessed through the web interface on port 4533 via browser, the Amperfy app on iPhone, or Subtracks on Android. 
```bash
docker run -d --name Navidrome -p 4533:4533 -v /home/lab/navidrome/config:/data -v /home/lab/media/music:/music:ro -e ND_ENABLEINSIGHTSCOLLECTOR=false -e PUID=1000 -e PGID=1000 --restart unless-stopped deluan/navidrome:latest
```

## Calibre Web Automated: 8083
Calibre Web is an ebook manager and distribution channel. It works with a variety of ebook readers. It can be accessed through the web interface on port 8083 via browser, or a variety of ebook readers. 
```bash
docker run -d --name Calibre -p 8083:8083  -v /home/lab/cwa/config:/config -v /home/lab/cwa/ingest:/cwa-book-ingest -v /home/lab/cwa/library:/calibre-library -v /home/lab/cwa/config/plugins:/config/.config/calibre/plugins -e PUID=1000 -e PGID=1000 -e TZ=America/New_York -e NETWORK_SHARE_MODE=false --restart unless-stopped crocodilestick/calibre-web-automated:latest
```




## PiHole DNSBL: 81/admin/
PiHole is a very light DNSBL tool that blocks ads, malware, tracking, and other unwanted connections based on what source lists you provide it.  This install is specifically for the DNSBL alone.  It can be accessed through the web interface on port 81/admin via browser. 
```bash
docker run -d --name PiHole -p 53:53/tcp -p 53:53/udp -p 81:80/tcp -p 444:443/tcp -p 67:67/udp -p 123:123/udp -v /home/lab/pihole:/etc/pihole -e TZ=America/New_York -e FTLCONF_webserver_api_password=defaultpassword -e FTLCONF_dns_listeningMode=ALL --restart unless-stopped pihole/pihole:latest
```


## PiHole DNSBL and NTP: 81/admin/
PiHole is a very light DNSBL tool that blocks ads, malware, tracking, and other unwanted connections based on what source lists you provide it.  This install is specifically for the DNSBL and the NTP service.  You'll need a real time clock hat to get the most out of this install.  It can be accessed through the web interface on port 81/admin via browser.
```bash
docker run -d --name PiHole-Time -p 53:53/tcp -p 53:53/udp -p 81:80/tcp -p 444:443/tcp -p 67:67/udp -p 123:123/udp -v /home/lab/pihole:/etc/pihole -e TZ=America/New_York -e FTLCONF_webserver_api_password=defaultpassword -e FTLCONF_dns_listeningMode=ALL -e SYS_TIME --restart unless-stopped pihole/pihole:latest
```

## PiHole DNSBL, NTP and DHCP server:  81/admin/
PiHole is a very light DNSBL tool that blocks ads, malware, tracking, and other unwanted connections based on what source lists you provide it.  This install is specifically for the DNSBL, the NTP, and DHCP services.  You'll need a real time clock hat to get the most out of this install.  Do not use this install if you already have a DHCP server on your network. It can be accessed through the web interface on port 81/admin via browser.
```bash
docker run -d --name PiHole-Time-DHCP -p 53:53/tcp -p 53:53/udp -p 81:80/tcp -p 444:443/tcp -p 67:67/udp -p 123:123/udp -v /home/lab/pihole:/etc/pihole -e TZ=America/New_York -e FTLCONF_webserver_api_password=defaultpassword -e FTLCONF_dns_listeningMode=ALL -e SYS_TIME -e NET_ADMIN --restart unless-stopped pihole/pihole:latest
```

## Immich: 2283
Immich is an open source photo manager that organizes your pictures into a timeline.  By default it applies one of several machine learning models on the images to build profiles on who's in them and where they are. This provides metadata useful when searching for specific photos.  This service is fairly heavy if you leave the machine learning enabled and upload a large volume of photos. Once the ML service finishes the CPU utilization drops to nothing. ~5,000 photos took approximately an hour to ingest before settling down.  It can be accessed through the web interface on port 2283 via browser.
```bash
wget -O docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
mv example.env .env
docker compose up -d
```
## Nextcloud
It works but it sucks. Hopefully it's better in a container.
```bash
$ docker run -d --name NextCloud -p 8080:80 -v /home/lab/nextcloud/data:/var/www/html -v /home/lab/nextcloud/apps:/var/www/html/custom_apps -v /home/lab/nextcloud/config:/var/www/html/config -v /home/lab/nextcloud/db:/var/lib/mysql -v /home/lab/theme:/var/www/html/themes/ -e skip_domain_validation=true -e nextcloud_upload_limit=16g -e nextcloud_memory_limit=512m --restart unless-stopped nextcloud
```
`PHP_MEMORY_LIMIT` (default `512M`)
PHP_UPLOAD_LIMIT (default 512M) This sets the upload limit (post_max_size and upload_max_filesize) for big files. Note that you may have to change other limits depending on your client, webserver or operating system. Check the Nextcloud documentation for more information.
APACHE_BODY_LIMIT (default 1073741824 [1GiB]) This restricts the total size of the HTTP request body sent from the client. It specifies the number of bytes that are allowed in a request body. A value of 0 means unlimited. Check the Nextcloud documentation for more information.


## Opencloud
New fork for filesharing.  Written in Go. Very fast.

| services: | opencloud-rolling: | container_name: opencloud | volumes: | - opencloud-data:/var/lib/opencloud | - opencloud-config:/etc/opencloud | image: opencloudeu/opencloud-rolling | ports: | - 127.0.0.1:9200:9200 | entrypoint: | - /bin/sh | command: ["-c", "opencloud init --insecure true || true; opencloud server"] | environment: | - IDM_CREATE_DEMO_USERS=false | - OC_URL=https://your-ip-or-hostname | volumes: | opencloud-data: | opencloud-config:

## CopyParty
Nextcloud that doesn't suck.
```bash
docker run -d --name CopyParty -p 3923:3923 -v /home/lab/copyparty/share:/w -v /home/lab/copyparty/cfgdir:/cfg -e PUID=1000 -e PGID=1000 copyparty/ac
```

## GPS Bridge
Stuff
```bash
docker run -d --name GPS-Bridge -p 20000:20000/udp --restart unless-stopped --device=/dev/ttyUSB0 debian:bookworm-slim bash -c "apt-get update && apt-get install -y gpsd gpsd-clients socat && gpsd -G -n /dev/ttyUSB0 && gpspipe -r | socat - UDP-DATAGRAM:192.168.1.255:20000,broadcast"
```
docker run -d --name GPS-Bridge-TCP -p 20000:20000 --restart unless-stopped --device=/dev/ttyUSB0 debian:bookworm-slim bash -c "apt-get update && apt-get install -y gpsd gpsd-clients socat && gpsd -G -n /dev/ttyUSB0 && socat TCP-LISTEN:20000,reuseaddr,fork EXEC:'gpspipe -r'"

## Containerized Firefox: 3006
Running Firefox in a container is occasionally useful.  During use the container monopolizes CPU time and the performance is quite poor.  This is mostly here for edge cases.  It can be accessed through the web interface on port 3006 via browser.
```bash
docker run -d --name Firefox -p 3005:3000 -p 3006:3001 --security-opt seccomp=unconfined `#optional` -e PUID=1000 -e PGID=1000 -e TZ=Etc/UTC -e FIREFOX_CLI=https://www.google.com/ `#optional` --shm-size="1gb" --restart unless-stopped lscr.io/linuxserver/firefox:latest
```

## Containerized Libre Office: 3011
Running LibreOffice in a container is occasionally useful.  During use the container monopolizes CPU time and the performance is quite poor.  This is mostly here for edge cases.  It can be accessed through the web interface on port 3011 via browser.
```bash
docker run -d --name LibreOffice -p 3010:3000 -p 3011:3001 --security-opt seccomp=unconfined `#optional` -e PUID=1000 -e PGID=1000 -e TZ=America/New_York --restart unless-stopped lscr.io/linuxserver/libreoffice:latest
```








# Post install
Change ownership
```bash
sudo chown -R $USER /home/lab/
```
# Configuration details
## Portainer:
* You have 5 minutes to login to Portainer after you start it. If you miss this window you can restart the container with 
```bash
docker restart Portainer
```
* Settings -> Authentication will allow you to change password complexity slider and session timeout. 

## Jellyfin
* If you decide to setup a library to expose your music to Jellyfin, uncheck the box for "Enable LUFS scan" in the library settings. This will prevent Jellyfin from attempting to create normalization mappings every time a user logs in.

## Calibre Web Automated:
* Default Admin Login: Username: admin Password: admin123. 
* In basic configuration\feature configuration, enable kobo sync.  
* In UI configuration/view configuration change the title to the book club name, books per page to 200, and default theme to dark. 
* CWA Admin functions/CWA settings/CWA Auto-Ingest Automerge set to Ignore.
## PiHole:
* After your initial configuration, remove the default password environment variable from the container so you can set a unique password.

Nextcloud needs this frequently.
sudo -u www-data /usr/bin/php /var/www/nextcloud/occ files:scan-app-data
```console
docker exec --user www-data CONTAINER_ID php occ
```

# Workflows
## Calibre Web Automated
* Import books into Calibre desktop.  Update metadata, add tags, confirm it's been parsed correctly.  Highlight all new books, click save to disk.  Drop the files created there into CWA ingest directory. 
* To segregate the books of two different user groups, create a tag for each group.  Apply the tag to the books for that group. In user preferences, add the tag for that users group to allowed tags, and any other groups tags to denied tags.  By default CWA operates on most permission so one allow tag and ten deny tags will still let the user see that book.  


# Tools.

### Change ownership
If you get an error attempting to copy data into any of the media or library directories using the same account used to create the cointaner, run this command.
```bash
sudo chown -R $USER /home/lab/
``` 

### Check what containers are running from the prompt.
```bash
docker ps
```

### Update the EEPROM:
```bash
sudo rpi-eeprom-update -a
```

### Change boot order through Raspi-config
```bash
sudo raspi-config
```
Navigate to 6 Advanced Options > A5 Boot Order. Select NVMe/USB Boot.

Change the boot order through the command line interactively
```bash
sudo rpi-eeprom-config --edit
```
Look for the line that says `BOOT_ORDER=0x...`. Change the hex value to your desired sequence.
Code	Device	Description
**1**	SD Card	Standard MicroSD slot
**4**	USB-MSD	USB Thumb drives or External SSDs
**6**	NVMeM.2 SSDs (Raspberry Pi 5 only)
**2**	Network	PXE Network boot
**f**	Restart	Tells the Pi to loop back to the first device if all fail

### Boot from NVMe, USB, SDcard non-interactively
```bash
echo -e "[all]\nBOOT_ORDER=0xf641" | sudo rpi-eeprom-config --apply /dev/stdin
```
### Boot from SDcard non-interactively
```bash
echo -e "[all]\nBOOT_ORDER=0xf1" | sudo rpi-eeprom-config --apply /dev/stdin
```


### Enable PCIe Gen 3 interactively
```bash
sudo nano /boot/firmware/config.txt
```
Add this line at the bottom: 
```bash
dtparam=pciex1_gen=3
```
Save and exit (Ctrl+O, Enter, Ctrl+X).

### Enable PCIe Gen 3 non-interactively
```bash
sudo echo 'dtparam=pciex1_gen=3' >> /boot/firmware/config.txt
```
### Manually partitioning, adding filesystem, and mount point through the command prompt.
List the drives that can be seen by the OS.
```bash
lsblk
```
Create partition assuming disk is "sda".
```bash
sudo fdisk /dev/sda
```
Create new partition.
```
n
```
Remove signature (may not be an issue)
```
y
```
Write the partition.
```
w
```
Confirm the partition has been created.  Would be sda1 in this example.
```
lsblk
```
Create ext4 filesystem on new partition.
```
sudo mkfs.ext4 /dev/sda1
```
Create mount point for the new drive.
```
sudo mkdir /mnt/deedrive
```
Get the UUID for the partition. 
``` 
sudo blkid /dev/sda1
```
Edit fstab to mount the drive at boot.
```
sudo nano /etc/fstab
```
Add this line to fstab. 
```
UUID=(whatever your drive UUID was) /mnt/deedrive ext4 defaults 0 0 
```
Mount the drives listed in fstab.
```
sudo mount -a
```
Confirm the correct mount point is associated with the drive. 
```
lsblk
```

### Filesystem
This document builds a standardized filesystem located in   `/home/lab`.  Occasionally it may be necessary to `chown $USER` these directories if you get an error message while attempting to copy data to them. 

```bash
/home/lab/copyparty
/home/lab/cwa/config
/home/lab/cwa/config/plugins
/home/lab/cwa/ingest
/home/lab/cwa/library
/home/lab/immich/modelcache
/home/lab/immich/postgres
/home/lab/immich/library
/home/lab/jellyfin/config
/home/lab/media/anime
/home/lab/media/animetv
/home/lab/media/holidaymovies
/home/lab/media/holidaytv
/home/lab/media/images
/home/lab/media/kidsmovies
/home/lab/media/kidstv
/home/lab/media/movies
/home/lab/media/music
/home/lab/media/tv
/home/lab/navidrome/config
/home/lab/ollama
/home/lab/open-webui
/home/lab/pihole
/home/lab/
/home/lab/
/home/lab/
```

Ports
```
0081 Pihole
2283 Immich
3000 Open WebUI
3005 Firefox
3006 Firefox
3010 LibreOffice
3011 LibreOffice
4533 Navidrome
8080 NextCloud
8083 Calibre Web Automated
8096 Jellyfin
9443 Portainer
```





Mangement
X Portainer - Front end for managing docker - https://www.portainer.io
X PiHole - dnsblacklist - https://github.com/pi-hole/docker-pi-hole

Syncthing - Sync directories between devices - https://syncthing.net
Watchtower - Automatic Docker container updater. Developer dropped, find a fork. - https://github.com/containrrr/watchtower
mailrise - Accepts notification emails and forwards them to other services. - github.com/YoRyan/mailrise
netbox - track systems and configurations on your network. - netbox.dev
nginx proxy manager - manage reverse proxy to containers and maintains SSL certs - nginxproxymanager.com
phpipam - manages IPs and networks, compare to netbox - phpipam.net
duplicati - backup containers and container data
Beszel - System stats pane of glass - https://beszel.dev/
dozzle - displays realtime logs in one pane of glass - 
pulse - docker monitoring - pulserelay.pro
komodo - alternative to portainer
uptime kuma - tracks services and alerts if they're unresponsive - kuma.pet
Searxng - private search engine



Games
ROMM - play retro games in browser - https://romm.app/
Veloren - Minecraft like game - https://veloren.net/

Information
X Open WebUI- Web front end and AI host = https://docs.openwebui.com
Kiwix - Wikipedia backup - https://kiwix.org/en/
Hoarder- Now called Karakeep - bookmark browser - https://karakeep.app
Homarr - landing page - https://homarr.dev
Homepage - Landing page - https://gethomepage.dev/
Dashdot - CPU and RAM widgets for homarr - https://getdashdot.com/docs/configuration/cpu
GetMy - Shared lists by everyone - https://getmydocs.opensourceisawesome.com/


Communication
Copyparty - Nextcloud that doesn't suck -  https://github.com/9001/copyparty
Voce chat - offline discord - https://voce.chat/
docmost - google docs replacement - https://github.com/docmost/docmost
Bookstack - Note taking and lists.
Firefox container - Browser in a container. 

Media
X Immich - for hosting photos and video - https://immich.app
X Jellyfin - video stream - https://jellyfin.org  Plugins: Jellyscrub, skipintro
X Navidrome - music stream - https://www.navidrome.org
Amperfy - iPhone client for Navidrome - https://github.com/BLeeEZ/amperfy
Subtracks - Android client for Navidrome - https://f-droid.org/packages/com.subtracks/
X Calibre Web Automated- ebooks - https://github.com/janeczku/calibre-web
Audiobookshelf - audiobooks/podcasts - https://www.audiobookshelf.org
Komga - comics/magazines - https://komga.org


https://github.com/scaronni/spotify-ripper

netbird/tailscale for VPN.

keep the client apks in a file share.


Option 1: Waydroid (Android inside Raspberry Pi OS)
This is the modern, "best of both worlds" approach. It allows you to run Android apps in windows alongside your regular Linux desktop.

Requirements:
Raspberry Pi 4 or 5 (Pi 5 highly recommended for performance).

64-bit Raspberry Pi OS (Bookworm).

Wayland desktop (enabled by default on Pi 5).

Installation Steps:
Switch to 4k Kernel Pages (Pi 5 only): Waydroid doesn't support the Pi 5's default 16k page size yet.

Open terminal and run: sudo raspi-config

Navigate to Advanced Options > Kernel Page Size > 4K.

Finish and Reboot.

Enable Pressure Stall Information (PSI):

sudo nano /boot/firmware/cmdline.txt

Add psi=1 to the end of the line, save (Ctrl+O), and exit (Ctrl+X).

Install via Pi-Apps (Easiest Method):

Install Pi-Apps if you don't have it: wget -qO- https://raw.githubusercontent.com/Botspot/pi-apps/master/install | bash

Open Pi-Apps, go to Tools > Emulation > Waydroid, and click Install.

Follow the prompts to download the Android image (select "GAPPS" if you want Google Play).

### The "Raspberry Pi 5" Workaround (Recommended)

If you need 5V/5A for a single-board computer, the most reliable method today is not a specific power bank, but a **PD-to-5V/5A Converter**.

-   **Pichondria / GeekPi / 52Pi PD Expansion Boards:**
    
    -   These boards take a standard **12V or 15V** output from _any_ high-quality PD power bank (like an Anker 737) and buck it down to a stable **5V/5A** or even **5V/8A**.
        
    -   **Setup:** `Power Bank (15V/3A)` $\rightarrow$ `USB-C Cable` $\rightarrow$ `Converter Board` $\rightarrow$ `Device`.
        

### 3. Comparison of High-Current Options

**Model**

**Advertised Max at ~5V**

**Protocol**

**Best Use Case**

**Anker 737 (PowerCore 24K)**

5V / 3A

USB-PD

General high-power devices

**Baseus Blade 100W**

5V / 4.5A (USB-A)

SCP / Proprietary

High-current 5V legacy devices

**Shargeek Storm 2**

Adjustable DC

Manual DC

DIY / Variable Voltage needs

### Important Note on Cables

To actually achieve 5A, you **must** use a cable rated for 5A (often labeled as **100W** or **240W** cables). Standard USB-C cables are frequently limited to 3A by an internal chip (E-Marker). If the cable is only 3A, the power bank will never output 5A regardless of its specs.
