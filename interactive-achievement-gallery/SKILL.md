---
name: interactive-achievement-gallery
description: Scaffold and build an expanding Trophy Room gallery where users can view unlocked achievements. Features high-fidelity 3D VanillaTilt physics, blurred frosted-glass locked states, and click-to-spin animations.
---

# Interactive 3D Achievement Gallery

When building gamified applications, users need a dedicated, large-format space to view their unlocked achievements/badges. A "Trophy Room" maximizes the visual impact of premium assets by rendering them larger and enabling complex 3D physics.

This skill provides the architectural pattern for creating an `interactive-achievement-gallery`, complete with locked/unlocked visual states, XP thresholds, and 3D interactions.

## 1. UI Skeleton

Create a dedicated container independent of the main dashboard to give the assets space to breathe. Group them into distinct grids if supporting multiple achievement categories (e.g., Ranks, Streaks, Challenges).

```html
<div class="screen" id="screen-badges">
    <div class="card" style="border:none; background:transparent;">
        <div class="card-title">🎖️ Rank Showcase</div>
        <div class="trophy-grid" id="trophies-grid">
            <!-- Badges injected via JS -->
        </div>
    </div>
</div>
```

## 2. CSS Styling & Physics Containers

The secret to a great Trophy Room is how it handles the **locked** states versus the **unlocked** states. 
- **Locked**: Heavily desaturated, frosted-glass blur, and a padlock overlay. Not interactable.
- **Unlocked**: Vibrant colors, drop shadows, hover effects, and full 3D functionality.

```css
.trophy-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
    gap: 20px;
}

.trophy-card {
    background: var(--bg3);
    border-radius: 16px;
    padding: 24px 16px;
    display: flex;
    flex-direction: column;
    align-items: center;
    border: 1px solid var(--border);
    position: relative;
    transition: transform 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
    cursor: pointer;
    overflow: hidden;
}

/* Base Badge Image Container */
.trophy-badge {
    width: 120px;
    height: 120px;
    position: relative;
    margin-bottom: 20px;
    z-index: 2;
}

.trophy-badge img {
    width: 100%;
    height: 100%;
    object-fit: contain;
    /* Optional: filter: drop-shadow(...) for SVG/transparent PNGs */
}

/* === LOCKED STATE === */
.trophy-card.locked {
    opacity: 0.5;
    filter: grayscale(100%) brightness(0.6);
    cursor: not-allowed;
}

.trophy-card.locked .trophy-badge {
    filter: blur(10px); /* Frosted Glass Effect */
}

/* Ominous Padlock Overlay */
.trophy-lock-overlay {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    font-size: 32px;
    background: rgba(0,0,0,0.6);
    border-radius: 50%;
    width: 60px;
    height: 60px;
    display: flex;
    align-items: center;
    justify-content: center;
    backdrop-filter: blur(5px);
    z-index: 10;
    display: none;
}

.trophy-card.locked .trophy-lock-overlay {
    display: flex;
}

/* === UNLOCKED STATE === */
.trophy-card.unlocked {
    box-shadow: 0 0 40px -10px var(--glow-color);
    border-color: var(--glow-color);
}
.trophy-card.unlocked:hover {
    transform: translateY(-5px);
}
```

## 3. High-Speed Click Flip Animation
To add extreme realism and interactivity, attach a 360-degree Y-axis spin animation to the container. *Note: You must spin the outer container, not the inner `<img>` tags, or VanillaTilt's inline transforms will overwrite the animation.*

```css
@keyframes coin-flip {
    0% { transform: perspective(800px) rotateY(0deg); }
    100% { transform: perspective(800px) rotateY(360deg); }
}
.coin-flip-anim {
    animation: coin-flip 0.8s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
}
```

**JavaScript Click Listener:**
```javascript
document.addEventListener('click', e => {
    // Only spin unlocked cards!
    const badge = e.target.closest('.trophy-card.unlocked');
    if (badge) {
        // Target the inner badge container to spin alongside the VanillaTilt image depth
        const iconContainer = badge.querySelector('.trophy-badge');
        if (iconContainer && !iconContainer.classList.contains('coin-flip-anim')) {
            iconContainer.classList.add('coin-flip-anim');
            iconContainer.style.transformStyle = 'preserve-3d'; 
            
            // Remove class after animation concludes so it can be spun again
            setTimeout(() => iconContainer.classList.remove('coin-flip-anim'), 850);
        }
    }
});
```

## 4. Render Logic & Physics Injection
It is critical to manually initialize `VanillaTilt` via Javascript specifically against the dynamically injected `.unlocked` nodes.

```javascript
function renderTrophyRoom() {
    const grid = document.getElementById('trophies-grid');
    if (!grid) return;
    
    // Map your achievements (Array of objects containing requirements, names, and SVG/img assets)
    grid.innerHTML = ranks.map(r => {
        const unlocked = userTotalXP >= r.min_xp; 
        
        // Pass dynamic --glow-color via style tag to ensure correct bounding box drop-shadow color
        return `<div class="trophy-card ${unlocked ? 'unlocked' : 'locked'}" style="${unlocked ? '--glow-color: '+r.color+'33;' : ''}">
            <div class="trophy-badge">${r.badge_image_html}</div>
            <div class="trophy-lock-overlay">🔒</div>
            
            <div class="trophy-name" style="color: ${unlocked ? r.color : '#888'}">${unlocked ? r.name : '???'}</div>
            <div class="trophy-req">${r.min_xp} XP to unlock</div>
        </div>`;
    }).join('');
    
    // Apply 3D physics natively ONLY to unlocked trophies with intense perspective scaling
    if (typeof VanillaTilt !== 'undefined') {
        VanillaTilt.init(document.querySelectorAll("#trophies-grid .unlocked .trophy-badge img"), {
            max: 35,                 // Higher max angle than dashboard
            speed: 400,
            glare: true,
            "max-glare": 0.5,        // Strong geometric shine
            scale: 1.25,             // Extreme scaling pop
            perspective: 800         // Deep field of view
        });
    }
}
```

## Conclusion

This implementation guarantees a stunning, high-performance UI layout where users are visually rewarded for unlocking achievements, while locked achievements exist as frosted-glass silhouettes that naturally drive gamification loops.
