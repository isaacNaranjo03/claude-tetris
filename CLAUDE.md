# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A vanilla-JS Tetris clone. No build step, no dependencies, no `package.json`, no tests, no linter. Three files do everything: `index.html`, `style.css`, `game.js`.

UI text shown to the player is in Spanish (overlay messages, control labels, README). Keep new player-facing strings in Spanish; code identifiers and comments are English.

## Running

Open `index.html` directly, or serve statically (`python3 -m http.server 8000`, `npx serve .`). A server is only needed if browser `file://` restrictions get in the way — there is no reason it should here.

## Architecture (`game.js`)

Single-file, module-scoped mutable state — no classes. All game state lives in the top-level `let board, current, next, score, ...` declaration; `init()` (re)initializes it and is also the restart handler.

- **Board model**: `board` is a `ROWS × COLS` array of ints. `0` = empty; `1–7` index into both `COLORS` and `PIECES` (the piece type IS the color index).
- **Pieces**: `PIECES[type]` is a square matrix; a live piece is `{ type, shape, x, y }` where `shape` is a deep copy that gets mutated in place on rotation. `rotateCW` = transpose + reverse rows.
- **`collide(shape, x, y)`** is the one spatial predicate — every move/rotate/drop tests a candidate position through it before committing.
- **Game loop**: `loop(ts)` is a `requestAnimationFrame` loop accumulating `dropAccum` until it exceeds `dropInterval`, then drops one row or locks. `animId` holds the frame handle; pause/game-over/`init` all `cancelAnimationFrame(animId)`.
- **Lock sequence**: `lockPiece()` → `merge()` (stamp shape into `board`) → `clearLines()` → `spawn()`. `spawn()` promotes `next` to `current`, rolls a new `next`, and calls `endGame()` if the fresh piece already collides.
- **Scoring/level**: `clearLines()` owns line count, score (`LINE_SCORES[n] * level`), level (`floor(lines/10)+1`), and recomputes `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Rendering**: `draw()` clears and repaints every frame — grid, locked board, ghost (`ghostY()` + alpha 0.2), then current piece. `drawNext()` paints the separate `next-canvas` and is called only on spawn, not per frame.

Input is a single `keydown` listener at the bottom of the file; `updateHUD()` is called after it and after any scoring change.

## Gotchas

- `COLS`, `ROWS`, `BLOCK` in `game.js` must stay in sync with the `<canvas id="board">` `width`/`height` in `index.html` (`width = COLS*BLOCK`, `height = ROWS*BLOCK`). Same for `next-canvas` vs. the `NB`/`4`-cell grid in `drawNext()`.
- No hold piece, no 7-bag randomizer (pure `Math.random`), no lock delay, no line-clear animation — don't assume standard-Tetris features exist.
