# Math Games

Simple browser-based math games for elementary school practice. Each game is a single, self-contained HTML file that runs on phones and tablets — no build step, no server required.

## Play

Open `index.html` in a browser (or host it via GitHub Pages) to see the game menu. Tap a tile to launch a game. Individual game files in `games/` can also be opened directly.

## Structure

- `index.html` — hub / launcher, lists all games
- `games/` — one self-contained HTML file per game
- `CLAUDE.md` — conventions and guide for AI-assisted development

## Adding a game

1. Copy an existing file in `games/` as a template.
2. Register it in the `GAMES` array in `index.html`.
3. Test in a mobile-sized browser viewport.

See `CLAUDE.md` for the full conventions.
