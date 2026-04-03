---
name: pwa-identity-update
description: Update a Progressive Web App's icon and layout, including PowerShell script to resize images automatically, manifest configuration for true fullscreen, and the browser re-installation process.
---

# Skill: Update PWA Identity & Icons

Use this skill when you need to change a PWA's app icon, convert an image to the exact PWA requirements (192x192 and 512x512) on Windows without external tools, or upgrade the app's display to true fullscreen without the browser title bar.

---

## Part 1 — Resizing Icons via PowerShell

PWAs strictly require `192x192` and `512x512` PNG images declared in the manifest. If you have a high-res square image (e.g. 1024x1024), you can use this PowerShell script to perfectly resize and save them — no Photoshop required.

Run this in your terminal, modifying `$sourceUrl` and `$dir` to match your paths:

```powershell
# Define paths
$sourceUrl = "C:\path\to\your\original_image.png"
$dir = "C:\path\to\your\project\icons"
$dest192 = "$dir\icon-192.png"
$dest512 = "$dir\icon-512.png"

# Delete old icons if they exist
Remove-Item -Force $dest192, $dest512 -ErrorAction SilentlyContinue

# Load native Windows drawing library
Add-Type -AssemblyName System.Drawing
$src = [System.Drawing.Image]::FromFile($sourceUrl)

# Generate 192x192
$b192 = New-Object System.Drawing.Bitmap(192, 192)
$g192 = [System.Drawing.Graphics]::FromImage($b192)
$g192.InterpolationMode = [System.Drawing.Drawing2D.InterpolationMode]::HighQualityBicubic
$g192.DrawImage($src, 0, 0, 192, 192)
$b192.Save($dest192, [System.Drawing.Imaging.ImageFormat]::Png)
$g192.Dispose(); $b192.Dispose()

# Generate 512x512
$b512 = New-Object System.Drawing.Bitmap(512, 512)
$g512 = [System.Drawing.Graphics]::FromImage($b512)
$g512.InterpolationMode = [System.Drawing.Drawing2D.InterpolationMode]::HighQualityBicubic
$g512.DrawImage($src, 0, 0, 512, 512)
$b512.Save($dest512, [System.Drawing.Imaging.ImageFormat]::Png)
$g512.Dispose(); $b512.Dispose()

$src.Dispose()
Write-Host "Icons generated successfully!"
```

---

## Part 2 — Enabling True Fullscreen (Window Controls Overlay)

By default, PWAs install with `"display": "standalone"`, which still shows an ugly thick browser title bar at the top on Windows desktop. 

To merge your app's background with the system control buttons (close/minimize), update `manifest.json`:

```json
{
  "name": "Your App",
  "display": "fullscreen",
  "display_override": ["window-controls-overlay", "minimal-ui"],
  "theme_color": "#your_app_background_hex",
  "icons": [
    {
      "src": "/icons/icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```

> **Note on `theme_color`:** This color is absolutely vital when using `window-controls-overlay` because Windows will use that exact hex code to paint the background behind the Minimize/Close buttons. It should perfectly match your app's CSS background color.

---

## Part 3 — The "Hard Reset" Reinstallation 

When you change `manifest.json` configurations or update core app icons, simply refreshing your app will **not** trigger the OS to update its taskbar icon or window frame. You must forcefully reinstall it:

1. Open the currently installed PWA.
2. Click the `...` menu in the top bar.
3. Select **Uninstall [App Name]**.
4. Open the app URL (e.g. `http://localhost:3000`) in Chrome/Edge.
5. Press `Ctrl + F5` to bypass the cache.
6. Click the **Install App (Monitor with downward arrow)** icon in the address bar to reinstall.

The app will instantly launch with the new icon and the updated fullscreen UI configuration!
