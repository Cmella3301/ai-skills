---
name: setup-nodejs-pwa
description: Sets up a self-hosted Node.js web app from scratch and converts it into an installable Progressive Web App (PWA). Handles Node.js installation, npm setup, PowerShell execution policy fixes, and all PWA assets (manifest, service worker, icons).
---

# Skill: Setup Node.js App as a PWA

Use this skill whenever the user wants to:
- Bootstrap a new Node.js/Express app with a `package.json`
- Install Node.js on a Windows machine
- Convert an existing self-hosted web app into a PWA
- Make an app installable from the browser (Chrome/Edge install prompt)

---

## Step 1 — Create `package.json`

If no `package.json` exists, create one in the project root. Standard template for a self-hosted Express app:

```json
{
  "name": "<app-name>",
  "version": "1.0.0",
  "description": "<description>",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "sqlite3": "^5.1.6",
    "cors": "^2.8.5",
    "body-parser": "^1.20.2"
  },
  "engines": {
    "node": ">=16.0.0"
  }
}
```

---

## Step 2 — Install Node.js (Windows)

If `npm` is not found, install Node.js via winget (preferred — avoids browser download errors):

```powershell
winget install OpenJS.NodeJS.LTS
```

- This will prompt for **UAC (administrator)** approval — the user must click Yes.
- After install, the **current terminal session won't have Node in PATH**. Must use a new terminal OR set PATH manually.

### Fix PATH in current session (avoids reopening terminal):
```powershell
$env:PATH = "C:\Program Files\nodejs;" + $env:PATH
```

---

## Step 3 — Fix PowerShell Execution Policy

On many Windows machines, PowerShell blocks npm scripts with:
> `npm.ps1 cannot be loaded because running scripts is disabled on this system`

Fix it with:
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser -Force
```

Then use the full path to npm if still not recognized:
```powershell
& "C:\Program Files\nodejs\npm.cmd" install
```

Combined one-liner (most reliable):
```powershell
$env:PATH = "C:\Program Files\nodejs;" + $env:PATH; & "C:\Program Files\nodejs\npm.cmd" install
```

---

## Step 4 — Install Dependencies

```powershell
$env:PATH = "C:\Program Files\nodejs;" + $env:PATH
& "C:\Program Files\nodejs\npm.cmd" install
```

Warnings about deprecated packages are normal and can be ignored.

---

## Step 5 — Start the Server

```powershell
$env:PATH = "C:\Program Files\nodejs;" + $env:PATH
& "C:\Program Files\nodejs\node.exe" server.js
```

Or if PATH is already set in a new terminal:
```powershell
npm start
```

The app will be available at `http://localhost:<PORT>` (default 3001).

---

## Step 6 — Add PWA Support

### 6a. Create `manifest.json`

Place in project root:

```json
{
  "name": "<App Name>",
  "short_name": "<App>",
  "description": "<description>",
  "start_url": "/",
  "display": "standalone",
  "background_color": "<bg-color>",
  "theme_color": "<theme-color>",
  "orientation": "portrait-primary",
  "icons": [
    {
      "src": "/icons/icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/icons/icon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any maskable"
    }
  ],
  "categories": ["productivity"]
}
```

### 6b. Create `sw.js` (Service Worker)

Place in project root. Uses **network-first for `/api/` routes** and **cache-first for the UI shell**:

```js
const CACHE_NAME = '<app-name>-v1';
const STATIC_ASSETS = ['/', '/index.html', '/manifest.json'];

self.addEventListener('install', (event) => {
  event.waitUntil(caches.open(CACHE_NAME).then(c => c.addAll(STATIC_ASSETS)));
  self.skipWaiting();
});

self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then(keys =>
      Promise.all(keys.filter(k => k !== CACHE_NAME).map(k => caches.delete(k)))
    )
  );
  self.clients.claim();
});

self.addEventListener('fetch', (event) => {
  const url = new URL(event.request.url);
  if (url.pathname.startsWith('/api/')) {
    event.respondWith(fetch(event.request).catch(() =>
      new Response(JSON.stringify({ error: 'Offline' }), {
        headers: { 'Content-Type': 'application/json' }
      })
    ));
    return;
  }
  event.respondWith(
    caches.match(event.request).then(cached => {
      return cached || fetch(event.request).then(response => {
        if (event.request.method === 'GET' && response.status === 200) {
          caches.open(CACHE_NAME).then(c => c.put(event.request, response.clone()));
        }
        return response;
      });
    }).catch(() => {
      if (event.request.mode === 'navigate') return caches.match('/index.html');
    })
  );
});
```

### 6c. Generate App Icons

Use `generate_image` tool to create a square icon matching the app's color scheme. Save as:
- `icons/icon-192.png`
- `icons/icon-512.png`

### 6d. Inject PWA Tags into `index.html`

Add to the `<head>` section, **before the `<style>` tag**:

```html
<!-- PWA -->
<link rel="manifest" href="/manifest.json" />
<meta name="theme-color" content="<theme-color>" />
<meta name="mobile-web-app-capable" content="yes" />
<meta name="apple-mobile-web-app-capable" content="yes" />
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
<meta name="apple-mobile-web-app-title" content="<App Name>" />
<link rel="apple-touch-icon" href="/icons/icon-192.png" />
<link rel="icon" type="image/png" sizes="192x192" href="/icons/icon-192.png" />

<!-- Service Worker Registration -->
<script>
  if ('serviceWorker' in navigator) {
    window.addEventListener('load', () => {
      navigator.serviceWorker.register('/sw.js')
        .then(reg => console.log('SW registered:', reg.scope))
        .catch(err => console.warn('SW registration failed:', err));
    });
  }
</script>
```

---

## Step 7 — Verify PWA Install Works

1. Restart the server
2. Open **Chrome** or **Edge** and navigate to `http://localhost:<PORT>`
3. Look for the **install icon (⊕)** in the address bar
4. Click **Install** — the app installs to Start menu and taskbar

> ⚠️ PWA desktop install requires Chrome or Edge. Firefox does not support it.

---

## Notes & Gotchas

| Issue | Fix |
|-------|-----|
| `npm` not recognized after Node install | Set `$env:PATH = "C:\Program Files\nodejs;" + $env:PATH` |
| `npm.ps1` blocked by execution policy | Run `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser -Force` |
| `sqlite3` fails to build | Ensure `node.exe` is in PATH before running `npm install` |
| No install prompt in browser | Must use HTTPS or localhost; Chrome/Edge only |
| Icons not loading | Ensure `server.js` serves static files with `app.use(express.static(__dirname))` |
