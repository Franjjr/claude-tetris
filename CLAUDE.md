# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Classic Tetris implemented in vanilla JavaScript with HTML5 Canvas and CSS. No dependencies, no build step, no package.json, no test suite.

## Running the game

```bash
open index.html                 # macOS, direct file open
python3 -m http.server 8000     # or: npx serve .  /  php -S localhost:8000
```

There is no build, lint, or test command — verify changes by opening `index.html` (or a local server URL) in a browser and playing.

## Architecture

Three files, no modules: `index.html` loads `style.css` and `game.js` directly, and `game.js` is a single top-level script that runs immediately (`init()` is called at the end of the file, with no `DOMContentLoaded` guard).

All game state lives in module-level `let` variables in `game.js` (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) — there is no state container or class; functions mutate these globals directly.

Key pieces of `game.js`:
- **Board model**: `ROWS × COLS` matrix, each cell is `0` (empty) or an index 1–7 into `COLORS`/`PIECES` identifying the piece that placed it.
- **Pieces**: the 7 tetrominoes as square matrices in `PIECES`. Rotation (`rotateCW`) is a transpose, no piece-specific rotation tables.
- **Collision** (`collide`): bounds + overlap check against `board`, used for movement, rotation, and ghost-piece projection.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` and keeps the first that doesn't collide.
- **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates delta time in `dropAccum` and drops the piece when it exceeds `dropInterval`.
- **Locking/clearing**: `lockPiece` → `merge` (bake piece into `board`) → `clearLines` (bottom-up full-row removal, updates score/lines/level) → `spawn` (promote `next` to `current`, generate new `next`; if the new piece collides immediately, `endGame()` fires).
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 pts/row dropped, soft drop adds 1 pt/row.
- **Level/speed**: level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
- **Rendering**: `draw()` clears and redraws grid, locked board, ghost piece (`ghostY()`, alpha 0.2), and the current piece each frame; `drawNext()` renders the preview canvas separately.
- Input is handled by a single `keydown` listener (arrows + `X` to rotate, `Space` for hard drop, `P` to pause); the restart button rebinds by simply calling `init()` again.

## Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, initial `dropInterval`. If `COLS`, `ROWS`, or `BLOCK` change, update the `<canvas id="board">` `width`/`height` in `index.html` to match (`COLS × BLOCK`, `ROWS × BLOCK`).
