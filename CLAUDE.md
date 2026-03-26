# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

```bash
python3 -m http.server 8080
# Open http://localhost:8080
```

There is no build step, bundler, or dependencies. The entire game is a single self-contained file: `index.html`.

## Architecture

Everything lives in `index.html` — HTML, CSS, and JS in one file. No frameworks, no external assets.

### Game state machine

All mutable state is in a single `state` object. The core turn loop is:

```
picking → (player picks a number) → collecting → (opponent selects factors + clicks Commit) → picking → …
```

- `state.phase`: `'picking'` | `'collecting'`
- `state.currentPlayer`: index (0 or 1) of the player whose turn it is to **pick**
- `state.collector`: index of the player currently collecting factors
- `state.busy`: true while the computer is "thinking" (blocks user input)

### Key functions

| Function | Role |
|---|---|
| `handlePick(n)` | Validates and processes a number pick; switches phase to `collecting` |
| `handleCollectClick(n)` | Toggles a factor in/out of `state.selectedFactors` |
| `commitCollecting()` | Scores selected factors, fires celebration, advances turn |
| `advanceTurn(nextPicker)` | Sets next picker, checks for game over, triggers computer AI |
| `showResult()` | Renders the end/level-up overlay and plays jingles |
| `launchGame()` | Resets state and starts a new game from current settings |
| `goHome()` | Returns to setup screen, restarts home music |

### Computer AI

`computeAIPick()` — greedy: maximizes `n − factorSum` for the picked number. In Quest mode, uses `questLevel / 10` probability of optimal play (rest is random from valid picks). In PvC mode, always optimal.

A number is only a valid pick if at least one of its proper factors is still available.

### Audio system (Web Audio API)

- `startHomeMusic()` — schedules a 12-second looping chiptune (square wave melody + triangle bass) and re-schedules itself via `setTimeout` just before the loop ends
- `stopHomeMusic()` — clears the timer and stops all active oscillator nodes
- `playJingle(notes)` — one-shot fanfare (win or level-up)
- AudioContext is created lazily on first `pointerdown` to satisfy browser autoplay policy

### Pixel art sprites

Sprites are defined as arrays of character rows with a palette map. `makeSpriteHTML(key, px)` converts them to inline SVG `<rect>` elements. `px` is the pixel size (5 = game avatars, 7 = home scene avatars).

### Screens

Three mutually exclusive views toggled via the `hidden` attribute:
- `#home-scene` + `#setup-screen` — home / lobby
- `#game-screen` — active game
- `#result-overlay` — end-of-game / level-up modal (overlays game screen)

### Quest mode

10 levels defined in `QUEST_LEVELS`. Grid grows from 4×4 to 10×10. AI difficulty scales from 10% optimal (level 1) to 100% optimal (level 10). Player must beat the computer to advance; ties count as a loss.
