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
sudo echo 'dtparam=pciex1_gen=3' >> /boot/firmware/config.txt
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

* Install Open WebUI with Ollama: port 3000.
```bash
docker run -d --name OpenWebUI -p 3000:8080 -v /home/lab/ollama:/root/.ollama -v /home/lab/open-webui:/app/backend/data -e PUID=1000 -e PGID=1000 --restart=unless-stopped ghcr.io/open-webui/open-webui:ollama
```

* Install Jellyfin: port 8096.
```bash
docker run -d --name Jellyfin -p 8096:8096/tcp -p 7359:7359/udp -p 1900:1900/udp  -v /home/lab/jellyfin/config:/config -v /home/lab/media/movies:/data/movies -v /home/lab/media/tv:/data/tv -v /home/lab/media/kidsmovies:/data/kidsmovies -v /home/lab/media/kidstv:/data/kidstv -v /home/lab/media/holidaymovies:/data/holidaymovies -v /home/lab/media/holidaytv:/data/holidaytv -v /home/lab/media/anime:/data/anime -v /home/lab/media/animetv:/data/animetv -v /home/lab/media/music:/data/music -e PUID=1000 -e PGID=1000 -e TZ=America/New_York --restart unless-stopped linuxserver/jellyfin
```

* Install Navidrome: port 4533.
```bash
docker run -d --name Navidrome -p 4533:4533 -v /home/lab/navidrome/config:/data -v /home/lab/media/music:/music:ro -e ND_ENABLEINSIGHTSCOLLECTOR=false -e PUID=1000 -e PGID=1000 --restart unless-stopped deluan/navidrome:latest
```

* Install Calibre Web Automated: port 8083. 
```bash
docker run -d --name Calibre -p 8083:8083  -v /home/lab/cwa/config:/config -v /home/lab/cwa/ingest:/cwa-book-ingest -v /home/lab/cwa/library:/calibre-library -v /home/lab/cwa/config/plugins:/config/.config/calibre/plugins -e PUID=1000 -e PGID=1000 -e TZ=America/New_York -e NETWORK_SHARE_MODE=false --restart unless-stopped crocodilestick/calibre-web-automated:latest
```






* Install PiHole DNSBL: 81/admin/
```bash
docker run -d --name PiHole -p 53:53/tcp -p 53:53/udp -p 81:80/tcp -p 444:443/tcp -p 67:67/udp -p 123:123/udp -v /home/lab/pihole:/etc/pihole -e TZ=America/New_York -e FTLCONF_webserver_api_password=defaultpassword -e FTLCONF_dns_listeningMode=ALL --restart unless-stopped pihole/pihole:latest
```


* Install PiHole DNSBL and NTP: 81/admin/
```bash
docker run -d --name PiHole -p 53:53/tcp -p 53:53/udp -p 81:80/tcp -p 444:443/tcp -p 67:67/udp -p 123:123/udp -v /home/lab/pihole:/etc/pihole -e TZ=America/New_York -e FTLCONF_webserver_api_password=defaultpassword -e FTLCONF_dns_listeningMode=ALL -e SYS_TIME --restart unless-stopped pihole/pihole:latest
```

* Install PiHole DNSBL, NTP and DHCP server:  81/admin/
```bash
docker run -d --name PiHole -p 53:53/tcp -p 53:53/udp -p 81:80/tcp -p 444:443/tcp -p 67:67/udp -p 123:123/udp -v /home/lab/pihole:/etc/pihole -e TZ=America/New_York -e FTLCONF_webserver_api_password=defaultpassword -e FTLCONF_dns_listeningMode=ALL -e SYS_TIME -e NET_ADMIN --restart unless-stopped pihole/pihole:latest
```

* Install Immich: 2283
```bash
wget -O docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
mv example.env .env
docker compose up -d
```
* Install containerized Firefox: 3005
```bash
docker run -d --name Firefox -p 3005:3000 -p 3006:3001 --security-opt seccomp=unconfined `#optional` -e PUID=1000 -e PGID=1000 -e TZ=Etc/UTC -e FIREFOX_CLI=https://www.google.com/ `#optional` --shm-size="1gb" --restart unless-stopped lscr.io/linuxserver/firefox:latest
```

* Install containerized Libre Office: 3010
```bash
docker run -d --name LibreOffice -p 3010:3000 -p 3011:3001 --security-opt seccomp=unconfined `#optional` -e PUID=1000 -e PGID=1000 -e TZ=America/New_York --restart unless-stopped lscr.io/linuxserver/libreoffice:latest
```








# Post install
Change ownership
```bash
sudo chown -R $USER /home/lab/
```
# Additional configuration details
## Portainer:
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

# Workflows
## Calibre Web Automated
* Import books into Calibre desktop.  Update metadata, add tags, confirm it's been parsed correctly.  Highlight all new books, click save to disk.  Drop the files created there into CWA ingest directory. 
* To segregate the books of two different user groups, create a tag for each group.  Apply the tag to the books for that group. In user preferences, add the tag for that users group to allowed tags, and any other groups tags to denied tags.  By default CWA operates on most permission so one allow tag and ten deny tags will still let the user see that book.  


# Tools.

### Change ownership
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


  Filesystem
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
8083 Calibre Web Automated
8096 Jellyfin
9443 Portainer






Mangement
Syncthing - Sync directories between devices - https://syncthing.net
Portainer - Front end for managing docker - https://www.portainer.io
PiHole - dnsblacklist - https://github.com/pi-hole/docker-pi-hole
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


Games
ROMM - play retro games in browser - https://romm.app/
Veloren - Minecraft like game - https://veloren.net/

Information
Kiwix - Wikipedia backup - https://kiwix.org/en/
Hoarder- Now called Karakeep - bookmark browser - https://karakeep.app
Open WebUI- Web front end and AI host = https://docs.openwebui.com
Homarr - landing page - https://homarr.dev
Homepage - Landing page - https://gethomepage.dev/
Dashdot - CPU and RAM widgets for homarr - https://getdashdot.com/docs/configuration/cpu
GetMy - Shared lists by everyone - https://getmydocs.opensourceisawesome.com/


Communication
Voce chat - offline discord - https://voce.chat/
docmost - google docs replacement - https://github.com/docmost/docmost
Copyparty - Nextcloud that doesn't suck -  https://github.com/9001/copyparty
Bookstack - Note taking and lists.
Firefox container - Browser in a container. 

Media
Immich - for hosting photos and video - https://immich.app
Jellyfin - video stream - https://jellyfin.org  Plugins: Jellyscrub, skipintro
Navidrome - music stream - https://www.navidrome.org
Amperfy - iPhone client for Navidrome - https://github.com/BLeeEZ/amperfy
Subtracks - Android client for Navidrome - https://f-droid.org/packages/com.subtracks/
Audiobookshelf - audiobooks/podcasts - https://www.audiobookshelf.org
Calibre web automated- ebooks - https://github.com/janeczku/calibre-web
Komga - comics/magazines - https://komga.org


netbird/tailscale for VPN.

https://www.youtube.com/watch?v=3DL6hfcGjV4

keep the client apks in a file share.


