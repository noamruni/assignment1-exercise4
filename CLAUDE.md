# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A 3D rolling ball obstacle course game built with Three.js. Single HTML file with no build process — all game logic, rendering, and UI are contained in `index.html`.

## Running the Game

Open `index.html` directly in a browser:
```bash
# Simple HTTP server (Python)
python3 -m http.server 5500

# Or use Live Server extension in VS Code
```

Navigate to `http://localhost:5500/index.html`

For debug mode with automated tests:
```
http://localhost:5500/index.html?debug=1
```

## Architecture

### Single-File Structure
All code lives in `index.html` with three sections:
1. **HTML/UI**: Tailwind-styled overlays (start screen, game over, HUD)
2. **CSS**: Minimal — mostly handled by Tailwind CDN
3. **JavaScript**: Game engine, physics, Three.js scene management

### Core Systems

**Scene Graph** (Three.js)
- `scene`: root container
- `camera`: follows ball with lerp smoothing
- `ball`: player-controlled sphere with physics
- `platforms[]`: collidable surfaces with Box3 bounds
- `obstacles[]`: hazards (static cylinders, moving boxes)
- `collectibles[]`: rotating octahedrons worth 100 points

**Game Loop**
- `animate()`: requestAnimationFrame driver
- `update(delta)`: physics, input, collision, camera follow
- `clock.getDelta()`: capped at 0.1s to prevent physics explosions

**Physics (custom, not a library)**
- Gravity: -30 units/s²
- Velocity integration with friction (0.98 decay)
- Ground detection: ray-free Box3 intersection checks
- Jump: only when `onGround` flag set by platform collision

**State Management**
```js
gameStarted: false → true (hides start screen)
gameOver: false → true (shows game over screen)
lives: 3 → 0 (respawn vs game over)
score: 0 → ∞ (collectibles +100, future: height/speed bonuses)
```

### Entity Lifecycle

**Platforms**: created in `createCourse()` → never destroyed
- Stored with `{mesh, box}` for collision
- Types: normal, gap pairs, ramps, goal

**Obstacles**: created in `createCourse()` → updated in `update()`
- Static: cylinder sentries
- Moving: boxes with `startX`, `endX`, `direction`, `speed`

**Collectibles**: created in `createCourse()` → marked `collected: true`
- Bob animation: `sin(time)` on Y axis
- Not removed from scene, just hidden (`visible = false`)

## Feature Roadmap

The README contains a 14-step feature list that must be implemented in order. Key phases:

1. **Core (steps 1-3)**: Game flow, scoring, leaderboard
2. **Platform mechanics (4-6)**: Labels, falling, moving
3. **Combat (7-8)**: Enemy spawning, AI, shooting
4. **Powerups (9)**: Star invincibility, spring jumps
5. **Polish (10-12)**: Assets, audio, mobile
6. **Advanced (13-14)**: MediaPipe camera control, custom features

Each feature should be committed separately as `feat: <description>`.

## Testing Strategy

In-browser debug module triggered by `?debug=1` URL param. Logs test results to console:

**Key tests**:
- Spawn ratios (enemies 5%, types 50/40/10%, springs 4%, moving 6%)
- Platform overlap detection (Box3 intersection checks)
- Falling platform timer (standTime > 4s)
- Leaderboard localStorage persistence
- Powerup invincibility mechanics
- Mobile touch handler registration

See README "Automated Testing Strategy" section for full implementation sketch.

## Constraints & Design Decisions

**No build step**: keep dependencies via CDN only (Three.js r128, Tailwind browser mode)

**Single file**: avoid splitting until complexity demands it (not yet)

**Fixed course**: platforms/obstacles generated once in `createCourse()`, not procedural

**Lives system**: 3 lives with respawn-on-hit, game over at 0

**Win condition**: `ball.position.z < -205` (reaching goal platform at z=-210)

## Common Patterns

**Adding a new obstacle type**:
1. Create mesh in `createObstacle*(x, y, z)` function
2. Push to `obstacles[]` with `{mesh, box, type, ...customFields}`
3. Add update logic in `update()` loop
4. Handle collision in existing `obstacles.forEach()` block

**Adding a new collectible**:
1. Create mesh in `createCollectible*(x, y, z)`
2. Push to `collectibles[]` with `{mesh, collected, ...customFields}`
3. Add animation/collection logic in existing `collectibles.forEach()`

**Extending UI**:
- Add div to HTML with Tailwind classes
- Reference via `getElementById()` in JS
- Toggle visibility with `classList.add/remove('hidden')` and `style.display`
