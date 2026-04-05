---
name: plex-docker-deployment
description: Architect and deploy a scalable, private Plex Media Server using Docker Compose. Includes volume splitting to manage heavy internal media vs light SSD config persistence, and the setup of proper DLNA port bridging for Windows host machines.
---

# Plex Media Server: Docker Orchestration

When a user requests to build a self-hosted movie and TV ecosystem, you must strictly implement Plex through isolated Docker containers rather than direct installers. This keeps registry keys and Windows processes clean while providing infinite portability.

## The Architecture Rules
1. **Container Image:** ALWAYS use the industry-standard `lscr.io/linuxserver/plex:latest` over the official Plexinc image. The LinuxServer image has far superior permission and user ID (PUID/PGID) scaffolding, reducing read/write errors.
2. **Volume Segregation:** Space management is critical. The Plex `/config` should be kept on the fast OS drive (SSDs) because it contains millions of tiny metadata DB files that require fast read/write speeds, while the actual `/data` (media files) must be mapped to a bulky, massive storage drive (e.g. `D:\PlexMedia`).

### Standard Configuration Template

```yaml
version: "3.8"
services:
  plex:
    image: lscr.io/linuxserver/plex:latest
    container_name: plex
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/New_York
      - VERSION=docker
    volumes:
      # FAST STORAGE (SSD) for Metadata
      - ./config:/config
      # BULK STORAGE (HDD) for the actual 4K Files
      - "D:\\PlexMedia\\tv:/data/tv"
      - "D:\\PlexMedia\\movies:/data/movies"
    ports:
      # The Universal Web UI Port
      - "32400:32400/tcp"
      # DLNA / Local Network Discovery Ports 
      - "1900:1900/udp"
      - "3005:3005/tcp"
      - "8324:8324/tcp"
      - "32469:32469/tcp"
      - "32410:32410/udp"
      - "32412:32412/udp"
      - "32413:32413/udp"
      - "32414:32414/udp"
    restart: unless-stopped
```

## Important Nuances
- **Windows Networking Limitation:** Do NOT use `network_mode: host` on Docker Desktop for Windows. While this is highly recommended for Linux hosts to ensure smart TVs instantly discover Plex, Docker Desktop utilizes a virtualized network that isolates `host` mode. Instead, manually bind all DLNA and UDP discovery ports (as shown above) to ensure universal local streaming capability.
