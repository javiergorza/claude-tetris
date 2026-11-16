# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Tetris-style falling-block game in vanilla JavaScript + HTML5 Canvas + CSS. No dependencies, no `package.json`, no build step, no tests, no linter. UI text and README are in Spanish — keep user-facing strings in Spanish.

## Running

Open `index.html` directly in a browser, or serve the folder statically (preferred):

```bash
python -m http.server 8000   # then open http://localhost:8000
npx serve .
```

Verification is manual: reload the page and play.

## Architecture

Three files: `index.html` (DOM: `#board` canvas, side panel with `#score`/`#lines`/`#level`, `#next-canvas` preview, `#overlay` for pause/game over), `style.css` (dark theme), and `game.js` (all logic, single script, module-level globals, `'use strict'`, no modules).

Key concepts in `game.js`:

- **Board**: `ROWS × COLS` matrix; each cell is `0` (empty) or a piece type index `1–7`. The same index selects the color in `COLORS` and the shape in `PIECES` (index 0 is `null` in both). Piece shape matrices store the type index as their cell value, so `merge()` copies values straight onto the board.
- **Piece object**: `{ type, shape, x, y }`. Rotation (`rotateCW`) builds a new matrix; `tryRotate` applies wall kicks by trying x offsets `[0, -1, 1, -2, 2]`.
- **Collision** (`collide(shape, ox, oy)`) is the single source of truth for movement, rotation, ghost projection (`ghostY`), and spawn/game-over detection. Cells with `y < 0` are allowed (above the board).
- **Game loop** (`loop`): `requestAnimationFrame`, accumulates `dt` into `dropAccum`, steps the piece down when it reaches `dropInterval`. `animId` holds the current frame handle; pause and game over cancel it, `init()` restarts everything (also wired to the restart button).
- **Lock sequence**: `lockPiece()` → `merge()` → `clearLines()` → `spawn()`. `spawn()` moves `next` into `current`, generates a new `next`, and calls `endGame()` if the new piece collides immediately.
- **Scoring/level**: `LINE_SCORES[cleared] * level`; soft drop +1 per row, hard drop +2 per cell. Level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 − (level − 1) × 90)`.
- **Rendering**: `draw()` redraws grid, board, ghost (alpha 0.2), and current piece every frame. `drawNext()` only runs on spawn. `drawBlock` is shared between both canvases (size passed in).

## Gotchas

- Changing `COLS`, `ROWS`, or `BLOCK` requires updating `width`/`height` of `<canvas id="board">` in `index.html` to `COLS×BLOCK` × `ROWS×BLOCK`.
- The next-piece preview assumes a 4×4 grid of 30px cells (`#next-canvas` is 120×120).
- Adding a piece type requires entries in both `COLORS` and `PIECES` and updating the `* 7` in `randomPiece()`.
