# 🧠 Skills Collection

A growing library of reusable AI agent skills for self-hosted apps, DevOps, and development workflows.
Each skill folder contains a `SKILL.md` with step-by-step instructions, gotchas, and command references.

---

## 📦 Available Skills

### 🐳 [`docker-deployment-troubleshooting`](./docker-deployment-troubleshooting/SKILL.md)
**Debug and fix Docker containerized applications.**
- Container won't start or keeps crashing
- ERR_CONNECTION_REFUSED / port issues
- Volume mounting & database permission errors
- Build failures and dependency problems
- Full diagnostic checklist + command cheat sheet

---

### 🚀 [`setup-nodejs-pwa`](./setup-nodejs-pwa/SKILL.md)
**Set up a self-hosted Node.js app and convert it into an installable PWA.**
- Create `package.json` from scratch
- Install Node.js on Windows via `winget`
- Fix PowerShell execution policy blocking npm
- Install npm dependencies with PATH workarounds
- Add `manifest.json`, service worker, and icons
- Inject PWA tags into `index.html`
- Verify browser install prompt (Chrome/Edge)

---

## 🗂️ Structure

```
.agents/
└── skills/
    ├── README.md                              ← You are here
    ├── docker-deployment-troubleshooting/
    │   ├── SKILL.md
    │   └── DEPLOYMENT_HANDOVER.md
    └── setup-nodejs-pwa/
        └── SKILL.md
```

---

## ➕ Adding a New Skill

1. Create a folder: `.agents/skills/<skill-name>/`
2. Add a `SKILL.md` with YAML frontmatter:
   ```
   ---
   name: skill-name
   description: One-line description of when to use this skill
   ---
   ```
3. Write clear steps, commands, gotchas, and verification
4. Add it to this README index

---

## 💡 Skill Ideas (Future)

| Skill | Description |
|-------|-------------|
| `windows-autostart` | Auto-launch apps on Windows boot (Task Scheduler / startup folder) |
| `local-network-share` | Expose local apps to other devices on home network |
| `sqlite-backup` | Automate SQLite database backups on Windows |
| `nginx-reverse-proxy` | Set up nginx to serve multiple local apps on one port |
