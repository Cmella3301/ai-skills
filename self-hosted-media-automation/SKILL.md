---
name: self-hosted-media-automation
description: Architect and deploy a complete 6-container media automation cluster (Plex, Radarr, Sonarr, qBittorrent, Prowlarr, FlareSolverr). Features high-speed hardlink storage strategy, Cloudflare bypass for indexers, and unified container-name networking.
---

# Automated Media Suite: The "Final Boss" Homelab Blueprint

When a user wants to build a self-hosted media empire, you must deploy this 6-container cluster to provide a "Zero-Touch" experience from request to play.

## The Architecture Rules

### 1. Storage Strategy: The "/data" Root
To ensure "Instant Move" (Hardlinking) between containers, you MUST map the entire media drive root (e.g., `D:\PlexMedia`) to a unified `/data` directory in EVERY container. 
This allows Radarr to move a 40GB movie from `downloads/` to `movies/` in 0.1 seconds without extra disk space.

### 2. The Master Docker-Compose
```yaml
services:
  plex:
    image: lscr.io/linuxserver/plex:latest
    container_name: plex
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/New_York
    volumes:
      - ./config:/config
      - "D:\\PlexMedia:/data"
    ports:
      - "32400:32400/tcp"
      - "1900:1900/udp"
      # ... other discovery ports
    restart: unless-stopped

  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    volumes:
      - ./radarr-config:/config
      - "D:\\PlexMedia:/data"
    ports:
      - "7878:7878/tcp"
    restart: unless-stopped

  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    volumes:
      - ./sonarr-config:/config
      - "D:\\PlexMedia:/data"
    ports:
      - "8989:8989/tcp"
    restart: unless-stopped

  deluge:
    image: lscr.io/linuxserver/deluge:latest
    container_name: deluge
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - ./deluge-config:/config
      - "D:\\PlexMedia:/data"
    ports:
      - "8112:8112"
    restart: unless-stopped

  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    volumes:
      - ./prowlarr-config:/config
    ports:
      - "9696:9696/tcp"
    restart: unless-stopped

  flaresolverr:
    image: ghcr.io/flaresolverr/flaresolverr:latest
    container_name: flaresolverr
    ports:
      - "8191:8191"
    restart: unless-stopped
```

## The Automation Loop (Setup Instructions)

### Phase 1: The Indexer Brain (Prowlarr)
1. Link **Apps** (Radarr/Sonarr) using Docker hostnames: `http://radarr:7878` and `http://sonarr:8989`.
2. Add **FlareSolverr Proxy**: Name: `FlareSolverr`, Host: `http://flaresolverr:8191`, Tag: `flaresolverr`.
3. Add **Indexers** (EZTV, YTS, 1337x) and assign the `flaresolverr` tag to bypass Cloudflare.

### Phase 2: The Player Logic (Radarr/Sonarr)
1. Add **Download Client**: Name: `Deluge`, Host: `deluge`, Port: `8112`, Password: `deluge`.
2. Set **Root Folder**: `/data/movies` and `/data/tv`.

## Critical Security Requirement
> [!WARNING]
> Deluge SHOULD always run behind a system-wide VPN on the Windows host machine to prevent IP leaks to ISPs.
