# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Classic Tetris implemented in vanilla JavaScript with HTML5 Canvas and CSS. No dependencies, no build process, no package.json. README (in Spanish) is the primary reference and stays in sync with `game.js` — update it when game mechanics or tunable constants change.

## Running

No install/build step. Open `index.html` directly, or serve statically:

```bash
npx serve .
# or
python3 -m http.server 8000
```

There are no tests, linter, or bundler configured.

## Architecture

Three files, no modules/bundler — `game.js` is loaded as a plain `<script>` and relies on global scope:

- **index.html** — DOM structure: `#board` canvas (300×600, the 10×20 grid at `BLOCK=30`px/cell), `#next-canvas` for the next-piece preview, HUD spans (`#score`, `#lines`, `#level`), and the `#overlay` used for both Pause and Game Over.
- **style.css** — dark/retro arcade visual theme only.
- **game.js** — all game logic, organized around a small set of global `let` state variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, ...) mutated in place by the functions below rather than passed around.

Key mechanics in `game.js`:

- **Board model**: `ROWS × COLS` matrix; each cell is `0` (empty) or a piece color index `1–7`.
- **Pieces**: square matrices in `PIECES`; `rotateCW` rotates via transpose + row-reverse.
- **Collision** (`collide`): bounds + occupied-cell check against `board`.
- **Wall kicks** (`tryRotate`): after rotating, tries x-offsets `[0, -1, 1, -2, 2]` until one doesn't collide.
- **Game loop** (`loop`): driven by `requestAnimationFrame`, accumulates elapsed time in `dropAccum` and advances the piece once it exceeds `dropInterval`.
- **Line clearing** (`clearLines`): scans bottom-up, splices full rows out and unshifts empty rows at top.
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` × current `level`; hard drop adds 2 pts/cell dropped, soft drop 1 pt/row.
- **Leveling**: `level` increases every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)` ms.
- **Ghost piece** (`ghostY`): projects the current piece straight down to its landing row, drawn at `globalAlpha = 0.2`.

Flow: `init()` builds the board, seeds `next`, calls `spawn()` (which promotes `next` to `current` and generates a new `next`, ending the game via `endGame()` if the new piece immediately collides), then starts the `requestAnimationFrame` loop. Input is handled by a single `keydown` listener (arrows, `X` to rotate, `Space` for hard drop, `P` to pause).

Tunable constants live at the top of `game.js` (`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval`). Changing `COLS`/`ROWS`/`BLOCK` requires updating the `#board` canvas `width`/`height` in `index.html` to match (`COLS × BLOCK` by `ROWS × BLOCK`).
