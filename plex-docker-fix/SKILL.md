---
name: plex-docker-troubleshooting
description: Debug and fix connectivity issues in a 6-container Plex Media Cluster on Windows Docker Desktop. Resolves 'Unable to connect securely' and 'Empty Response' errors by aligning local subnet white-listing and firewall port unblocking.
---

# Plex Docker Troubleshooting Cluster Fix

When a Plex Media Server is running inside Docker on Windows Desktop, it often suffers from "Loopback Blindness" or "Playback Errors."

## The 5-Step "Master Fix"

### 1. The Preferences "Handshake" Fix
If the server says "Unavailable" or "Empty Response," verify the `Preferences.xml`. Ensure `secureConnections="0"` for local HTTP if needed and `allowedNetworks` is white-listed for the LAN.

### 2. The Docker Identity Fix
In the `docker-compose.yml`, ensure the server is advertising its **Actual Host IP** and using a **Fresh Claim Token**.

### 3. The "Master Reset" (Identity Wipe)
If the server is stuck in an identity loop, perform a "Factory Reset" by renaming `Preferences.xml` to `Preferences.old` and re-claiming with a fresh token.

### 4. The "Smooth Motion" (Transcode) Fix
If playback fails with "Unexpected Error," map a dedicated `/transcode` volume to a fast local drive (e.g., `D:\PlexMedia\transcode`) to give the engine more workspace.

### 5. The Windows "Gatekeeper" Fix
Windows Firewall often blocks Port 32400. Use:
```powershell
New-NetFirewallRule -DisplayName "Plex Local" -Direction Inbound -LocalPort 32400 -Protocol TCP -Action Allow -Profile Private
```

## Critical Architecture Notes
- **Paths:** Always map `/data` and `/transcode` to the host media drive (D:).