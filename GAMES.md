# Game Specs

This is the source of truth for what games exist and what each game does. When adding a game, first write or update its spec here, then implement it in `games/<id>.html` and register it in `index.html`.

Level scaling should follow the convention in `CLAUDE.md`:
- L1 — single digit
- L2 — two-digit up to 30, no carry/borrow
- L3 — two-digit up to 30, with carry/borrow
- L4 — three-digit, carry/borrow
- L5+ — larger numbers, mixed operations, twists

Not every game needs every level; skip levels that don't make sense for the skill.

**Scoring convention:** every game awards `LEVEL_POINTS[level]` per correct answer, where `LEVEL_POINTS = [1, 3, 7, 13, 20]`. Games with fewer than 5 levels use the leading portion. Wrong-answer penalties (where they exist) stay at −1.

---

## Spec template

Copy this block to add a new game. Delete any lines that don't apply.

```
## <game-id>

- Status: planned | in progress | built | needs update
- Title:
- Emoji:
- Accent color:
- Skill:

Question format:
  How the prompt is displayed.

Answer input:
  Multiple choice (N options) | numeric keypad | drag-and-drop | tap-to-select.

Levels:
  - L1 — …
  - L2 — …
  - L3 — …

Feedback:
  - Correct → …
  - Wrong → …

Notes:
  - Anything unusual: distractor rules, edge cases, sounds, timing.
```

---

# Games

## addition-basic

- Status: built
- Title: Addition
- Emoji: ➕
- Accent color: `#ff9b3d` (orange)
- Skill: Whole-number addition, scaling from single digit to three-digit with carry.

Question format:
  Operands joined by ` + `, centered. Font size scales with viewport width so multi-digit / multi-operand prompts stay on one line.

Answer input:
  4 multiple-choice buttons in a 2×2 grid, tap to answer.

Levels:
  - L1 — A, B ∈ [0, 9]
  - L2 — A, B ∈ [10, 30], no carry (ones digits sum to ≤ 9)
  - L3 — A, B ∈ [10, 30], always carries (ones digits sum to ≥ 10)
  - L4 — A, B ∈ [100, 999], carry allowed
  - L5 — three-addend sum A + B + C with A, B, C ∈ [10, 99]; no carry in either column

Feedback:
  - Correct → green border + light-green fill on the chosen button, score +1, auto-advance after ~650ms.
  - Wrong → red border + light-red fill + 300ms shake on the chosen button. Score unchanged. Question stays, retry allowed.

Notes:
  - Distractors: at least one option always shares the correct answer's ones digit (typically `answer − 10`, the classic "forgot to carry" mistake — falls back to `+10` / `±20` if `−10` is negative) so the child can't shortcut by matching only the ones column. Remaining slots use offsets in [−5, +5] excluding 0; discard negatives; fallback filler guarantees 4 distinct non-negative choices.
  - Level persists in `localStorage` key `mathgames.addition-basic.level`.
  - Answer choices shuffled every question.

---
## shooter
* Status: built
* Title: Math Shooter
* Emoji: 🚀
* Accent Color: purple (`#c084fc`)

This game is designed off of the classic space invaders game where the invaders are numbers. Numbers should come down in a grid from the top centered in the gameplay area. Along the bottom is a space ship icon. Tapping in the lower portion of the gameplay area moves the ship to the horizontal component of the touch location and shoots an animated projectile up and impacts a number. The numbers are possible answers to a basic math equation that's at the very top of the gameplay area, above where the numbers start to come down. After each shot has impacted or gone off the top of the play area, the numbers 'advance' in classic space invaders fashion, sliding to one side until they reach the edge before moving down one row then switching the slide direction. Once they reach the bottom, the game is over. The score is based on successful hits vs misses or wrong answers. Wrong answers subtract a point, correct answers add a point, misses do nothing. The equation changes after each shot, and should always be answerable with a shot (i.e. be one of the bottom most numbers in each column).

Implementation notes:
  - Grid: 6 columns × 4 rows. Ship is a CSS triangle (not an emoji, to avoid cross-platform rotation differences).
  - Tap zone: tap anywhere in the playfield to fire (more forgiving than a strict lower-portion rule). Ship still slides to the tap x.
  - Wave clear: if the player shoots down every tile, the grid silently refills and play continues.
  - Levels:
    - L1 — single-digit addition; tile values 0–18
    - L2 — mixed single-digit +/−; tile values 0–18
    - L3 — mixed +/− with terms up to 30 (may involve carry/borrow); tile values 0–40
  - Equation generation is target-first: pick a random bottom-most tile, then generate a level-appropriate equation whose answer equals it. Guarantees the invariant that the current equation is always answerable.
  - **Wrong shots ricochet.** After a wrong tile flashes red and fades, a red projectile launches from that tile's position back down toward the ship's current X and impacts it. The ship shakes and flashes red for ~½s. Score is still just −1; the ship is not destroyed (a kid game shouldn't punish a single miss too harshly, but the visual makes the "wrong answer = you get hit" cause-and-effect concrete).
  - Game over shows an overlay with final score and a "Play again" button.

---
## sentence-draw

* Status: built
* Title: Math Snake
* Emoji: 🐍
* Accent Color: forest green (`#2d7a3d`)

This game provides a grid of numbers that the user can draw axis aligned lines across to create sequences. The top of play are provides a template for the math equation to be created, using blank spaces, operations, and the equals sign. So for example, the top might show __ + __ = __ meaning the player draws a line connecting three numbers where the first two must sum to the last. If the line drawn is valid, the numbers are deleted from the grid and numbers above fall down to take their space with new numbers appearing from above. Play continues until no match can be found.

Implementation notes:
  - Grid: 7×7. Cells positioned absolutely so they can animate falls with CSS transitions.
  - Line interpretation: **snake path** — pointerdown on any cell starts a path, drag onto an orthogonally adjacent cell to extend, drag back onto the previous cell to shrink, drag onto a non-adjacent cell does nothing. The path can turn corners. Caps at the template length. Release evaluates.
  - Line reads in either direction — dragging in either order counts if the numbers satisfy the equation.
  - Template display fills in numbers as the player drags (`? + ? = ?` → `5 + ? = ?` → `5 + 3 = ?` → `5 + 3 = 8`), so the child sees the equation build in real time.
  - **Powerup cells** (rare): on each new cell spawn, there's a 3% chance the cell is a 3× powerup (pink) and a 7% chance it's a 2× powerup (yellow). If a matched path passes through any powerup cells, the match's points are the product of their multipliers (e.g., 2× and 3× in the same match → 6 points). A floating `+N` pops up at the last cell of the match. The multiplier is conveyed by cell color only — a small legend in the topbar shows what each color means (no per-cell `×2`/`×3` badge, to keep the number itself readable).
  - **HUD layout**: score, equation template, and remaining-match count share a single row above the grid: `Score 5   [ ? + ? = ? ]   12 left`. This makes the score visible right next to the equation as it changes, and the "left" count updates after every collapse so the child can see how many matches remain.
  - Collapse: on a match, matched cells fade out; remaining cells fall to the bottom of their columns; new random cells spawn above the grid and fall in. New cells re-roll powerup status.
  - Game-over detection: DFS over connected paths of the template length across the grid — if none satisfy the equation (in either direction), show the overlay.
  - New-game grid guaranteed to contain at least one match (retries generation up to 40 times).
  - Levels:
    - L1 — `? + ? = ?`, cell values 0–9
    - L2 — `? − ? = ?`, cell values 0–9

Follow-ups worth considering:
  - L3+ (three-addend `? + ? + ? = ?`, larger numbers, or multiplication)
  - Legend/tutorial for powerup colors (currently color-only — kid learns by seeing score jump)

---

## clock-read

* Status: built
* Title: Read the Clock
* Emoji: ⏰
* Accent Color: teal (`#26c6a4`)

The opposite of `time-challenge`: a clock is displayed at a random time and the player has to read it and enter the time. The time is entered through three tap-to-open popups — one for the hour, one for the tens digit of the minute, and one for the ones digit — so the child breaks the problem into three simpler reads instead of typing a full number.

Levels:
  - L1 — 15 minute granularity
  - L2 — 5 minute granularity
  - L3 — 1 minute granularity

Implementation notes:
  - Reuses the same SVG clock rendering as `time-challenge` (viewBox −100..100, 60 tick marks with major every 5, numbers 1..12, fractional hour-hand position). The clock is read-only — no pointer handlers.
  - Selector row: three tiles reading `Hour : Tens Ones`. Each shows the current selection or `?`. Tapping a tile opens a modal popup with a grid of number buttons:
    - Hour → 3×4 grid, values 1..12
    - Tens → 3×2 grid, values 0..5 (max 59 minutes)
    - Ones → 5×2 grid, values 0..9
  - Popup closes on: tap on a number, tap on the backdrop, or the Cancel button.
  - Check button is disabled until all three slots are filled. Combined minute = `tens * 10 + ones`.
  - Feedback: correct → green face flash + score +1 + auto-advance after 800ms. Wrong → red face + shake, no score change, retry allowed.
  - Level persists in `localStorage` key `mathgames.clock-read.level`.

---

## time-challenge
* Status: built
* Title: Time Challenge
* Emoji: 🕐
* Accent Color: light blue (`#4ec3f0`)

This presents a standard 12 hour time for the player to set the corresponding analog representation. There should be a clock where clicking on the inner area moves the small hand corresponding to the hour, and clicking the outer area sets the minute hand. Then there's a button to check the current clock setting as the answer.

Levels:
  - L1 — 15 minute granularity
  - L2 — 5 minute granularity
  - L3 — 1 minute granularity

Implementation notes:
  - Analog clock rendered as inline SVG (viewBox −100..100). 12 hour numbers around an inner ring; 60 tick marks (major every 5) around the outer rim.
  - Click/drag zone: tap or drag inside radius 68 sets the hour (snaps to nearest 1..12). Outside radius 68 sets the minute (always snaps to 1-minute precision — the level's granularity controls the target time, not the input, so the player can always dial in the exact answer). A subtle dashed inner ring hints at the boundary. The minute band is intentionally the larger of the two zones because it needs the finer targeting.
  - Drag is supported — pointerdown captures the zone and pointermove keeps updating it. The zone is locked at pointerdown so accidentally crossing the boundary mid-drag doesn't hijack the other hand.
  - Hour hand renders at the fractional position (`hour + minute/60`) so, e.g., 3:30 shows the hour hand halfway between 3 and 4, matching a real clock. This is a core teaching moment — the kid learns to read analog times, not just place hands.
  - Check button validates selectedHour + selectedMinute against the target exactly.
  - Feedback: correct → green face flash + score +1 + auto-advance after 800ms. Wrong → red face + shake, no score change, retry allowed.
  - Level persists in `localStorage` key `mathgames.time-challenge.level`.



---

## subtraction-basic

- Status: built
- Title: Subtraction
- Emoji: ➖
- Accent color: `#4a7cff` (blue)
- Skill: Whole-number subtraction, mirroring addition-basic levels.

Question format:
  L1–L4: vertical stacked layout (minuend on top, `− subtrahend` below, horizontal rule), right-justified with `font-variant-numeric: tabular-nums` so columns align. Always A ≥ B (no negative answers).
  L5: inline `? − B = C` with the `?` shown in accent color.

Answer input:
  4 multiple-choice buttons in a 2×2 grid.

Levels:
  - L1 — A, B ∈ [0, 9], A ≥ B
  - L2 — A ∈ [10, 30], no borrow (ones digit of A ≥ ones digit of B)
  - L3 — A ∈ [10, 30], always borrows (ones digit of A < ones digit of B)
  - L4 — A ∈ [100, 999], borrow allowed
  - L5 — missing-minuend variant: shown as `? − B = C` with B, C ∈ [1, 20]; solve for `?`

Feedback: same conventions as addition-basic.

Notes:
  - Distractors: at least one option shares the correct answer's ones digit (typically `answer + 10`, the classic "forgot to borrow" mistake — falls back to `−10` / `±20` if not viable). Remaining slots use offsets in [−5, +5] excluding 0; discard negatives; fallback filler guarantees 4 distinct non-negative choices.
  - Level persists in `localStorage` key `mathgames.subtraction-basic.level`.

---

## times-tables

- Status: built
- Title: Times Tables
- Emoji: ✖️
- Accent color: `#7b5cff` (purple)
- Skill: Single-digit multiplication facts, then extending.

Question format:
  `A × B` shown centered inline. The `×` is rendered in accent color to visually anchor the operator.

Answer input:
  4 multiple-choice buttons in a 2×2 grid.

Levels:
  - L1 — one factor is 2, 5, or 10; other ∈ [1, 10] (easy tables). Factor order randomized.
  - L2 — both factors ∈ [1, 10] (full times tables).
  - L3 — one factor ∈ [11, 12], other ∈ [1, 10]. Factor order randomized.
  - L4 — one factor ∈ [10, 30], other ∈ [1, 9]. Factor order randomized.

**`× 0` is intentionally excluded** at every level. It's a single memorized rule ("anything × 0 = 0"), not a fact worth drilling with distractors, and it degenerates the off-by-one row/column distractors — `(a±1) × 0` both collapse to `0`, leaving only two useful candidates instead of four, and the kid can reasonably think "any of these numbers × 0 = 0" and be confused when only the `0` button is accepted.

Feedback: same conventions as addition-basic.

Notes:
  - Distractors favor common wrong-table confusions: `(a±1) * b` and `a * (b±1)` (off-by-one row/column reads from the times table). Remaining slots filled with random offsets scaled to the answer's magnitude; discard negatives; fallback filler guarantees 4 distinct non-negative choices.
  - Level persists in `localStorage` key `mathgames.times-tables.level`.

---

## missing-number

- Status: built
- Title: Missing Number
- Emoji: ❓
- Accent color: `#26a69a` (teal)
- Skill: Algebraic thinking — find the unknown in a simple equation.

Question format:
  Inline equation with the unknown shown as a `?` in accent color with an underline (blank-to-fill look), e.g., `3 + ? = 10` or `? − 4 = 5`. Operators and equals sign are also accent-colored to visually separate them from the numbers.

Answer input:
  4 multiple-choice buttons in a 2×2 grid.

Levels:
  - L1 — addition, single-digit terms (0..9), unknown in position `a` or `b` (never the result).
  - L2 — subtraction, single-digit terms (0..9), unknown in position `a` or `b`; A ≥ B guaranteed.
  - L3 — mix of addition and subtraction with terms up to 30.
  - L4 — multiplication `? × B = C` (or occasionally `A × ? = C`) with both factors in [0, 10] so it only draws on L1-L2 times-table facts.

Feedback: same conventions as addition-basic.

Notes:
  - Which operand is unknown is randomized every question so the child can't shortcut with position-based pattern-matching.
  - Distractors follow the addition-basic rule: at least one option shares the ones digit (typically `answer − 10`), rest are ±5 offsets, non-negative, filler fallback for 4 distinct choices.
  - Level persists in `localStorage` key `mathgames.missing-number.level`.

---

