# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids clone in plain HTML5 Canvas + vanilla ES6. No dependencies, no bundler, no build step, no tests, no linter. All game logic lives in `game.js`; `index.html` only hosts an 800×600 `<canvas id="canvas">` and loads the script. Deployed as a static site on GitHub Pages (demo link in README).

## Running

Open `index.html` directly in a browser, or serve the folder:

```bash
npx serve .   # then visit http://localhost:3000
```

## Architecture (`game.js`)

- **Main loop**: `requestAnimationFrame(loop)` computes `dt` in seconds (clamped to 0.05 s) and calls `update(dt)` then `draw()`. All motion is time-based: velocities are px/s, timers count down in seconds.
- **State machine**: global `state` is `'playing' | 'dead' | 'gameover'`. `update()` branches on it: `'dead'` keeps asteroids/particles moving while `deadTimer` counts down to respawn; `'gameover'` waits for Space to call `initGame()`.
- **Entities**: `Ship`, `Bullet`, `Asteroid`, `Particle` classes, each with `update(dt)` and `draw()`. Removal uses a `dead` flag, then arrays are rebuilt with `.filter(x => !x.dead)`. Module-level arrays `bullets`, `asteroids`, `particles` hold the entities.
- **Toroidal space**: positions wrap with `wrap(v, max)` using the constants `W`/`H` (keep them in sync with the canvas size in `index.html`). Particles don't wrap.
- **Asteroid sizes**: `size` 3 (large) → 2 → 1; `RADII`, `SPEEDS`, `POINTS` are lookup arrays indexed by size (index 0 unused). `split()` returns two smaller asteroids.
- **Collisions**: circle-distance checks in `update()`. Bullet-vs-asteroid collects split children in `newAsteroids` and appends them after the loop. Ship-vs-asteroid uses `a.radius * 0.82` for leniency and is skipped while `ship.invincible > 0` (3 s after reset, drawn as blinking).
- **Levels**: when `asteroids` is empty, `nextLevel()` resets the ship and spawns `3 + level` large asteroids, at least `SAFE_DIST` from the center.
- **Input**: `keys[code]` holds held keys (rotation/thrust); `pressed(code)` is a consume-once edge trigger (shooting, restart). Shooting rate is also limited by `ship.shootCooldown`.

## Conventions

- Code comments, HUD text and the README are in Spanish (e.g. `NIVEL`, `PUNTAJE`); keep new user-facing text and comments in Spanish.
- Visual style is white vector strokes on black (`lineWidth` ~1.5, `lineJoin = 'round'`).
- Power-ups and the "shooting star" asteroid were removed (commit `13e713f`), although the README description still mentions them.
