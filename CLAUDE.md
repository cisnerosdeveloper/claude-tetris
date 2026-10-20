# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Tetris in vanilla JavaScript + HTML5 Canvas. No `package.json`, bundler, linter or tests — there is nothing to build or install. The README and UI text are in Spanish; keep new user-facing strings in Spanish.

## Running

Open `index.html` directly, or serve statically (e.g. `python -m http.server 8000`) and visit `http://localhost:8000`.

## Architecture

Three files, all loaded by `index.html` (`game.js` is a plain `<script>`, no modules):

- `index.html` — fixed DOM that `game.js` looks up by id at load: `board` (300×600 canvas), `next-canvas` (120×120), `score`/`lines`/`level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`. Renaming an id breaks `game.js`.
- `style.css` — dark/retro styling; the overlay is shown/hidden via the `hidden` class.
- `game.js` — all logic, using module-level mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`, ...) that `init()` resets.

Key points in `game.js`:

- Board is a `ROWS×COLS` matrix of `0` or a color/piece index 1–7; `PIECES` and `COLORS` are indexed by the same 1–7 value (index 0 is `null`). A piece is `{ type, shape, x, y }`, with `shape` a square matrix whose non-zero cells hold its color index.
- Game loop (`loop`) is `requestAnimationFrame`-driven: accumulates `dropAccum` and gravity-drops one row when it reaches `dropInterval`. Pause/game-over cancel the frame via `animId`; unpausing restarts `loop` manually and resets `lastTime`.
- Piece lifecycle: `lockPiece()` = `merge()` → `clearLines()` → `spawn()`. `spawn()` promotes `next` to `current` and calls `endGame()` if it collides immediately. `clearLines()` also updates level and `dropInterval` (`max(100, 1000 − (level−1)·90)`).
- Rotation is `rotateCW` plus horizontal-only kicks `[0, -1, 1, -2, 2]` in `tryRotate`; a failed rotation is silently dropped.
- Scoring: `LINE_SCORES × level`, +1 per soft-drop row, +2 per hard-drop row.
- `collide()` allows `ny < 0` (cells above the board) without checking the board.
- If changing `COLS`, `ROWS` or `BLOCK`, update the `width`/`height` attributes of `<canvas id="board">` in `index.html` to `COLS×BLOCK` × `ROWS×BLOCK`. The next-piece preview uses its own hardcoded 4×4 grid at 30px.
