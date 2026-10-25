# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JavaScript Tetris on HTML5 Canvas. No dependencies, no `package.json`, no build step, no linter, no test suite. User-facing text and the README are in Spanish; keep new UI strings in Spanish.

## Running

Open `index.html` directly in a browser, or serve the folder statically (recommended):

```bash
python3 -m http.server 8000   # then open http://localhost:8000
npx serve .
```

Verification is manual: play the game in the browser.

## Architecture

Three files: `index.html` (DOM: board canvas, side panel HUD, pause/game-over overlay), `style.css` (dark theme), and `game.js` (all logic, a single non-module script with `'use strict'` and module-level mutable state).

Key ideas in `game.js`:

- **One integer means both piece type and color.** `PIECES[i]` shapes are filled with the value `i`, and `COLORS[i]` is its color (index 0 is `null` = empty). Board cells store that same value once a piece is merged, so `drawBlock` renders board, ghost and active piece from the same index. Adding/reordering a piece means keeping `PIECES`, `COLORS` and the `randomPiece()` range (`1..7`) in sync.
- **Global state** (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, ...) is declared once with `let` and reset entirely in `init()`, which is also the restart handler.
- **`collide(shape, ox, oy)` is the single legality check** used for movement, rotation (`tryRotate` with kick offsets `[0,-1,1,-2,2]`), gravity, ghost projection (`ghostY`) and game-over detection in `spawn()`. Cells with `y < 0` are allowed (above the board).
- **Piece lifecycle:** `lockPiece()` → `merge()` → `clearLines()` (also updates level and `dropInterval = max(100, 1000 - (level-1)*90)`) → `spawn()` (promotes `next` to `current`, ends the game if it collides immediately).
- **Loop:** `requestAnimationFrame`-driven `loop(ts)` accumulates `dropAccum` and applies gravity when it exceeds `dropInterval`, then redraws everything each frame. Pause/game over work by `cancelAnimationFrame(animId)`; resuming resets `lastTime` before restarting the loop.
- Input is handled in one `keydown` listener, which calls `updateHUD()` after every action.

## Coupled values

- Board canvas size in `index.html` (`300×600`) must equal `COLS*BLOCK × ROWS*BLOCK`.
- `drawNext()` assumes a 4×4 preview grid with 30px cells, matching the `120×120` `#next-canvas`.
