# Game Specs

This is the source of truth for what games exist and what each game does. When adding a game, first write or update its spec here, then implement it in `games/<id>.html` and register it in `index.html`.

Level scaling should follow the convention in `CLAUDE.md`:
- L1 — single digit
- L2 — two-digit up to 30, no carry/borrow
- L3 — two-digit up to 30, with carry/borrow
- L4 — three-digit, carry/borrow
- L5+ — larger numbers, mixed operations, twists

Not every game needs every level; skip levels that don't make sense for the skill.

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
  - L3 — A, B ∈ [10, 30], carry allowed
  - L4 — A, B ∈ [100, 999], carry allowed
  - L5 — three-addend sum A + B + C with A, B, C ∈ [10, 99]; no carry in either column

Feedback:
  - Correct → green border + light-green fill on the chosen button, score +1, auto-advance after ~650ms.
  - Wrong → red border + light-red fill + 300ms shake on the chosen button. Score unchanged. Question stays, retry allowed.

Notes:
  - Distractors: 3 wrong options at offsets in [−5, +5] excluding 0; discard negatives; guaranteed 4 distinct non-negative choices via fallback filler.
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
  - Game over shows an overlay with final score and a "Play again" button.

---
## sentence-draw

* Status: planned
* Title: Math Snake
* Emoji: Unknown
* Accent Color: forest green

This game provides a grid of numbers that the user can draw axis aligned lines across to create sequences. The top of play are provides a template for the math equation to be created, using blank spaces, operations, and the equals sign. So for example, the top might show __ + __ = __ meaning the player draws a line connecting three numbers where the first two must sum to the last. If the line drawn is valid, the numbers are deleted from the grid and numbers above fall down to take their space with new numbers appearing from above. Play continues until no match can be found.

---

## subtraction-basic

- Status: planned
- Title: Subtraction
- Emoji: ➖
- Accent color: `#4a7cff` (blue)
- Skill: Whole-number subtraction, mirroring addition-basic levels.

Question format:
  `A − B` shown centered in large digits. Always A ≥ B (no negative answers).

Answer input:
  4 multiple-choice buttons.

Levels:
  - L1 — A, B ∈ [0, 9], A ≥ B
  - L2 — A ∈ [10, 30], no borrow (ones digit of A ≥ ones digit of B)
  - L3 — A ∈ [10, 30], borrow allowed
  - L4 — A ∈ [100, 999], borrow allowed
  - L5 — missing-minuend variant: shown as `? − B = C`, solve for `?`

Feedback: same conventions as addition-basic.

Notes:
  - Distractor generation same as addition-basic but must never be negative.

---

## times-tables

- Status: planned
- Title: Times Tables
- Emoji: ✖️
- Accent color: `#7b5cff` (purple)
- Skill: Single-digit multiplication facts, then extending.

Question format:
  `A × B` shown centered.

Answer input:
  4 multiple-choice buttons.

Levels:
  - L1 — one factor is 2, 5, or 10; other ∈ [0, 10] (easy tables)
  - L2 — both factors ∈ [0, 10] (full times tables)
  - L3 — one factor ∈ [11, 12], other ∈ [0, 10]
  - L4 — one factor is a two-digit number ≤ 30, other ∈ [0, 9]

Feedback: same conventions as addition-basic.

Notes:
  - Distractors should favor common wrong-table confusions (e.g., off-by-one row/column) plus one random distractor.
  - Optional: after N in a row correct on L1, celebrate a "table cleared" moment.

---

## missing-number

- Status: planned
- Title: Missing Number
- Emoji: ❓
- Accent color: `#26a69a` (teal)
- Skill: Algebraic thinking — find the unknown in a simple equation.

Question format:
  Equation with one box replaced by a `?`, e.g., `3 + ? = 10` or `? − 4 = 5`.

Answer input:
  4 multiple-choice buttons.

Levels:
  - L1 — addition with single-digit terms, unknown in either position (a + ? = c or ? + b = c)
  - L2 — subtraction with single-digit terms, unknown in either position
  - L3 — mix of addition and subtraction, values up to 30
  - L4 — introduce multiplication: `? × B = C` with L1-2 times-table facts

Feedback: same conventions as addition-basic.

Notes:
  - Randomize which position holds the unknown so the child can't shortcut with pattern-matching.

---

## greater-less

- Status: planned
- Title: Greater or Less
- Emoji: ⚖️
- Accent color: `#e91e63` (pink)
- Skill: Comparing quantities and understanding `<`, `>`, `=`.

Question format:
  Two numbers side by side with a blank comparator in the middle, e.g., `47 ⬚ 74`.

Answer input:
  Three big tap targets: `<`, `=`, `>`.

Levels:
  - L1 — both numbers single-digit
  - L2 — both numbers ≤ 30
  - L3 — both numbers ≤ 999
  - L4 — introduce simple expressions on each side (e.g., `3 + 4 ⬚ 8`)

Feedback: same conventions as addition-basic.

Notes:
  - Ensure `=` cases appear often enough to matter (~1 in 4) so it isn't a trap answer.
