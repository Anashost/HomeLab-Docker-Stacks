# HomeLab Docker Stacks

<details>
  <summary>Plex</summary>
  
```
version: "2.1"
services:
  plex:
    image: lscr.io/linuxserver/plex:latest
    container_name: plex
    network_mode: host
    environment:
      - PUID=0
      - PGID=0
      - VERSION=docker
      - PLEX_CLAIM= #optional
    volumes:
      - /srv/plex/config:/config
      - /srv/dev-disk-by-uuid-nas/plex/TV:/tv
      - /srv/dev-disk-by-uuid-nas/plex/Movies:/movies
      - /srv/dev-disk-by-uuid-nas/plex/Music:/music
    restart: unless-stopped
```
</details>

<details>
  <summary>Jellyfin</summary>
  
```
services:
  jellyfin:
    image: jellyfin/jellyfin
    container_name: jellyfin
    ## port ui: 8096
    network_mode: 'host'
    volumes:
      - /srv/jellyfin/config:/config
      - /srv/jellyfin/cache:/cache
      - type: bind
        source: /srv/dev-disk-by-uuid-nas/media
        target: /media
        read_only: true
    restart: 'unless-stopped'
```
</details>

<details>
  <summary>Open-webui</summary>
  
```
version: '3.8'
services:
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    environment:
      - OLLAMA_BASE_URL=http://10.0.0.100:11434 ## change me
    ports:
      - "3232:8080"
    volumes:
      - open-webui:/app/backend/data
    restart: 'unless-stopped'
volumes:
  open-webui:
```
</details>

<details>
  <summary>Pihole</summary>
  
```
version: '3.8'
services:
  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    ports:
      - "8085:80"        # HTTP for Pi-hole web interface
      - "443:443"        # HTTPS (optional, if SSL is configured)
      - "5354:53/tcp"    # Use port 5353 on the host for DNS (TCP)
      - "5354:53/udp"    # Use port 5353 on the host for DNS (UDP)
    volumes:
      - pihole-data:/etc/pihole
      - dnsmasq-data:/etc/dnsmasq.d
    environment:
      - WEBPASSWORD=0000
      - DNS1=8.8.8.8
      - DNS2=8.8.4.4
      - ServerIP=10.0.0.100 #change me
    restart: unless-stopped
volumes:
  pihole-data:
  dnsmasq-data:
```
</details>

<details>
  <summary>Uptime-kuma</summary>
  
```
version: '3.8'
services:
  uptime-kuma:
    image: louislam/uptime-kuma:latest
    container_name: uptime-kuma
    ports:
      - "3001:3001"    # UI
    volumes:
      - uptime-kuma-data:/app/data  # Persist Uptime Kuma data
    environment:
      - DB_TYPE=sqlite
    restart: unless-stopped
volumes:
  uptime-kuma-data:
```
</details>

<details>
  <summary>nginx proxy manager</summary>
  
```
services:
  app:
    image: 'docker.io/jc21/nginx-proxy-manager:latest'
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    restart: unless-stopped
```
</details>

<details>
  <summary>Nextcloud</summary>
  
```
version: '2'
volumes:
  nextcloud:
  db:
services:
  db:
    image: mariadb
    restart: always
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW --innodb-file-per-table=1 --skip-innodb-read-only-compressed

    volumes:
      - db:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=admin
      - MYSQL_PASSWORD=xxxxxxx ## change this
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
  app:
    image: nextcloud
    restart: always
    ports:
      - 8888:80
    links:
      - db
    volumes:
      - /srv/dev-disk-by-uuid-nas/nextcloud:/var/www/html ## storage
    environment:
      - MYSQL_PASSWORD=xxxxxxx  ## change this to match the mysql_password above
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_HOST=db
```
</details>
