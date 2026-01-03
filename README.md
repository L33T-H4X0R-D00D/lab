# Lab
Tools for building and maintaining the smallest travel lab possible.

  Filesystem
This document builds a standardized filesystem located in   `/home/lab`.  Occasionally it may be necessary to `chown $USER` these directories if you get an error message while attempting to copy data to them. 

```bash
/home/lab/ollama
/home/lab/open-webui
/home/lab/media/movies
/home/lab/media/tv
/home/lab/media/kidsmovies
/home/lab/media/kidstv
/home/lab/media/holidaymovies
/home/lab/media/holidaytv
/home/lab/media/anime
/home/lab/media/animetv
/home/lab/media/music
/home/lab/jellyfin/config
/home/lab/navidrome/config
/home/lab/cwa/config
/home/lab/cwa/ingest
/home/lab/cwa/library
/home/lab/cwa/config/plugins
/home/lab/pihole
/home/lab/immache
/home/lab/copyparty
/home/lab/
/home/lab/
/home/lab/
```

Check what containers are running from the prompt.
```bash
docker ps
```

This documentation assumes you have a fresh Raspberry Pi install that you've completed the initial setup.  It is highly recommended that you use storage other than SDcard for this build.  NVMe and SATA adapters are incredibly cheap and significantly faster, as well as more durable than SDcard.  The Raspberry Pi needs to be on the network and you'll need to connect to it via SSH.  

Install Docker. This is taken directly from the Docker website.
```bash
curl -sSL https://get.docker.com | sh
```

Create Docker user
```bash
sudo usermod -aG docker $USER
```

Reboot
```bash
sudo shutdown -r now
```

Once the Raspberry Pi reboots, log back in and check that the Docker group was created.  
```bash
groups
```

Install Portainer container.
```bash
docker pull portainer/portainer-ce:latest
```

Create Portainer disk volume.
```bash
docker volume create portainer_data
```

Install Portainer container on port 9443.
```bash
docker run -d --name Portainer -p 8000:8000 -p 9443:9443 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data --restart always portainer/portainer-ce:latest
```

Install Open WebUI with Ollama running on port 3000.
```bash
docker run -d --name Open-Webui -p 3000:8080 -v /home/lab/ollama:/root/.ollama -v /home/lab/open-webui:/app/backend/data -e PUID=1000 -e PGID=1000 --restart=unless-stopped ghcr.io/open-webui/open-webui:ollama
```

Install Jellyfin listening on port 8096.
```bash
docker run -d --name Jellyfin -p 8096:8096/tcp -p 7359:7359/udp -p 1900:1900/udp  -v /home/lab/jellyfin/config:/config -v /home/lab/media/movies:/data/movies -v /home/lab/media/tv:/data/tv -v /home/lab/media/kidsmovies:/data/kidsmovies -v /home/lab/media/kidstv:/data/kidstv -v /home/lab/media/holidaymovies:/data/holidaymovies -v /home/lab/media/holidaytv:/data/holidaytv -v /home/lab/media/anime:/data/anime -v /home/lab/media/animetv:/data/animetv -v /home/lab/media/music:/data/music -e PUID=1000 -e PGID=1000 -e TZ=America/New_York --restart unless-stopped linuxserver/jellyfin
```

install Navidrome on port 4533.
```bash
docker run -d --name Navidrome -p 4533:4533 -v /home/lab/navidrome/config:/data -v /home/lab/media/music:/music:ro -e PUID=1000 -e PGID=1000 --restart unless-stopped deluan/navidrome:latest
```

Install Calibre Web Automated on port 8083. Default Admin Login: Username: admin Password: admin123. In basic configuration\feature configuration, enable anonymous browsing and kobo sync.  In UI configuration/view configuration change the title to the book club name and books per page to 200. CWA Admin functions/CWA settings/CWA Auto-Ingest Automerge set to Ignore.
```bash
docker run -d --name Calibre -p 8083:8083  -v /home/lab/cwa/config:/config -v /home/lab/cwa/ingest:/cwa-book-ingest -v /home/lab/cwa/library:/calibre-library -v /home/lab/cwa/config/plugins:/config/.config/calibre/plugins -e PUID=1000 -e PGID=1000 -e TZ=America/New_York -e NETWORK_SHARE_MODE=false --restart unless-stopped crocodilestick/calibre-web-automated:latest
```

Install PiHole for DNSBL, 443
```bash
docker run -d --name PiHole -p 53:53/tcp -p 53:53/udp -p 80:80/tcp -p 443:443/tcp -p 67:67/udp -p 123:123/udp -v /home/lab/pihole:/etc/pihole -e TZ=America/New_York -e FTLCONF_webserver_api_password: 'defaultpassword' -e FTLCONF_dns_listeningMode: 'ALL' restart: unless-stopped pihole/pihole:latest
```

Install PiHole for DNSBL and NTP, 443
```bash
docker run -d --name PiHole -p 53:53/tcp -p 53:53/udp -p 80:80/tcp -p 443:443/tcp -p 67:67/udp -p 123:123/udp -v /home/lab/pihole:/etc/pihole -e TZ=America/New_York -e FTLCONF_webserver_api_password: 'defaultpassword' -e FTLCONF_dns_listeningMode: 'ALL' -e SYS_TIME restart: unless-stopped pihole/pihole:latest
```
Install PiHole for DNSBL, NTP and DHCP server, 443
```bash
docker run -d --name PiHole -p 53:53/tcp -p 53:53/udp -p 80:80/tcp -p 443:443/tcp -p 67:67/udp -p 123:123/udp -v /home/lab/pihole:/etc/pihole -e TZ=America/New_York -e FTLCONF_webserver_api_password: 'defaultpassword' -e FTLCONF_dns_listeningMode: 'ALL' -e SYS_TIME -e NET_ADMIN restart: unless-stopped pihole/pihole:latest



 NET_ADMIN
      # Required if you are using Pi-hole as your NTP client to be able to set the host's system time
      - SYS_TIME
      # Optional, if Pi-hole should get some more processing time
      - SYS_NICE


 













Mangement
Syncthing - Sync directories between devices - https://syncthing.net
Portainer - Front end for managing docker - https://www.portainer.io
Watchtower - Automatic Docker container updater. Developer dropped, find a fork. - https://github.com/containrrr/watchtower
mailrise - Accepts notification emails and forwards them to other services. - github.com/YoRyan/mailrise
netbox - track systems and configurations on your network. - netbox.dev
nginx proxy manager - manage reverse proxy to containers and maintains SSL certs - nginxproxymanager.com
phpipam - manages IPs and networks, compare to netbox - phpipam.net
duplicati - backup containers and container data
pihole - dnsblacklist - https://github.com/pi-hole/pi-hole
dozzle - displays realtime logs in one pane of glass - 
pulse - docker monitoring - pulserelay.pro
komodo - alternative to portainer
uptime kuma - tracks services and alerts if they're unresponsive - kuma.pet


Games
ROMM - play retro games in browser - https://romm.app/

Information
Kiwix - Wikipedia backup - https://kiwix.org/en/
Hoarder- Now called Karakeep - bookmark browser - https://karakeep.app
Open WebUI- Web front end and AI host = https://docs.openwebui.com
Homarr - landing page - https://homarr.dev
Dashdot - CPU and RAM widgets for homarr - https://getdashdot.com/docs/configuration/cpu

Communication
Voce chat - offline discord - https://voce.chat/
docmost - google docs replacement - https://github.com/docmost/docmost
Copyparty - Nextcloud that doesn't suck -  https://github.com/9001/copyparty

Media
Immich - for hosting photos and video - https://immich.app
Jellyfin - video stream - https://jellyfin.org
Navidrome - music stream - https://www.navidrome.org
Amperfy - iPhone client for Navidrome - https://github.com/BLeeEZ/amperfy
Subtracks - Android client for Navidrome - https://f-droid.org/packages/com.subtracks/
Audiobookshelf - audiobooks/podcasts - https://www.audiobookshelf.org
Calibre web automated- ebooks - https://github.com/janeczku/calibre-web
Komga - comics/magazines - https://komga.org


netbird/tailscale for VPN.

https://www.youtube.com/watch?v=3DL6hfcGjV4

keep the client apks in a file share.
