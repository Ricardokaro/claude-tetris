# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A classic Tetris implementation in vanilla JavaScript (ES6+), HTML5 Canvas, and CSS. No dependencies, no build step, no package.json — just three files: `index.html`, `style.css`, `game.js`.

## Running the game

Open `index.html` directly in a browser, or serve it with any static server, e.g.:

```bash
python3 -m http.server 8000
npx serve .
```

There is no build, lint, or test tooling in this repo — changes to `game.js`/`index.html`/`style.css` are effective immediately on reload.

## Architecture

All game logic lives in `game.js` (single file, no modules). Key pieces:

- **Board model**: `board` is a `ROWS × COLS` matrix where each cell is `0` (empty) or an integer 1–8 indexing into `COLORS`/identifying which tetromino occupies it.
- **Pieces**: `PIECES` defines each tetromino (I, O, T, S, Z, J, L, and an 8th custom "Nut" piece with a hollow center) as a square matrix of color indices. `randomPiece()` picks uniformly among all 8.
- **Rotation**: `rotateCW(shape)` transposes + reverses rows to rotate 90° clockwise. `tryRotate()` applies this and attempts wall kicks (`[0, -1, 1, -2, 2]` column offsets) until a non-colliding position is found.
- **Collision**: `collide(shape, ox, oy)` checks board bounds and overlap with locked cells at a given offset; used for movement, rotation, and ghost-piece projection.
- **Game loop**: `loop(ts)` runs via `requestAnimationFrame`, accumulates elapsed time in `dropAccum`, and advances the piece one row once `dropAccum >= dropInterval`. On collision below, it calls `lockPiece()` (merge → clear lines → spawn next).
- **Scoring/leveling**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`. Level increases every 10 cleared lines; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms. Hard drop awards 2 pts/row dropped, soft drop 1 pt/row.
- **Ghost piece**: `ghostY()` projects the current piece straight down to its landing row; drawn at `globalAlpha = 0.2`.
- **Theme**: light/dark mode toggled via a checkbox, persisted to `localStorage` under `tetris-theme`, applied by toggling a `light-theme` class on `<body>` (see `style.css` for the corresponding CSS variables).
- **Rendering**: two canvases — `#board` (main play field, 300×600) and `#next-canvas` (next-piece preview, 120×120), both drawn with the same `drawBlock()` helper.

### Adding/tuning pieces

Adding a new piece means: append its shape to `PIECES`, add a matching entry to `COLORS`, and bump the random range in `randomPiece()` (`Math.floor(Math.random() * N) + 1`, where `N` = piece count).

### Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK` control board dimensions/cell size — if changed, update the `#board` canvas `width`/`height` in `index.html` to match (`COLS × BLOCK`, `ROWS × BLOCK`).
