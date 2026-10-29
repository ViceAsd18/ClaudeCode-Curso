# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids clone in plain HTML5 Canvas. No dependencies, bundler, linter, or tests. README and in-game text are in Spanish; keep UI strings and comments in Spanish.

## Run

Open `index.html` in a browser, or `npx serve .` and visit `http://localhost:3000`.

## Architecture

All logic lives in `game.js`, loaded by `index.html` via a plain `<script>` (no modules). The canvas is fixed at 800x600 (`W`/`H` constants; `index.html` canvas size must match).

- **Entities**: classes `Ship`, `Bullet`, `Asteroid`, `Particle`. Each has `update(dt)` and `draw()`; removal is via a `dead` flag, and the main update filters arrays (`bullets`, `asteroids`, `particles`) afterwards. Entities draw themselves directly on the global `ctx`.
- **World is toroidal**: positions use `wrap(v, max)`. Particles do not wrap.
- **Game state**: module-level globals (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`) plus `state` = `'playing' | 'dead' | 'gameover'`. `update(dt)` branches on `state`; `'dead'` is a 2s respawn timer (`deadTimer`) where asteroids keep moving; `'gameover'` restarts on Space via `initGame()`.
- **Input**: `keys` (held) and `justPressed` (edge-triggered, consumed by `pressed(code)`). Shooting uses `pressed('Space')`, so it fires once per keypress, not while held.
- **Loop**: `requestAnimationFrame` with `dt` in seconds, clamped to 0.05.
- **Tuning data**: asteroid size tables `RADII`, `SPEEDS`, `POINTS` are indexed by size 1-3 (index 0 unused). Splitting goes size 3 -> 2 -> 1. Collisions are circle-based (`dist`); ship-vs-asteroid uses `a.radius * 0.82`.
- Level clear (`asteroids.length === 0`) calls `nextLevel()`, which spawns `3 + level` large asteroids outside a safe radius around the center.

- **Power-up Disparo Triple**: class `PowerUp` drops once per level (`powerUpSpawned`, reset in `nextLevel()`): random (`POWERUP_CHANCE`) when a bullet destroys an asteroid, forced when 2 or fewer asteroids remain so every level gets one. Picking it up sets global `tripleShotTimer` (`TRIPLE_DURATION` = 5s); `Ship.tryShoot(triple)` then fires 3 bullets fanned by `TRIPLE_SPREAD`. Timer resets on ship death.
- **Power-up Escudo**: `PowerUp` has a `type` (`'triple'` | `'shield'`, random 50/50 on spawn; still one power-up per level). Picking up a shield sets `shieldTimer` (`SHIELD_DURATION` = 5s, reset on ship death). While active, the ship collision radius becomes `SHIELD_RADIUS`; the first asteroid hit goes through `destroyAsteroid(a)` (score + explosion + split, same as a bullet), clears the shield and gives `ship.invincible = 1`. Drawn in `Ship.draw()`, HUD line `ESCUDO Ns`.
- **Bomba Nova**: rare item, separate from the per-level power-up. A `PowerUp` of type `'nova'` (not in `POWERUP_TYPES`) drops with `NOVA_CHANCE` when a bullet kills an asteroid, at most once per level (`novaSpawned`, reset in `nextLevel()`) and never while one is stored. Picking it up sets `hasNova` (max 1, kept across levels and deaths). Key `B` calls `detonateNova()`: destroys all asteroids with no split, adds points, sets `novaFlash` (full-screen flash), which usually triggers `nextLevel()`. HUD `NOVA [B]` top right.

The README mentions a "shooting star" asteroid type; it is not implemented in `game.js`.
