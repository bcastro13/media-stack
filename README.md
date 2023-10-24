# media_server

A stack of self-hosted media managers and streamer along with VPN.
- [Requirements](#requirements)
- [Installation](#installation)
- [Links to Services](#links-to-services)
- [Setup Services](#setup-services)
- [Configure Nginx (Optional)](#configure-nginx-optional)
    - [Apply SSL in Nginx](#apply-ssl-in-nginx)
    - [Radarr Nginx Reverse Proxy](#radarr-nginx-reverse-proxy)
    - [Sonarr Nginx Reverse Proxy](#sonarr-nginx-reverse-proxy)
    - [Prowlarr Nginx Reverse Proxy](#prowlarr-nginx-reverse-proxy)
    - [qBitTorrent Nginx Reverse Proxy](#qbittorrent-nginx-reverse-proxy)
    - [Jellyfin Nginx Reverse Proxy](#jellyfin-nginx-reverse-proxy)


## Requirements

- Docker version 23.0.5 and above
- Docker compose version v2.17.3 and above
- It may also work on some of lower versions, but its not tested.

## Installation
Edit the docker-composed.yml file to suit your config. The steps are numbered.

Run the following commands
```
docker network create mynetwork
docker compose up -d
```

Optional if you want to use Nginx as a reverse proxy
```
docker compose -f docker-compose-nginx.yml up -d
```

## Links to Services

- Jellyfin
    - http://localhost:8096
- Jellyseer
    - http://localhost:5055
- Radarr
    - http://localhost:7878
- Sonarr
    - http://localhost:8989
- Prowlarr
    - http://localhost:9696
- qBitTorrent
    - http://localhost:5080

## Setup Services
1. qBitTorrent
    - Default login is username: admin, password: changeme
    - Change the password in settings under the Web UI tab
2. Prowlarr
    - Default login is username: admin, password: changeme
    - Change the password in settings under the General tab. Make sure to save the changes
    - Create a new api key and click restart
3. Sonarr
    - Default login is username: admin, password: changeme
    - Change the password in settings under the General tab. Make sure to save the changes
    - Create a new api code and click restart
    - Change the qBittorrent login credentials under the Download Clients tab
    - Update the Prowlarr api key for all the indexers under the Indexers tab
4. Radarr
    - Default login is username: admin, password: changeme
    - Change the password in settings under the General tab. Make sure to save the changes
    - Create a new api code and click restart
    - Change the qBittorrent login credentials under the Download Clients tab
    - Update the Prowlarr api key for all the indexers under the Indexers tab
5. Jellyfin
    - Walkthrough setup guide taking note of username and password
6. Jellyseer
    - Walkthrough setup guide
    - Sign in with same Jellyfin credentials when prompted
    - Use api keys for Sonarr and Radarr when prompted
    - Default Sonarr profile to use (choose based on personal preference):
        - WEB-2160p
        - WEB-1080p
    - Default Radarr profile to use (choose based on personal preference):
        - UHD Bluray + WEB
        - HD Bluray + WEB
    - Notes:
        - Quality profiles can be selected on a per request basis
        - When requesting older content prefer the 1080/HD profiles

## Configure Nginx (Optional)
### NOTE: This has not be tested

- Get inside Nginx container
- `cd /etc/nginx/conf.d`
- Add proxies for all tools.

`docker cp nginx.conf nginx:/etc/nginx/conf.d/default.conf && docker exec -it nginx nginx -s reload`
- Close ports of other tools in firewall/security groups except port 80 and 443.


### Apply SSL in Nginx

- Open port 80 and 443.
- Get inside Nginx container and install certbot and certbot-nginx `apk add certbot certbot-nginx`
- Add URL in server block. e.g. `server_name  localhost armdev.navratangupta.in;` in /etc/nginx/conf.d/default.conf
- Run `certbot --nginx` and provide details asked.

### Radarr Nginx Reverse Proxy

- Settings --> General --> URL Base --> Add base (/radarr)
- Add below proxy in nginx configuration

```
location /radarr {
    proxy_pass http://radarr:7878;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $http_connection;
  }
```

- Restart containers.

### Sonarr Nginx Reverse Proxy

- Settings --> General --> URL Base --> Add base (/sonarr)
- Add below proxy in nginx configuration

```
location /radarr {
    proxy_pass http://sonarr:8989;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $http_connection;
  }
```

### Prowlarr Nginx Reverse Proxy

- Settings --> General --> URL Base --> Add base (/prowlarr)
- Add below proxy in nginx configuration

This may need to change configurations in indexers and base in URL.

```
location /prowlarr {
    proxy_pass http://prowlarr:9696;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $http_connection;
  }
```

- Restart containers.

### qBittorrent Nginx Reverse Proxy

```
location /qbt/ {
    proxy_pass         http://qbittorrent:5080/;
    proxy_http_version 1.1;

    proxy_set_header   Host               http://qbittorrent:5080;
    proxy_set_header   X-Forwarded-Host   $http_host;
    proxy_set_header   X-Forwarded-For    $remote_addr;
    proxy_cookie_path  /                  "/; Secure";
}
```

**Note: If VPN is enabled, then qbittorrent is reachable on vpn's service name**

### Jellyfin Nginx Reverse Proxy

- Add base URL, Admin Dashboard -> Networking -> Base URL (/jellyfin)
- Add below config in Ngix config

```
 location /jellyfin {
        return 302 $scheme://$host/jellyfin/;
    }

    location /jellyfin/ {

        proxy_pass http://jellyfin:8096/jellyfin/;

        proxy_pass_request_headers on;

        proxy_set_header Host $host;

        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $http_host;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $http_connection;

        # Disable buffering when the nginx proxy gets very resource heavy upon streaming
        proxy_buffering off;
    }
```

- Restart containers
