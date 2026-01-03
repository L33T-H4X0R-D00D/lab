# Lab
Documents for my travel lab



Check what containers are running
docker ps


1. Install Docker
curl -sSL https://get.docker.com | sh

2. Create Docker user
sudo usermod -aG docker $USER

3. reboot
sudo shutdown -r now

4. After login, check for Docker group.
groups

5. Pull down Portainer container.
docker pull portainer/portainer-ce:latest

6. Create Portainer disk volume.
docker volume create portainer_data

7. Install Portainer container on port 9443.
docker run -d -p 8000:8000 -p 9443:9443 --name portainer -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data --restart unless-stopped portainer/portainer-ce:latest

8. Install Open WebUI with Ollama running on port 3000.
docker run -d -p 3000:8080 -v ollama:/root/.ollama -v open-webui:/app/backend/data --name open-webui --restart=unless-stopped ghcr.io/open-webui/open-webui:ollama

9. Install Jellyfin listening on port 8096.
docker run -d -p 8096:8096/tcp -p 7359:7359/udp -p 1900:1900/udp --name jellyfin -v /home/pi/jellyfin/config:/config -v /home/pi/jellyfin/movies:/data/movies -v /home/pi/jellyfin/tv:/data/tv -e PUID=1000 -e PGID=1000 -e TZ=America/New_York --restart unless-stopped linuxserver/jellyfin

10. Install Calibre Web Automated on port 8083. Default Admin Login: Username: admin Password: admin123. In basic configuration\feature configuration, enable anonymous browsing and kobo sync.  In UI configuration/view configuration change the title to the book club name and books per page to 200. CWA Admin functions/CWA settings/CWA Auto-Ingest Automerge set to Ignore.
docker run -d -p 8083:8083 --name calibre-web-automated -v /home/pi/cwa/config:/config -v /home/pi/cwa/ingest:/cwa-book-ingest -v /path/to/your/calibre/library:/calibre-library -v /home/pi/cwa/config/plugins:/config/.config/calibre/plugins -e PUID=1000 -e PGID=1000 -e TZ=America/New_York -e NETWORK_SHARE_MODE=false --restart unless-stopped crocodilestick/calibre-web-automated:latest

