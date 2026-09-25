# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Classic Tetris implemented in vanilla JavaScript with HTML5 Canvas and CSS. No dependencies, no framework, no build process — just three files: `index.html`, `style.css`, `game.js`.

## Running the game

There is no build/lint/test tooling (no `package.json`). To run:

```bash
# Open directly
start index.html          # Windows

# Or serve locally (recommended, e.g. for consistent module/asset loading)
python3 -m http.server 8000
npx serve .
```

Then open the page (or `http://localhost:8000`) in a browser. To verify a change works, open the game in a browser and actually play it (movement, rotation, line clears, level-up, pause, game over/restart) — there are no automated tests.

## Architecture

Everything lives in `game.js` (~300 lines), structured around a `requestAnimationFrame` game loop with plain global state (no classes, no modules):

- **Board model**: `board` is a `ROWS × COLS` (20×10) matrix; each cell is `0` (empty) or a color index `1–7` identifying which piece locked there.
- **Pieces**: `PIECES` defines the 7 tetrominoes as square matrices (indices 1–7 map to `COLORS`). Rotation is computed on the fly via `rotateCW` (transpose + reverse rows), not pre-baked rotation states.
- **Collision** (`collide`): checks a shape against board bounds and already-locked cells.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` columns until a non-colliding position is found, else the rotation is discarded.
- **Game loop** (`loop`): accumulates elapsed time (`dropAccum`) each animation frame; when it exceeds `dropInterval`, the piece drops one row or locks if it can't.
- **Locking a piece** (`lockPiece`): `merge()` writes the piece into `board` → `clearLines()` removes full rows (scored via `LINE_SCORES` × `level`) → `spawn()` promotes `next` to `current` and generates a new `next`; if the new piece immediately collides, `endGame()` fires.
- **Scoring/leveling**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 pts/row dropped, soft drop 1 pt/row. Level increments every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)` ms.
- **Rendering** (`draw`): clears and redraws the grid, locked board cells, a semi-transparent ghost piece (`ghostY()` projects straight down), then the active piece. `drawNext()` renders the next-piece preview on a separate canvas (`#next-canvas`).
- **Input**: a single `keydown` listener drives arrow keys (move/soft-drop), `ArrowUp`/`X` (rotate), `Space` (hard drop, `preventDefault`ed), and `P` (pause toggle via `togglePause`).

`index.html` just declares the DOM (`#board` canvas 300×600, `#next-canvas` 120×120, HUD spans for score/lines/level, and the pause/game-over `#overlay`) and loads `game.js`. `style.css` provides the dark/retro-arcade look (flex layout, monospace HUD, backdrop-blur overlay).

### Tunable constants (in `game.js`)

`COLS`, `ROWS`, `BLOCK` (cell size), `COLORS`, `LINE_SCORES`, `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, also update the `#board` canvas `width`/`height` in `index.html` to match (`COLS × BLOCK` by `ROWS × BLOCK`).
