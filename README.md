<!-- anashost_support_badges_start -->
[![Revolut.Me][revolut_me_shield]][revolut_me]
[![PayPal.Me][paypal_me_shield]][paypal_me]
[![ko_fi][ko_fi_shield]][ko_fi_me]
[![buymecoffee][buy_me_coffee_shield]][buy_me_coffee_me]
<!-- anashost_support_badges_end -->
<!-- 
```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
-->

# HomeLab Docker Stacks

>This repository contains a collection of Docker Compose configurations for setting up and managing various services in a home lab environment.
>Each stack is tailored for ease of use, scalability, and maintainability, with detailed descriptions and customizable options.

```yaml
               ##         .
             ## ## ##        ==
          ## ## ## ## ##    ===
      /"""""""""""""""""\___/ ===
 ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
      \______ o           __/
        \    \         __/
         \____\_______/
  _    _                         _           _        _____             _             
 | |  | |                       | |         | |      |  __ \           | |            
 | |__| | ___  _ __ ___   ___   | |     __ _| |__    | |  | | ___   ___| | _____ _ __ 
 |  __  |/ _ \| '_ ` _ \ / _ \  | |    / _` | '_ \   | |  | |/ _ \ / __| |/ / _ \ '__|
 | |  | | (_) | | | | | |  __/  | |___| (_| | |_) |  | |__| | (_) | (__|   <  __/ |   
 |_|  |_|\___/|_| |_| |_|\___|  |______\__,_|_.__/   |_____/ \___/ \___|_|\_\___|_|   
```

## Available Stacks

<details>
  <summary><strong>Plex</strong> - Media Server</summary>
  Plex is a powerful media server solution that organizes your media and allows streaming to various devices.

  ```yaml
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
        - PLEX_CLAIM= # optional
      volumes:
        - /srv/plex/config:/config
        - /srv/dev-disk-by-uuid-nas/plex/TV:/tv
        - /srv/dev-disk-by-uuid-nas/plex/Movies:/movies
        - /srv/dev-disk-by-uuid-nas/plex/Music:/music
      restart: unless-stopped
  ```
</details>

<details>
  <summary><strong>Jellyfin</strong> - Open Source Media Server</summary>
  Jellyfin is an open-source alternative to Plex, offering media organization and streaming capabilities.

  ```yaml
  services:
    jellyfin:
      image: jellyfin/jellyfin
      container_name: jellyfin
      network_mode: 'host'
      volumes:
        - /srv/jellyfin/config:/config
        - /srv/jellyfin/cache:/cache
        - type: bind
          source: /srv/dev-disk-by-uuid-nas/media
          target: /media
          read_only: true
      restart: unless-stopped
  ```
</details>

<details>
  <summary><strong>Open WebUI</strong> - Interface for LLMs</summary>
  Open WebUI provides a convenient interface for managing and interacting with language models.

  ```yaml
  version: '3.8'
  services:
    open-webui:
      image: ghcr.io/open-webui/open-webui:main
      container_name: open-webui
      environment:
        - OLLAMA_BASE_URL=http://10.0.0.100:11434 # Change this to your server IP
      ports:
        - "3232:8080"
      volumes:
        - open-webui:/app/backend/data
      restart: unless-stopped
  volumes:
    open-webui:
  ```
</details>

<details>
  <summary><strong>Pi-hole</strong> - Network-wide Ad Blocking</summary>
  Pi-hole is a DNS sinkhole that protects your devices from unwanted ads and trackers.

  ```yaml
  version: '3.8'
  services:
    pihole:
      image: pihole/pihole:latest
      container_name: pihole
      ports:
        - "8085:80"        # Web interface
        - "443:443"        # HTTPS (optional)
        - "5354:53/tcp"    # DNS (TCP)
        - "5354:53/udp"    # DNS (UDP)
      volumes:
        - pihole-data:/etc/pihole
        - dnsmasq-data:/etc/dnsmasq.d
      environment:
        - WEBPASSWORD=0000
        - DNS1=8.8.8.8
        - DNS2=8.8.4.4
        - ServerIP=10.0.0.100 # Change this to your server IP
      restart: unless-stopped
  volumes:
    pihole-data:
    dnsmasq-data:
  ```
</details>

<details>
  <summary><strong>Uptime Kuma</strong> - Monitoring</summary>
  Uptime Kuma is a self-hosted monitoring solution for websites, APIs, and services.

  ```yaml
  version: '3.8'
  services:
    uptime-kuma:
      image: louislam/uptime-kuma:latest
      container_name: uptime-kuma
      ports:
        - "3001:3001" # Web interface
      volumes:
        - uptime-kuma-data:/app/data # Persist data
      environment:
        - DB_TYPE=sqlite
      restart: unless-stopped
  volumes:
    uptime-kuma-data:
  ```
</details>

<details>
  <summary><strong>Nginx Proxy Manager</strong> - Reverse Proxy</summary>
  Nginx Proxy Manager simplifies reverse proxy management with an intuitive web interface.

  ```yaml
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
  <summary><strong>Nextcloud</strong> - File Sharing and Collaboration</summary>
  Nextcloud is a self-hosted productivity platform for file sharing, collaboration, and more.

  ```yaml
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
        - MYSQL_PASSWORD=xxxxxxx # Change this
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
        - /srv/dev-disk-by-uuid-nas/nextcloud:/var/www/html # Storage
      environment:
        - MYSQL_PASSWORD=xxxxxxx # Match the password above
        - MYSQL_DATABASE=nextcloud
        - MYSQL_USER=nextcloud
        - MYSQL_HOST=db
  ```
</details>

<details>
  <summary><strong>AdGuard Home</strong> - DNS-based Ad Blocking</summary>
  AdGuard Home is a network-wide ad and tracker blocking DNS server.

  ```yaml
  version: '3.8'
  services:
    adguardhome:
      image: adguard/adguardhome:latest
      container_name: adguardhome
      restart: unless-stopped
      volumes:
        - adguard-workdir:/opt/adguardhome/work  # Docker volume for work directory
        - adguard-confdir:/opt/adguardhome/conf  # Docker volume for configuration directory
      ports:
        - 53:53/tcp
        - 53:53/udp
        - 784:784/udp
        - 853:853/tcp
        - 3000:3000/tcp
        - 80:80/tcp
        - 443:443/tcp
      networks:
        - adguard_net
  volumes:
    adguard-workdir:
    adguard-confdir:
  networks:
    adguard_net:
      driver: bridge
  ```
</details>

<details>
  <summary><strong>Code Server</strong> - Remote Development</summary>
  Code Server allows you to run VS Code in the browser for remote development.

  ```yaml
  version: '3.8'
  services:
    code-server:
      image: codercom/code-server:latest
      container_name: code-server
      ports:
        - "8081:8080"  # Changed port mapping
      volumes:
        - code-server-data:/home/coder/project
      environment:
        - PASSWORD=0000  # Use a more secure password
      restart: unless-stopped
  volumes:
    code-server-data:
  ```
</details>

---

[paypal_me_shield]: https://img.shields.io/badge/PayPal-00457C?style=for-the-badge&logo=paypal&logoColor=white

[paypal_me]: https://paypal.me/anasboxsupport

[revolut_me_shield]:
https://img.shields.io/badge/revolut-FFFFFF?style=for-the-badge&logo=revolut&logoColor=black

[revolut_me]: https://revolut.me/anas4e

[ko_fi_shield]: https://img.shields.io/badge/Ko--fi-F16061?style=for-the-badge&logo=ko-fi&logoColor=white

[ko_fi_me]: https://ko-fi.com/anasbox

[buy_me_coffee_shield]: 
https://img.shields.io/badge/Buy%20Me%20Coffee-ffdd00?style=for-the-badge&logo=buy-me-a-coffee&logoColor=black

[buy_me_coffee_me]: https://www.buymeacoffee.com/anasbox
