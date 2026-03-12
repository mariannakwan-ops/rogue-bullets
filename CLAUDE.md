# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a browser-based retro game project. All games are single self-contained HTML files — no build system, no dependencies, no bundler. Open any `.html` file directly in a browser to run it.

## Running the Games

```bash
open shooter.html       # macOS
open tictactoe.html
```

## Git & GitHub Workflow

**Commit and push to GitHub after every meaningful unit of work** — feature additions, bug fixes, sound/visual changes, etc. Never leave work uncommitted. This ensures we can always revert to a working state.

```bash
git add <file>
git commit -m "descriptive message"
git push
```

Commit message rules:
- First line: short imperative summary (e.g. `Add shield power-up`, `Fix enemy spawn off-screen bug`)
- Be specific — mention what changed and why if non-obvious
- Always include the co-author trailer: `Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>`

Remote: `https://github.com/mariannakwan-ops/rogue-bullets`

## shooter.html Architecture

The entire game is ~1000 lines of vanilla JS inside a single `<script>` tag. The sections in order:

1. **Constants** — speeds, radii, enemy stats (`ENEMY_STATS`), level configs (`LEVELS`)
2. **Audio (`sfx`)** — self-contained IIFE using Web Audio API; all sounds are procedurally synthesized. `sfx.shoot()`, `sfx.explosion(large)`, `sfx.levelComplete()`, etc.
3. **Input** — single global `input` object mutated by event listeners (`keys`, `mx`, `my`, `down`, `clicked`)
4. **Entities** — `Particle`, `Bullet`, `ScorePopup`, `Player`, `Enemy` classes, each with `update(dt)` and `draw(ctx)` methods
5. **Game state** — single `gameState` object; `gameState.current` drives the state machine: `'start'` → `'playing'` → `'levelComplete'` → `'playing'` → `'gameOver'`
6. **Systems** — `handleCollisions()`, `spawnDeathParticles()`, `startGame()`, `startLevel()`, `nextLevel()`
7. **Draw functions** — `drawBackground()`, `drawHUD()`, `drawCrosshair()`, `drawStartScreen()`, `drawLevelComplete()`, `drawGameOver()`
8. **Game loop** — fixed-timestep accumulator at 60 ticks/sec via `requestAnimationFrame`

### Key patterns

- All entity drawing uses `ctx.save()` / `ctx.translate()` / `ctx.rotate()` / `ctx.restore()` — sprites are drawn in local space (origin = entity center, right = forward)
- Enemy types (`grunt`, `fast`, `tank`, `shooter`) share the `Enemy` class; behavior differs via `ENEMY_STATS` and type checks in `update()` and `draw()`
- `LEVELS` array defines the first 5 levels; levels beyond index 5 are generated procedurally (increasing count, all types)
- High score is persisted via `localStorage` key `rogue_hs`
- The `sfx` AudioContext is created lazily on first user interaction to satisfy browser autoplay policy
