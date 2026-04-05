---
name: threat-intel-aggregator
description: Build a self-hosted Node.js RSS Aggregator that automatically scrapes external industry feeds on a cron schedule, saves incident reports to a local SQLite database, and serves an interactive "Mission Control" UI to view the global alerts.
---

# Threat Intelligence RSS Aggregator

A core capability of highly advanced homelabs is the ability to sever reliance on centralized, algorithmically-driven news sources by aggregating raw tactical RSS feeds directly from the source. 

When building a "Morning Briefing" or "Threat Intel" dashboard, use this architectural blueprint.

## The Architecture Stack
- **Backend Framework**: Express.js
- **Ingestion Engine**: `rss-parser` (running on an internal `setInterval` ping loop).
- **Storage**: SQLite DB utilizing `INSERT OR IGNORE` to elegantly prevent duplicate incident parsing based on the article's unique `guid`.
- **Frontend**: Vanilla HTML/CSS with dark mode, high-contrast monospace (Fira Code) fonts, and optimistic UI updates (hiding a row instantly before the server confirms).

## Implementation Steps

### 1. Database Initialization
Use SQLite to create a table combining standard RSS fields:
```sql
CREATE TABLE IF NOT EXISTS articles (
    guid TEXT PRIMARY KEY,
    title TEXT,
    link TEXT,
    snippet TEXT,
    source TEXT,
    pubDate INTEGER,
    status TEXT DEFAULT 'unread'
)
```

### 2. The Parse Loop
The node server must contain an array of RSS XML urls (like `CISA` or `BleepingComputer`).
```javascript
const Parser = require('rss-parser');
const parser = new Parser();

async function fetchIntel() {
    for (const feed of FEEDS) {
        const parsed = await parser.parseURL(feed.url);
        // Loop over parsed.items, insert into SQLite filtering out existing guids
    }
}
setInterval(fetchIntel, 60 * 60 * 1000); // 1 Hour Sweep
```

### 3. The UI Aesthetic
The frontend should utilize:
1. Hard lines and monospace fonts (e.g., `font-family: 'Fira Code', monospace;`).
2. Cyber accents (`:root { --green: #00ff66; --red: #ff3333; }`).
3. Triage buttons: An `[Acknowledge]` button that fires an API request to `PUT /api/articles/:guid/status` setting it to 'read' so it no longer appears in the `GET /api/articles` feed.

### 4. Dockerization
Wrap the app in an Alpine Node container mapping the local `/data` directory outwards. This ensures the SQLite brain containing read/unread statuses isn't wiped out when you pull a new image of the application.
