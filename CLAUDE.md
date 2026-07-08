# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A single-player Tetris implementation in vanilla JavaScript with HTML5 Canvas — no dependencies, no build process, no `package.json`. The entire game logic lives in `game.js` (~300 lines).

## Running the game

There is no build/lint/test tooling. Just serve or open the files directly:

```bash
open index.html          # macOS, or double-click the file
python3 -m http.server 8000   # or any static server, then visit localhost:8000
```

## Architecture

Three files cooperate with no module system — `game.js` is a single script loaded directly by `index.html` and operates on globals/DOM elements queried at the top of the file:

- **`index.html`** — DOM shell: `<canvas id="board">` (300×600, the 10×20 grid at `BLOCK`=30px/cell), a side panel with score/lines/level/next-piece canvas, and a shared overlay div used for both Pause and Game Over.
- **`style.css`** — dark/retro arcade visual styling only.
- **`game.js`** — all game state and logic, structured as:
  - **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a color index 1–7 identifying which piece type locked there.
  - **Pieces**: `PIECES` are square matrices (index = `COLORS` index too). Rotation is a matrix transpose+reverse (`rotateCW`), not precomputed rotation states.
  - **Collision** (`collide`): checks piece cells against board bounds and existing locked cells.
  - **Wall kicks** (`tryRotate`): on rotation collision, retries at x offsets `[0, -1, 1, -2, 2]` before giving up.
  - **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates `dt` and drops the piece one row once `dropAccum >= dropInterval`.
  - **Line clearing** (`clearLines`): scans bottom-up, splices full rows out and unshifts empty rows in; re-checks the same index after a splice.
  - **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` × current `level`; hard drop adds 2 pts/row dropped, soft drop adds 1 pt/row.
  - **Level/speed**: level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
  - **Ghost piece** (`ghostY`): projects the current piece straight down to its landing row, drawn at `globalAlpha = 0.2`.
  - Control flow: `init()` → `spawn()` (pulls `next` into `current`, generates new `next`, checks spawn-collision for game over) → `requestAnimationFrame(loop)`. Keydown handler moves/rotates/soft-drops/hard-drops/pauses; all state mutation funnels through `collide`/`lockPiece`/`updateHUD`.

When tuning gameplay constants (`COLS`, `ROWS`, `BLOCK`), the `<canvas id="board">` width/height in `index.html` must be updated to match (`COLS × BLOCK` and `ROWS × BLOCK`) — they are not computed from the constants.
