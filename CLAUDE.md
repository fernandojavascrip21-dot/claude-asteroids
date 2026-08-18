# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Clone of the classic arcade game **Asteroids**, implemented in pure HTML5 Canvas with vanilla JavaScript. No build step, no bundler, no dependencies, no package.json — the whole game is `game.js` loaded directly by `index.html`.

## Running the game

Open `index.html` directly in a browser, or serve it locally:

```bash
npx serve .
```

Then visit `http://localhost:3000`. There is no build, lint, or test tooling in this repo — changes to `game.js` are verified by reloading the page in a browser.

## Architecture

Everything lives in `game.js` (~420 lines), structured as a single self-contained script run top-to-bottom (no modules):

- **Input**: a global `keys` / `justPressed` map populated by `keydown`/`keyup` listeners; `pressed(code)` consumes a one-shot press (used for firing and restarting).
- **Entity classes**: `Bullet`, `Asteroid`, `Ship`, `Particle` — each owns its own `update(dt)` and `draw()`. Asteroids have 3 sizes (`RADII`/`SPEEDS`/`POINTS` indexed 1–3); splitting an asteroid (`Asteroid.split()`) spawns two of the next size down.
- **Global mutable game state**: `ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state` (`'playing' | 'dead' | 'gameover'`) are module-level `let` bindings, reset via `initGame()` / `nextLevel()`.
- **Game loop**: `requestAnimationFrame(loop)` computes `dt` (clamped to 50ms) and calls `update(dt)` then `draw()` each frame — a standard fixed-responsibility update/draw split, not an ECS.
- **Collision & scoring**: brute-force O(n·m) distance checks in `update()` — bullets vs. asteroids, ship vs. asteroids. Screen wrap (toroidal space) is handled by the `wrap(v, max)` helper used in every entity's position update.
- **Rendering**: all drawing is manual Canvas 2D path/stroke calls (no sprites/images); the canvas is a fixed 800×600 (`W`, `H` constants).

When adding features (new entity types, power-ups, etc.), follow the existing pattern: a class with `update(dt)`/`draw()`, pushed into the relevant global array, filtered out via a `dead` flag each frame.

Note: `README.md` currently mentions power-ups and a "shooting star" asteroid type that have since been removed from the code — don't treat the README as authoritative on current features.
