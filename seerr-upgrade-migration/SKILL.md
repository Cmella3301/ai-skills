---
name: Seerr Upgrade & Migration
description: Upgrades a standard Overseerr installation to its modern community successor, Seerr, while handling persistent permission/log errors.
---

# 🚀 Seerr Upgrade & Migration Skill

This skill allows you to safely transition a "frozen" Overseerr deployment to the modern, actively maintained **Seerr** codebase.

## 🛠️ Step 1: Update the Infrastructure
Modify your `docker-compose.yml` to swap the service engine.

```yaml
  seerr:
    image: ghcr.io/seerr-team/seerr:latest
    init: true  # Required for the Seerr process manager
    container_name: seerr
    # ... keep your existing networks, environment, and ports ...
    volumes:
      - ./overseerr-config:/app/config  # Reuse your existing config folder
    restart: unless-stopped
```

## 🔧 Step 2: Clear Permission Bottlenecks
Seerr runs as a non-root user (`node`). If your old logs are locked by the root user, the container will crash with `EACCES`.

**The 60-Second Fix:**
1. Stop any running Overseerr/Seerr containers.
2. Delete the old `logs` directory inside your config folder (e.g., `./overseerr-config/logs`).
3. Restart the container. Seerr will automatically recreate the logs with correct permissions.

## ✅ Step 3: Verification
1. Access the UI at `http://localhost:5055`.
2. verify the **[info][Seerr Migration]: Yeah! Overseerr to Seerr migration completed successfully!** message in the logs.
3. Your Plex, Radarr, and Sonarr connections will be preserved automatically.

> [!TIP]
> Seerr is significantly faster than Overseerr and includes updated API support for the latest versions of Plex and the *Arr stack.
