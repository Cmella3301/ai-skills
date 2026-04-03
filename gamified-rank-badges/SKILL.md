---
name: Gamified Rank Badges
description: Systematically generate and inject escalating 3D metallic rank progression ladders into a web UI using AI tools and CSS blending.
---

# Gamified Rank Badges

This skill provides a systematic approach for transforming basic, flat gamification interfaces into highly polished, premium "gaming-inspired" UI elements using generated graphic assets and CSS blending.

## When to use
Use this when you are building any application (e.g. habit tracker, dashboard, or game) that rewards users over time and the USER asks for a distinct, progressively cooler rank/badge system.

## 1. Asset Generation
Instead of using basic SVG icons, use an AI visual generator tool to create a series of assets.
1. Decide on a progression hierarchy (e.g. Bronze -> Silver -> Gold -> Diamond -> ... -> Final Boss/Champion).
2. Generate isolated images on a **pure black background** (e.g., `#000000`) rather than transparent. This ensures that intricate neons, glows, and blooms are rendered at the absolute highest fidelity.
3. Save them to the application's static assets directory (e.g. `/icons/ranks/` or `/public/ranks/`).

## 2. Dynamic Rendering
Store your progression milestones in a configuration array containing XP thresholds and raw HTML badge strings. 

Example approach:
```javascript
const progression = [
  { rank: 'Bronze', minXP: 0, badgeHtml: '<img src="/ranks/1_bronze.png" class="badge-gfx" />' },
  { rank: 'Silver', minXP: 1000, badgeHtml: '<img src="/ranks/2_silver.png" class="badge-gfx" />' },
  { rank: 'Gold', minXP: 3000, badgeHtml: '<img src="/ranks/3_gold.png" class="badge-gfx" />' }
];

// Logic to extract current rank based on totalXP
```

## 3. The "Mix-Blend" Engine
Because AI generation tools often struggle to output perfectly transparent alpha-channel PNGs with complex glowing neon details, generate them on pure black backgrounds. 

When embedding them in the UI, apply `mix-blend-mode: screen;` on the `<img>` tag snippet.
This elegantly strips the dark background, allowing intricate glowing highlights to integrate seamlessly onto dark DOM application backgrounds!

**Warning:** This relies entirely on the application having a dark mode or a dark UI container to work visually.

```css
.badge-gfx {
    width: 100%;
    height: 100%;
    object-fit: contain;
    mix-blend-mode: screen;
    filter: drop-shadow(0 0 10px rgba(255, 200, 0, 0.4)); /* Apply dynamic rank colors to shadow for extra bloom */
}
```

## 4. UI/UX Polishing
* Ensure the image is absolutely centered in its parent node.
* Add a very subtle CSS floating animation (`transform: translateY()`) to the badge container to emulate a video game artifact hovering.
