# AI Development Guide

This project is a collection of simple browser-based math games for an elementary school student. Games run on phones and tablets (primary target: portrait tablet) and are distributed as static HTML.

## Hard rules

- **Self-contained files.** Each game is one `.html` file with inline CSS and JS. No external dependencies, no build step, no frameworks. This lets a single game file be shared standalone (AirDrop, email, USB) and still work.
- **No network at runtime.** No CDNs, no Google Fonts, no analytics. Must work fully offline once the file is opened.
- **Touch-first.** Assume a tablet or phone. Interactive elements should be at least 60px tall. Never depend on hover.
- **Gentle feedback.** Wrong answers get a soft shake and a color hint — never scary sounds or big red X's. Correct answers get positive reinforcement (color flash, score bump).
- **Short sessions.** Games should be playable in 2–5 minute bursts. No lives system, no forced progression, no game-over screens.

## Difficulty scaling

Games should scale in difficulty whenever the math allows. Convention:

- Level 1 — single digit
- Level 2 — two-digit up to 30, no carry/borrow
- Level 3 — two-digit up to 30, with carry/borrow
- Level 4 — three digit, carry/borrow
- Level 5+ — larger  numbers, mixed operations, or added twists

Show a level picker at the top of the game so the child can choose freely. Persist the last-used level in `localStorage` under the key `mathgames.<game-id>.level`.

## Adding a new game

`GAMES.md` at the repo root is the source of truth for what each game does. Every game should have a spec entry there.

1. **Spec first.** If there is no entry for the game in `GAMES.md`, write one using the template at the top of that file. If there is one, re-read it — the spec wins over any assumption from the game name.
2. Create `games/<id>.html`. Use `games/addition-basic.html` as the starting template — copy it and rework the question logic to match the spec.
3. Add an entry to the `GAMES` array in `index.html` with `id`, `emoji`, `title`, `desc`, and `file`.
4. Test at 768x1024 (iPad portrait) and 375x667 (phone portrait). Play through every level, including intentionally wrong answers.
5. Update the spec entry's `Status` in `GAMES.md` to `built`.

## Design conventions

- Big, high-contrast text. Question font-size ≥ 4rem on tablets.
- One accent color per game (helps kids remember which game is which). Set it in the `:root` block.
- Every game has a "← Games" back link in the top-left, linking to `../index.html`.
- Use the system font stack — no custom fonts.
- No sound by default. If added later, include a persistent mute toggle.
- Prefer multiple-choice (4 options) for younger levels. A numeric keypad works for advanced games where students should compute the answer directly.
- **HUD width matches the play area.** If the play area is capped (e.g., `max-width: 500px`), constrain the topbar, level picker, and any status rows to the same width and center them (`margin-left: auto; margin-right: auto`). Otherwise on wide screens the score/back/legend/etc. drift to the viewport edges while the play area sits centered in the middle — visually disconnected. Everything HUD-related should sit next to the play area, not the window edges.

## Testing

There is no test framework. To verify a change:
1. Open `index.html` in a browser.
2. Use browser devtools to switch to a tablet viewport (iPad or similar).
3. Play through each difficulty level, including intentionally wrong answers.
4. Open the game file directly (`file://…/games/<slug>.html`) to confirm it also works standalone.
