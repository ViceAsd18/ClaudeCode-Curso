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

The README mentions power-ups and a "shooting star" asteroid type; these are not implemented in `game.js`.
