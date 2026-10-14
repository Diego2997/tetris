# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris: `index.html` + `style.css` + `game.js`. No build, no package.json, no tests, no linter. Run by opening `index.html` or serving the directory statically (`python3 -m http.server 8000`). README and UI strings are in Spanish.

## Architecture

All logic lives in `game.js` as top-level functions sharing module-level mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`, ...). `init()` resets all of it, and is also the restart handler.

- Board is a `ROWS x COLS` matrix of `0` or a piece-type index 1-7. The same index selects `COLORS[i]` and `PIECES[i]` (index 0 is `null` in both).
- Piece shape is a square matrix filled with its own type index (not 1), so `shape[r][c]` doubles as the color index when drawing.
- Piece flow: `lockPiece()` = `merge()` -> `clearLines()` -> `spawn()`. `spawn()` calls `endGame()` when the new piece collides immediately.
- Gravity runs in `loop()` (rAF, accumulates `dropAccum` against `dropInterval`). Player-input drops go through `softDrop()`/`hardDrop()`, which also award score.
- `clearLines()` owns level and speed recalculation: `level = max(startLevel, floor(lines/10)+1)` (`startLevel` 1-10 comes from the pause menu, saved in localStorage), `dropInterval = max(100, 1000 - (level-1)*90)`.
- Rotation is `rotateCW()` plus horizontal kicks `[0,-1,1,-2,2]` in `tryRotate()`. No vertical kicks.
- Pause/game over share one `#overlay` element; `endGame()`/`togglePause()` set its title text and unhide it.

## Gotchas

- Canvas size is hardcoded in `index.html` (`#board` 300x600, `#next-canvas` 120x120). Changing `COLS`/`ROWS`/`BLOCK` in `game.js` requires updating canvas `width`/`height` to match.
- Next-piece preview uses a fixed 4x4 grid with `NB = 30` in `drawNext()`.
- Keydown handler calls `updateHUD()` after every key, but `lockPiece()`/`hardDrop()` paths rely on that or on `clearLines()` for HUD refresh; keep that in mind when adding new score sources.
