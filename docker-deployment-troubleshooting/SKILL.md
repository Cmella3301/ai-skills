---
name: docker-deployment-troubleshooting
description: Debug and fix Docker containerized applications that won't start or respond. Use this skill when a user has a Docker application that is failing to run, not responding on expected ports, crashing on startup, or showing connection errors. This skill provides systematic diagnostics for container issues, database problems, port conflicts, volume mounting issues, and configuration errors. Works with docker-compose and standalone Docker containers. Perfect for when users say "my Docker app won't start," "can't connect to localhost," "container keeps crashing," "Docker logs show errors," or when deployment gets stuck during troubleshooting.
---

# Docker Deployment Troubleshooting Skill

Systematically diagnose and fix Docker containerized applications that fail to start, crash, or don't respond on expected ports.

## When to Use This Skill

✅ **Use when the user has:**
- Docker application won't start/run
- Container keeps crashing or exiting
- Can't connect to localhost:PORT (ERR_CONNECTION_REFUSED)
- Application runs but doesn't respond
- Port conflicts or not exposed correctly
- Database/volume mounting issues
- Permission errors in container
- Application times out or hangs

❌ **Don't use if:**
- User just wants help writing Docker files
- They need to learn Docker basics
- They're building Docker images from scratch (use docker-build skill instead)

## Workflow

### Step 1: Gather Context

Before troubleshooting, ask for:
1. **Container status** - `docker-compose ps` or `docker ps` output
2. **Full logs** - `docker-compose logs` or `docker logs <container>`
3. **Project structure** - What files are in the folder?
4. **Expected behavior** - What should happen vs. what's happening?
5. **Last changes** - What was done before it broke?
6. **Environment** - OS, Docker version, any special networking

### Step 2: Systematic Diagnosis

**Check 1: Container Running?**
```bash
docker-compose ps
# or
docker ps -a
```
Look for:
- Status: "Up" (good) vs "Exited" (crashed)
- Exit code: 0 (clean) vs non-zero (error)
- Uptime: How long has it been running?

**Check 2: Container Logs**
```bash
docker-compose logs --tail 50
# or
docker logs <container-id>
```
Common error patterns:
- `EADDRINUSE` - Port already in use
- `ENOENT` - Missing file
- `SQLITE_CANTOPEN` - Database file permission
- `Connection refused` - Service not listening
- Syntax errors - Code issues in app
- Module not found - Dependency missing

**Check 3: Port Accessible?**
```bash
# Test if port is listening
curl http://localhost:3001

# Check what's on the port
netstat -ano | findstr :3001  # Windows
lsof -i :3001                  # Mac/Linux

# Check Docker port forwarding
docker port <container-id>
```

**Check 4: Volume Mounting**
```bash
docker inspect <container-id> | grep -A 10 Mounts
# Check if volumes are mounted correctly
# Verify host paths exist
```

**Check 5: Environment Variables**
```bash
docker inspect <container-id> | grep -A 20 Env
# Check if all required ENV vars are set
```

**Check 6: Network**
```bash
docker network ls
docker network inspect <network>
# Verify container is on correct network
```

## Common Issues & Fixes

### Issue 1: Container Exits Immediately

**Symptoms:**
- `docker ps` shows container with status "Exited" or "Exited (1)"
- Container runs for 1-2 seconds then stops

**Diagnosis:**
```bash
docker logs <container>
```

**Common Causes & Fixes:**

**A) Application crashes on startup**
- Check logs for error messages
- Look for missing files, config errors, syntax errors
- Verify all environment variables are set
- Check if dependencies are installed

**B) Port already in use**
Error: `EADDRINUSE: address already in use :::3001`
```bash
# Find what's using the port
netstat -ano | findstr :3001

# Either:
# 1. Kill the other process
# 2. Change the port in docker-compose.yml
# 3. Use a different port
```

**C) Working directory issues**
Error: `ENOENT: no such file or directory`
```bash
# Check Dockerfile WORKDIR
# Verify COPY commands are correct
# Make sure files exist in build context
```

**D) Database file permission**
Error: `SQLITE_CANTOPEN: unable to open database file`
```bash
# Fix 1: Create data directory with correct permissions
mkdir -p ./data
chmod 755 ./data

# Fix 2: Update docker-compose.yml volumes:
volumes:
  - ./data:/app/data
  - ./app.db:/app/app.db
```

### Issue 2: Connection Refused

**Symptoms:**
- `curl localhost:3001` → Connection refused
- Browser shows ERR_CONNECTION_REFUSED
- Container is running (status = "Up")

**Diagnosis:**
```bash
# 1. Check if container is listening
docker exec <container> netstat -tlnp
# or
docker exec <container> lsof -i -P -n

# 2. Check port forwarding
docker port <container>

# 3. Check if bound to 127.0.0.1 instead of 0.0.0.0
```

**Common Causes & Fixes:**

**A) App listening on wrong interface**
```javascript
// ❌ Wrong - only localhost
app.listen(3001, 'localhost')

// ✅ Correct - all interfaces
app.listen(3001, '0.0.0.0')
// or just
app.listen(3001)
```

**B) Wrong port in docker-compose.yml**
```yaml
# ❌ Wrong
ports:
  - "3002:3001"  # Maps 3002 to 3001

# ✅ Correct
ports:
  - "3001:3001"  # Maps 3001 to 3001
```

**C) Port not exposed in Dockerfile**
```dockerfile
# Add this if missing
EXPOSE 3001
```

**D) Firewall blocking**
Windows:
```bash
# Add Windows Firewall exception
netsh advfirewall firewall add rule name="Docker Port 3001" dir=in action=allow protocol=tcp localport=3001
```

### Issue 3: App Runs But Returns Errors

**Symptoms:**
- Container is "Up"
- Page loads but shows errors
- API endpoints return 500 errors

**Diagnosis:**
```bash
# Check full application logs
docker logs <container> --tail 100

# Check for specific errors
docker logs <container> | grep -i error
docker logs <container> | grep -i warning
```

**Common Causes & Fixes:**

**A) Missing environment variables**
```bash
docker inspect <container> | grep -A 30 Env
# Compare with required vars in app
```

**B) Database not initialized**
- Check if database file exists
- Verify schema creation ran
- Check database permissions

**C) Dependency issues**
```bash
# Rebuild with fresh dependencies
docker-compose down
docker system prune -a
docker-compose up -d --build
```

### Issue 4: Database Errors

**Symptoms:**
- `SQLITE_CANTOPEN` errors
- `Cannot read database file`
- Data not persisting

**Diagnosis:**
```bash
# Check volume mounting
docker inspect <container> | grep -A 10 Mounts

# Check if file exists in container
docker exec <container> ls -la /app/

# Check file permissions
docker exec <container> ls -la /app/database.db
```

**Fixes:**

**A) Volume not mounted**
```yaml
# Fix docker-compose.yml
volumes:
  - ./data:/app/data
  - ./app.db:/app/app.db
```

**B) Wrong permissions**
```bash
# Fix from host
chmod 666 app.db
chmod 755 data/

# Or in Dockerfile
RUN chmod 755 /app/data
```

**C) Wrong database path**
```javascript
// ❌ Wrong path in container context
const db = new sqlite3.Database('/myapp/data.db')

// ✅ Correct
const db = new sqlite3.Database('/app/data.db')
```

### Issue 5: Build Failures

**Symptoms:**
- `docker-compose up --build` fails
- Docker build process stops with errors
- Image won't build

**Diagnosis:**
```bash
# Rebuild with verbose output
docker-compose build --no-cache --verbose
```

**Common Causes & Fixes:**

**A) Missing files**
Error: `failed to solve: failed to copy ... no such file or directory`
```dockerfile
# ❌ File doesn't exist
COPY missing-file.txt .

# ✅ Verify file exists in build directory
# Or fix the path
COPY package.json .
```

**B) Package install fails**
Error: `npm ERR! code EUSAGE`
```bash
# Usually missing package-lock.json
# Solutions:
# 1. Generate it: npm ci > npm-shrinkwrap.json
# 2. Use npm install instead: RUN npm install
# 3. Update Dockerfile npm command
```

**C) Network issues during build**
Error: `failed to solve: process exited with status 1`
```bash
# Try rebuild with fresh start
docker-compose down
docker system prune -a -f
docker-compose up -d --build
```

### Issue 6: Stale Container (Not Showing Local File Edits)

**Symptoms:**
- You changed code or replaced an image locally (e.g. `index.html` or `icon.png`)
- The app still shows the old version, or "nothing changed"
- The container is currently running without errors

**Diagnosis:**
Docker builds an "image" containing a snapshot of your files at the exact moment you run build. If you edit files on your hard drive *after* that, the container has no idea they changed unless you explicitly mapped a volume.

**Fix: The "Nuke and Rebuild" Method**
When you absolutely need Docker to scrap its cache and pull your new files, use these commands:

```bash
# 1. Stop the current containers
docker-compose down

# 2. Obliterate Docker's cache and old images completely
docker system prune -a

# 3. Rebuild from scratch and start
docker-compose up -d --build
```
> ⚠️ **Warning:** `docker system prune -a` deletes ALL unused Docker images on your system. It is highly effective for breaking cache, but will require re-downloading bases like `node:alpine` the next time you build.

## Diagnostic Commands Cheat Sheet

```bash
# Container status
docker-compose ps
docker ps -a
docker inspect <container>

# Logs
docker-compose logs
docker-compose logs --tail 50
docker logs <container> --follow

# Exec into container
docker exec -it <container> /bin/sh
docker exec <container> ls -la /app

# Network
docker network ls
docker network inspect <network>
docker port <container>

# Clean up
docker-compose down
docker system prune -a
docker volume rm <volume>

# Rebuild
docker-compose build --no-cache
docker-compose up -d --build

# Health check
curl http://localhost:3001
docker exec <container> curl http://localhost:3001
```

## Debugging Checklist

When troubleshooting, go through this systematically:

- [ ] Container running? (`docker-compose ps`)
- [ ] Check logs for errors (`docker logs`)
- [ ] Port accessible? (`curl localhost:PORT`)
- [ ] Port correctly exposed in docker-compose.yml?
- [ ] Port correctly exposed in Dockerfile?
- [ ] App listening on 0.0.0.0 not 127.0.0.1?
- [ ] All environment variables set?
- [ ] All required files copied into image?
- [ ] Volume paths correct?
- [ ] Database file permissions?
- [ ] Dependencies installed? (check package.json)
- [ ] No conflicting processes on port?
- [ ] Firewall blocking the port?
- [ ] Network correct? (docker-compose network)
- [ ] Resource limits reached? (memory, CPU)

## File Organization

For successful deployments, ensure:

```
my-app/
├── Dockerfile              # Correct COPY/WORKDIR
├── docker-compose.yml      # Correct ports & volumes
├── package.json           # All dependencies listed
├── package-lock.json      # If using Node
├── app.js                 # Main application file
├── index.html            # If serving static files
├── .dockerignore         # Optimize builds
└── data/                 # Data directory (if needed)
```

## Prevention Tips

1. **Always test locally first** - before Docker
2. **Use explicit ports** - avoid dynamic port assignment
3. **Volume mount for development** - easier debugging
4. **Keep logs accessible** - don't redirect to /dev/null
5. **Health checks** - add HEALTHCHECK to Dockerfile
6. **Proper error handling** - catch and log errors
7. **Environment variables** - never hardcode secrets
8. **Resource limits** - define memory/CPU limits
9. **Restart policies** - handle crashes gracefully
10. **Version pinning** - specify base image versions

## When to Escalate

If after systematic diagnosis you find:
- Base image is broken
- Docker daemon issues
- System-level network problems
- Kernel compatibility issues

Then recommend:
- Update Docker
- Check Docker logs: `docker logs dockerd`
- Restart Docker daemon
- Check Docker system: `docker system info`

## Example: Complete Debugging Session

```bash
# 1. Check container status
docker-compose ps
# Output: lock-in-app  Exited (1) 2 seconds ago

# 2. Get logs
docker-compose logs
# Output: Error: Cannot find module 'express'

# 3. Rebuild with fresh install
docker-compose down
docker system prune -a -f
docker-compose up -d --build

# 4. Verify
docker-compose ps
# Output: lock-in-app  Up 3 seconds

# 5. Test
curl http://localhost:3001
# Output: Success!
```

---

## Output Checklist

When helping user fix Docker issues, deliver:

- [x] Root cause identified
- [x] Specific error explained
- [x] Fix provided (code/config change)
- [x] Command to test
- [x] Verification steps
- [x] Prevention tips for future

---

This skill provides methodical, systematic approaches to debugging Docker deployment issues without requiring users to understand Docker internals.
