# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project

Asteroids clone in plain HTML5 Canvas + vanilla JS. No dependencies, no bundler, no build step, no tests, no linter. The README and in-game text are in Spanish; keep UI strings and comments consistent with that.

## Running

Open `index.html` directly in a browser, or serve locally with `npx serve .` (http://localhost:3000).

## Architecture

All game logic lives in `game.js` (loaded by `index.html` via a plain `<script>`; the canvas is fixed at 800x600 and `W`/`H` constants in `game.js` must match the canvas attributes in `index.html`).

- **Loop**: `requestAnimationFrame` → `update(dt)` then `draw()`. `dt` is in seconds and clamped to 0.05 to avoid big jumps after tab switches. All speeds/timers are in px/s or seconds.
- **State machine**: global `state` is `'playing' | 'dead' | 'gameover'`, driven by `update()`. `'dead'` runs a 2s `deadTimer` (world keeps moving, ship hidden) before `ship.reset()`; `'gameover'` waits for Space to call `initGame()`. Game state is module-level globals (`ship, bullets, asteroids, particles, score, lives, level`), not encapsulated.
- **Entities** (`Ship`, `Asteroid`, `Bullet`, `Particle`): each has `update(dt)`, `draw()`, and a `dead` flag; dead entities are removed with `.filter()` after each update pass. The world is toroidal — positions use `wrap()` (except particles, which don't wrap).
- **Asteroids**: size 1–3, with per-size lookup arrays `RADII`, `SPEEDS`, `POINTS` indexed by size (index 0 unused). `split()` returns two asteroids of `size - 1`. Smaller asteroids score more.
- **Collisions**: simple circle checks via `dist()`. Bullet vs asteroid uses `a.radius`; ship vs asteroid uses `ship.radius + a.radius * 0.82` and is skipped while `ship.invincible > 0` (3s after any `reset()`, shown as blinking).
- **Input**: `keys[code]` for held keys; `pressed(code)` consumes a one-shot "just pressed" edge (used for Space to shoot/restart, so holding Space doesn't auto-fire).
- **Levels**: when `asteroids` is empty, `nextLevel()` resets the ship and spawns `3 + level` large asteroids outside a 130px safe radius around screen center.

Power-ups and the shooting-star asteroid were removed in earlier commits; the README description still mentions them.
