# LOCK-IN App Deployment - Handover to Claude Opus

## Current Status

**User:** Wella (Hamilton, Ontario)
**Project:** Self-hosted LOCK-IN productivity/habit tracking app
**Platform:** Windows 10/11, Docker Desktop
**Location:** `C:\Users\Wella\Documents\locke-in-app`

## Files in Folder

All files have been extracted and are ready:
- ✅ server.js (edited - lines changed for static file serving)
- ✅ package.json (correct - no "type": "module")
- ✅ Dockerfile (edited - added `COPY index.html .`)
- ✅ docker-compose.yml
- ✅ index.html (the original LOCKIN_Prod_5.html UI)
- ✅ .env.example
- ✅ .dockerignore
- ✅ README.md, QUICK_START.md, DEPLOYMENT.md

## Problem We're Trying to Solve

The app is not loading at `http://localhost:3001`

**Current Symptoms:**
1. Server container keeps crashing or not staying running
2. Getting "ERR_CONNECTION_REFUSED" errors
3. Docker logs show container exiting repeatedly

## Server.js Changes Made

**Line ~12:** Changed from:
```javascript
app.use(express.static('public'));
```
To:
```javascript
app.use(express.static(__dirname));
```

**Lines ~313-320:** Changed from:
```javascript
app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'index.html'));
});
```
To:
```javascript
app.get('/', (req, res) => {
  res.sendFile(path.join(__dirname, 'index.html'));
});

app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname, 'index.html'));
});
```

## Dockerfile Changes Made

**Added line after `COPY server.js .`:**
```
COPY index.html .
```

And commented out (kept as reference):
```
# COPY public/ ./public/
```

## Last Attempted Commands

```bash
docker-compose down
docker system prune -a -f
docker-compose up -d --build
```

Result: Connection refused when trying to access localhost:3001

## What Needs to Happen

1. **Diagnose why container keeps crashing**
   - Check docker-compose ps output
   - Review docker-compose logs
   - Verify container stays in "Up" state

2. **Ensure server.js runs correctly**
   - The server should start without errors
   - Should listen on port 3001
   - Should serve index.html on GET /

3. **Verify database initialization**
   - SQLite database should create on first run
   - Tables should be created automatically
   - Data should persist in Docker volume

4. **Get the app accessible**
   - http://localhost:3001 should show LOCK-IN app
   - Should be the dark productivity UI with habit rings
   - Should respond to API calls from the HTML app

## Original HTML App Info

- **Name:** LOCK-IN
- **Type:** Habit tracking / productivity app
- **Features:** 
  - Habit rings with goals
  - Daily tracking
  - Calendar view
  - Journal entries
  - Schedule blocks
  - Statistics/XP system
  - GitHub-style heatmap visualization

- **Size:** 159 KB (index.html)
- **Architecture:** Single HTML file with embedded CSS/JS

## What Backend Should Do

- Serve the index.html on port 3001
- Provide REST API endpoints for:
  - GET /api/data - fetch all data
  - POST /api/ring/:ringId/value - update habit value
  - POST/DELETE /api/rings - manage habits
  - POST /api/journal/:date - save journal entries
  - POST/DELETE /api/schedule - manage schedule
  - POST /api/stats - update statistics
  - GET /api/health - health check

- Persist data to SQLite database (lockin.db)
- Auto-create database schema on startup

## Known Issues to Address

1. Container may not be staying alive
2. Database file might have permission issues (SQLITE_CANTOPEN errors seen in logs)
3. Port 3001 might not be properly exposed
4. Volume mounting for data persistence might need adjustment

## Docker-Compose Configuration

```yaml
services:
  lock-in:
    build: .
    container_name: lock-in-app
    ports:
      - "3001:3001"
    volumes:
      - lock-in-data:/app/data
      - ./lockin.db:/app/lockin.db
    environment:
      - NODE_ENV=production
      - PORT=3001
    restart: unless-stopped

volumes:
  lock-in-data:
    driver: local
```

## How to Test When Fixed

1. Run: `docker-compose ps` → should show "Up"
2. Run: `docker-compose logs` → should show "LOCK-IN server running on http://localhost:3001"
3. Navigate to: `http://localhost:3001`
4. Should see: Dark purple/black LOCK-IN interface with habit tracking rings
5. API test: `curl http://localhost:3001/api/health` → should return JSON with status: "ok"

## User's Environment

- **OS:** Windows 11
- **Docker:** Docker Desktop installed and running
- **Terminal:** PowerShell
- **Project Path:** C:\Users\Wella\Documents\locke-in-app

## Next Steps

1. **Verify current state** - Check if container is running
2. **Debug startup** - Look at full docker logs
3. **Fix any blocking issues** - Address root cause of crashes
4. **Test basic connectivity** - Ensure server responds
5. **Verify UI loads** - Confirm index.html serves correctly
6. **Test API endpoints** - Confirm backend responds

---

**Created:** April 3, 2026
**Status:** In Progress - Deployment phase
**Owner:** Wella
**Handoff:** To Claude Opus for troubleshooting
