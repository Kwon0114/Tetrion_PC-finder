# Tetrion PC Finder

[한국어](README.md) · [English](README.en.md)

A browser tool that finds Tetris **perfect clears (PC)** and calculates their **probability** exactly.
It runs from a single HTML file with no server and no installation. It works on both phone and PC browsers, and the interface is available in Korean and English (`EN` at the top right).

Since this is a web app, it's extremely light
---

## Contents

1. [Quick start](#quick-start)
2. [Layout](#layout)
3. [Editing the board](#editing-the-board)
4. [Next (queue) input syntax](#next-queue-input-syntax)
5. [PC finder](#pc-finder)
6. [Options](#options)
7. [Result screen](#result-screen)
8. [Setup finder](#setup-finder)
9. [Build odds (setup build mode)](#build-odds-setup-build-mode)
10. [Score calculation](#score-calculation)
11. [Importing and exporting fumen](#importing-and-exporting-fumen)
12. [Algorithm](#algorithm)
13. [Syntax compared with PC solver](#syntax-compared-with-pc-solver)
14. [Limitations](#limitations)
15. [Credits](#credits)

---

## Quick start

1. Open the HTML file in a browser.
2. Paint the blocks already on the **board** and set the **height**.
3. Type a queue into **Next**. Example: `[IOTSZJL]!` or `SZTOJLIOSZ`. The `Random 2 bags` button fills in two 7-bags for you.
4. On the **PC finder** tab, press `Find perfect clear`.
5. Check the probability, the solution list and the solution playback in the result.

> **Offline**
> All code is in one HTML file and no external fonts or scripts are loaded, so opening the file directly works without an internet connection.
> When you host it on the web (for example GitHub Pages), put `sw.js`, `manifest.webmanifest` and `icon.svg` in the same folder as `index.html`. A service worker caches the page, so after the first visit it opens offline too, and it can be installed to the home screen. (While online, the newest version is always fetched first.)

---

## Layout

| Area | Description |
|---|---|
| Top tabs | Switch between two modes: **PC finder** and **Build odds** |
| Board | Field editing, height, fumen import/export, color palette |
| Pieces / rules | Kick and score rules (TETR.IO / Jstris), add / duplicate / delete settings |
| Next | Queue input, syntax help (`i` button), case count |
| Options | Hold, 180° rotation, spins, per-mode options |
| Result | Probability, solution list, solution playback, fumen copy |

Below the Next field, the **number of cases** is shown live. If there are too many cases, a warning says the calculation may take long.

---

## Editing the board

- Tap a cell to paint or erase a block. Dragging by touch paints several cells at once.
- Change the **height** with the `−` / `+` buttons or by typing a number. (Up to 12 rows in PC finder, up to 23 rows in Build odds mode.)
- The **color palette** assigns piece colors, and the colors are kept when a solution is applied to the board.
- **Clear** resets the board.
- You can create **several boards** and manage them by number. Tapping a board chip switches to that board, and tapping the selected chip again lets you change its number. `Duplicate` and `Delete` are supported. (Up to 20 boards.)

---

## Next (queue) input syntax

The pieces are the seven letters `I O T S Z J L`. Input is case-insensitive.

### Basics

| Input | Meaning |
|---|---|
| `SZTOJLI` | A fixed queue in exactly this order |
| `[TIL]` | **One** of T, I, L |
| `[TIL]2` | Pick 2 and keep the order written (TI, TL, IL) |
| `[TIL]!` | All 3, **shuffled** |
| `[TIL]2!` | Pick 2 of the 3 and **shuffle** them |
| `[]!` | Same as `[IOTSZJL]!`, a 7-bag |
| `[]3!` | Only the first 3 of a 7-bag |

- Spaces are allowed inside `[ ]`, and groups can be **nested**. Example: `[[SZ]! [LJ]!]2!`
- A number goes **before** `!` and `%`. Example: `[IOTSZJL]3!`
- `%` flips whether a group is a bag. On a normal group it marks it as a bag, and on `[]` it removes the bag mark.
- Typing `[` adds `]` automatically, and writing only a closing bracket as in `SZ]` wraps it into `[SZ]`. `{ }` works the same way. (See the rules in [Algorithm 13](#13-auto-input-queue-field).)

### Order conditions `{ }`

Attach right after a group to restrict the **order inside that group**.

| Input | Meaning |
|---|---|
| `{LJ<SZ}` | L and J both before S and Z (the rest is free) |
| `{SZ>LJ}` | Same meaning as above |
| `{A<B<C}` | Chain: A before B and B before C |
| `{A<B,C<D}` | Several rules separated by commas |

Example: `[szljtt]!{tt>z}` shuffles S, Z, L, J, T, T but keeps only the queues where both Ts come after Z (120 of 360 cases).

Things to note:

- A condition applies **only to the group right before it**. In `T[]!{t>j}`, the leading `T` is unrelated to the condition.
- Only the queues that satisfy the condition are kept, and the probabilities are normalized again.
- `{ }` can be attached only once per group. For several rules, join them with commas.
- To compare pieces from different groups, attach the condition to an outer group that wraps both.
- If no queue satisfies the condition, an error is shown.

### Warnings

The calculation continues, but input that may differ from what you intended is flagged with a gold `⚠` under the field.

- An order condition uses a **piece that is not in the group**: that piece is ignored.
- An order condition is used on a group that **draws only some** of its pieces: pieces that were not drawn drop out of the condition.

---

## PC finder

Given a board and a queue, it finds **every solution that ends in a perfect clear** and calculates the **success probability** over the whole queue.

- If the queue has `[ ]`, every possible case is expanded with its weight (probability). Duplicate queues are merged exactly.
- Only boards where filled cells + 4 × pieces = 10 × rows are meaningful, so impossible boards and queues that are too short are reported with the reason.
- Only the leading pieces that matter for the PC are calculated. With hold, one more piece is looked at.
- The **See** value limits the number of visible next pieces (the player's-view probability).
- The search supports up to the bottom **8 rows**. If there are blocks above row 8, a notice is shown.

---

## Options

| Option | Description |
|---|---|
| Use hold | Whether to use hold |
| Start with empty hold | On: start with an empty hold. Off: the first next piece is already in the hold |
| 180° rotation | Allow 180° rotation |
| Include spins and tucks | Include spin and tuck moves according to the kick rules |
| Probability only (fast) | Finds all solutions but skips the minimal solutions, failing queues and leftover-mino statistics |
| Solutions only (fast) | Skips the probability and focuses on finding solutions |
| Back-to-back keep chance | Calculates the chance of starting with back-to-back on and still having it after the PC (can be slower) |
| Average score | Calculates solution scores and the average with the selected mode's score table (Jstris / TETR.IO Blitz) |
| Starting combo | Number of consecutive line clears already in progress |
| SPC/DPC | Calculates with the T-spin single and similar moves of a 5-row board already counted |
| Kick / score rules | **TETR.IO** / **Jstris** presets. Add, duplicate and delete settings to keep several |
| Parallel | Shows how many workers are used, and the reason when parallel processing is not available |

---

## Result screen

- The **success probability** and the number of cases used
- **Solution list**: each solution is labeled by its `save` (the mino left in hold), and you can select one to view only that solution
- **Solution playback**: step through one move at a time with `◀ play ▶`
- Statistics of the **bag minos left after the PC** (including by the minimal solutions)
- A list of **failing queues** (highest probability first)
- **Copy fumen**: copies the solution you are viewing as a fumen code
- Results are saved while the board and options stay the same, so you can reopen them.

---

## Setup finder

Turn on **Setup finder** in `Options`, and the `Find` button looks for a **setup built with the first N pieces** instead of a PC.

- Set the number of pieces used for the setup (1–9) with the **Pieces** input.
- **Used minos**: tap each mino to cycle `must use ✓` → `must not use ✕` → off. The minos marked as required cannot exceed the number of setup pieces.
- It can be used with no height limit. (The engine limit is the same 8 rows as the PC finder.)
- Results can be folded and unfolded in **groups of 50**, and up to 300 are shown.
- Each setup has "Details" and "Use as board" (choose the field being edited, an existing board or a new board).
- It also calculates the chance of reaching a PC when the setup exists (the conditional probability).

---

## Build odds (setup build mode)

Treat the board shape as the **target setup** and calculate "the probability that this queue can build that shape".

- Each board is assigned a **setting**. A setting stores the next queue, hold and conditions.
- **Conditions**: T-spin / T-spin single, double, triple / quad / all-spin (TETR.IO only). You can specify a count.
- **Union / intersection** switch: choose whether at least one of the boards works (union) or all of them work (intersection).
- You can limit the calculation to the boards you checked (if none is checked, all are used).
- Result: total build probability, probability per board, the **queues that cannot build it** (highest probability first), and an example solution.
- Board height can be up to 23 rows.
- With back-to-back on, the probability of building while keeping back-to-back is shown as well.

---

## Score calculation

Solution scores and the average are calculated with the score table of the selected mode (**Jstris** or **TETR.IO Blitz**).

- Level is fixed at 1
- Line clear points: single 100 / double 300 / triple 500 / quad 800, mini double 400, all clear 3500, and so on
- Bonuses such as T-spin, back-to-back ×1.5, combo +50×combo, and PC +3000

The exact table for each mode is shown in the tooltip of the `Average score` option.

---

## Importing and exporting fumen

- **Import**: paste a fumen code (`v115@…`) and it is turned into **one board per page**. The terrain is split automatically.
- **Export / copy fumen**: copies the current field or a solution as a fumen.
- In Build odds mode you can import fumen up to 23 rows. In the PC finder the limit is 12 rows.

---

## Algorithm

The overall flow is as follows.

```
Parse queue syntax → queue cases (with weights)
      ↓
 scan → cull → place   : find every "piece combination (solution)" that makes a PC
      ↓
 queue check           : decide which queue can actually be played with which solution
      ↓
 probability / minimal solutions / save / failing queues / score / back-to-back statistics
```

### 1. Board representation

- A row is a **10-bit integer**. Collision checks, line clears and position tests are all bit operations (AND, shift).
- A board is packed 5 rows per number (it fits exactly in a double) into the keys `K1, K2, K3`.
- The boards of one search stage are kept in **typed arrays**. The board table (an open-addressing hash), the per-board line-clear count, alive flag, queue state (a linked-list pool) and the edges to previous boards (a linked list) take about 40 bytes per board, which saves a lot of memory compared with object/Set based storage.

### 2. Landing position generation (bit-parallel)

- Row masks are precomputed for every piece and rotation, so a collision check is a few ANDs.
- For each rotation `r` and height `y`, one mask represents **all columns at once**.
  - `valid[r][y]`: the columns where the piece fits at that height
  - `reach[r][y]`: the columns that can actually be reached
- A soft drop is one AND with the next row's `valid`, sideways movement is a **shift-AND flood**, and rotation applies the kick tests in order to all columns at the same time (columns that already found a kick are excluded).
- Above the stack the piece can move freely, so only the drop is computed there to save time.
- **Kick tables**: TETR.IO SRS+ (symmetric I kicks, per-direction 180° kicks) and Jstris guideline SRS are supported. 180° kicks can be switched on and off per test.
- **Spin detection**: a landing where the last move was a rotation and the piece cannot move left, right or up counts as a spin. A T counts even when only 3 of the 4 corners are blocked, and a T rotation that used the 5th kick (the 1×2 move) counts as a proper T-spin even if the front corners are not all filled.
- Reachable landing positions are **cached** by the key (stacked lower rows, piece, 180° setting) and the cache is kept after the run ends.

### 3. scan / cull / place (finding solutions)

All solution candidates are found first without looking at the queue, and the queue is checked against the solutions afterwards.

1. **scan**: starting from the start board, place one piece at a time and advance stage by stage. Boards are stored **after line clears**, so equal boards merge. Each board remembers "which queue states can reach it" and "which boards it came from (previous edges)".
2. **cull**: walk backwards from the final (empty) board and keep only the boards that lead to the PC.
3. **place**: scan only the remaining boards again, this time remembering **which piece was placed**. A piece placed after a line was cleared can be split by the cleared line, so it is restored as a "picture" unfolded as if no line had been cleared. Each surviving piece combination is a **solution**.

**Pruning (`illegalBoard`)** — boards that can never be filled are dropped immediately.

- A column that is completely walled off sideways can only take a vertical I, so its number of empty cells must be a multiple of 4.
- If in every row one of two adjacent columns is filled, pieces cannot cross between those columns (a row only disappears when full), so the empty cells on each side must each be a multiple of 4.
- If exactly 4 empty cells remain, they must form one connected tetromino shape.

### 4. Queue state and hold

- The queue state is **13 bits** (pieces of the current bag not yet drawn) + **3 bits** (the piece in the hold slot).
- The piece drawn in a stage is either "placed as is" or "swapped with hold". With hold off, the hold slot only delays the queue by one piece, so only the swap is allowed.
- The result of a queue state does not depend on the board, so it is computed once in a **per-stage cache**.
- `bagSpecs` describes the first m pieces bag by bag. If it cannot be expressed as one bag, it is described loosely as a single draw that holds every piece that can appear at each position. This only makes the scan looser, and every queue is checked exactly afterwards, so the result is the same.

### 5. Queue check (probability calculation)

- For each solution, all **orders it can really be placed in** (including spins, tucks and line clears) are collected (`orderSet`). Only the orders that the queue's bags and hold can produce are generated, so impossible orders are never created.
- Prefixes of the possible orders are stored in a **trie**, so one step is a single array lookup with no string building.
- The decision for one queue uses "placement order + the queue position of the piece in hold" as its state, and the queue succeeds if it can be played to the end with some order of some solution.
- Queues that use the **same order set** are solved by the same solutions and have the same score, so each order set is computed once and the queues only add their weights.
- If only the solution list is needed, it stops at the moment every solution has obtained one queue that plays it.

### 6. Expanding queue cases

- In a group, every way of choosing the elements (pieces or sub-groups) has the **same probability**, and duplicate queues are summed exactly.
- Every queue is generated on demand from a **mixed-radix index**, so no matter how many cases there are, they are never all held in memory at once.
- `estOptions` computes an upper bound of the case count without enumerating, so input such as `[[][][][][][][][][]]!` is rejected before the page freezes.
- The search looks only at the **first m pieces** of the queue. Queues with the same first m pieces are merged into one case, and the choices of a group that straddles the boundary are merged by adding their weights.
- For `{ }` order conditions, the choices are generated first, only those that satisfy the condition are kept, and the weights are normalized again.

### 7. Minimal solution set

The set that covers all PC-making queues with the fewest solutions is found as an **exact minimum hitting set**.

1. **Reduction rules**: a group with only one solution forces that solution → a solution whose groups are contained in another solution's groups is removed → a group that contains another group is removed. Repeat until nothing more can be reduced.
2. Split into **independent components** that share no solutions and solve each separately.
3. For each component, try sizes k = 1, 2, … with **iterative deepening** (the greedy solution is the upper bound). The first k that succeeds is the proven minimum. Branch on the unsolved group with the fewest candidates, and forbid the earlier sibling branches in the later ones.
4. **Lower bounds**: (A) prune if the sum of the r solutions that cover the most is not enough, (B) unsolved groups that do not overlap each need their own solution.
5. If the time limit is exceeded, stop, use the best found so far, and mark it as "not exact".

The display order is by queue weight, largest first, and with equal weight the solution with more spins comes first.

### 8. Statistics

- **Save**: the pieces of the last bag that the solution did not use. Earlier bags can be calculated only if they were drawn in full; otherwise it is unknown (`[]`).
- **Failing queues**: queues that no solution can solve, and their probability.
- **Back-to-back keep**: the same calculation is redone using only solutions whose line clears are quads, spins and PCs.
- **Score**: the best score is found by a **subset DP** that decides the placement order of a solution. Pieces of the same kind can swap roles, and line-clear and spin judgments are computed once per (solution, piece). The combinations of line-clear types possible in one play are also collected as bitmasks and only the maximal sets are kept.

### 9. See N (limited visible next pieces)

This is an **expectation maximization (expectimax)** that assumes the player sees only the current piece and the next N (and the hold) and plays the best move.

- The state is `board, hold (none / piece), whether just held, the window of visible pieces, the unknown part`.
- The unknown part follows the distribution of the queue itself (queue elements are independent of each other), and states that differ only in order, like a bag, are **interchangeable** and use the same cache value.
- When only the probability is needed, a **fast path** is used in which boards of the same stage and the same rows share the cache.
- Walking along the optimal policy gives the per-solution probability, example plays, leftover bag pieces and failing prefixes. The minimal solution set starts from the solutions used by optimal play and removes the least likely solutions one by one while the probability stays the same.
- If memory grows too large, the detailed analysis is given up and only the probability is calculated.

### 10. Setup finder

The player aims for a **setup S (the board after placing the first N pieces)**, builds S with the visible pieces (current + next N + hold), and then plays with the normal See value.

- `weight(S) = Σ start-state probability × the best value among the ways of building S`
- **Conditional probability**: P(PC | S is built) = (queues that reach a PC through S) ÷ (queues that can build S).
- The used-mino filter (must / must not) is applied during the search, so the ranking is made over **all setups that match the filter**, not only the top candidates.
- Setup keys are exact up to 5 rows (10 bits per row), and a 6-row picture (60 bits) does not fit in a double, so it is hashed to 53 bits.

### 11. Build odds (setup build mode)

Each board has a specification (next, hold, conditions), and for every queue it decides whether the target board can be built and whether the conditions (T-spin, quad, etc.) are satisfied. The per-board results are combined by **union** (at least one) or **intersection** (all), and the queues that cannot be built are shown with the highest probability first. If the conditions are so large that memory is exceeded, only the probability is calculated.

### 12. Parallel processing

- Calculations run in a **Web Worker pool**. In environments where workers cannot be used, it switches to the main thread automatically.
- **Two ways of splitting**
  - Split the queue **by first piece**.
  - Split the **boards by hash**. Every board of stage m belongs to exactly one of K parts, and each part runs the same scan/cull/place while discarding boards it does not own. The solutions of the parts **partition** the full solution set, so merging them gives the complete result.
- If needed, a final job that merges the partial results is continued by one worker.
- The screen shows the number of workers and the reason when parallel processing was not used, and as a memory safeguard, calculations that are too large are reduced to probability only.
- The number of workers differs by device (phones use fewer than PCs).

### 13. Auto-input (queue field)

The Next field has a small **editing aid** that pairs brackets. It looks at one input event and fixes the text and the caret. It changes **only the typed string**, never the calculation.

**When it runs**

- It runs only when you **type characters directly** (`insertText`, including `insertCompositionText` from mobile keyboards). Pasting, deleting, undoing and setting the value from code are not touched, so a queue you have already written is never rewritten.
- It looks only at the character just typed `d`, the caret position `p` and the text before it. After a fix it sends the input event once more so that the **case count and the `⚠` warnings are recalculated for the new string**.
- The piece letters are `I O T S Z J L`, case-insensitive.

**Rules**

| Typed | Condition | Action | Example |
|---|---|---|---|
| `[` | always | adds `]` after the caret and puts the caret between the two | `[` → `[│]` |
| `]` | an open `[` remains before it (`[` count > `]` count) | no fix, typed as is | `[SZ` + `]` → `[SZ]` |
| `]` | `[` count ≤ `]` count (unmatched), preceded by `[]` + piece letters | moves those letters inside the empty pair | `[]SZ` + `]` → `[SZ]` |
| `]` | `[` count ≤ `]` count (unmatched), preceded by piece letters | wraps those letters in `[ ]` | `SZ` + `]` → `[SZ]` |
| `}` | `{` count ≤ `}` count (unmatched), preceded by an order condition | wraps the condition in `{ }` | `[]!SZ>LJ` + `}` → `[]!{SZ>LJ}` |

- Whether an open bracket remains is decided by **counting `[` and `]` (or `{` and `}`) in the text before the caret**. It only compares counts, so the same rule applies to nested brackets.
- An **order condition** is one or more `letters <|> letters` relations in a row (including chains like `A<B<C`), and several rules can be joined by commas. Spaces between the parts are allowed. It is wrapped in `{ }` only when it is at the **very end** of the text; if it is not a condition, `}` is simply typed.
- After wrapping, the caret is placed right after the closing bracket, so you can keep typing a number or `!`.
- The auto-fix is only an **input helper**. Syntax checking and error messages are separate (`⚠` warnings, case-count errors), and input with unbalanced brackets or that does not fit the rules is still reported as an error.

---

## Syntax compared with PC solver

The queue input syntax is based on PC solver, and the following parts were **extended**. (This is based on what was found not to be supported in PC solver's syntax, and it may differ by tool or version.)

| Item | PC solver | This tool |
|---|---|---|
| 7-bag notation | All seven pieces must be written as in `[szljoit]!`, and `[]!` cannot be used | `[]!` on one line sets the default 7-bag (`[]` = `[IOTSZJL]`) |
| Flip bag status `%` | No such syntax | `%` marks or unmarks a bag. If it is a bag, the leftover-bag statistics after the PC are shown, and if not, they are not calculated |
| Order conditions `{ }` | No such syntax | `{LJ<SZ}`, `{SZ>LJ}`, chains `{A<B<C}`, commas `{A<B,C<D}` |

Other input features provided by this tool:

- Drawing only some pieces from a bag, as in `[]3!`
- Nested groups such as `[[SZ]! [LJ]!]2!`
- Typing `[` adds `]`, and `SZ]` → `[SZ]`, `[]!SZ>LJ}` → `[]!{SZ>LJ}` are fixed automatically
- A `⚠` warning when an order condition uses a piece that is not in the group, or is used on a group that draws only some pieces
- The case count is shown while you type, and inputs that are too large are rejected or warned about beforehand

**Expressiveness**: there is no queue that this syntax cannot express. `[IT]2` is the one fixed queue `IT`, and putting such fixed queues in `[ ]` makes a group that picks one of them. Example: `[[ITS]3 [JLO]3 [ZSI]3]` gives each of the three queues 1/3. Repeating the same queue also gives it a weight. A set of queues that no rule describes simply gets longer to write.

For example, `T[]!{t>j}` writes in one line "T first, then a 7-bag, with J before T inside the bag".

---

## Limitations

- The PC search supports only the **bottom 8 rows**.
- Board height is up to 12 rows in the PC finder and up to 23 rows in Build odds mode.
- Queues with a very large number of cases can take long, or only the probability is calculated to protect memory.
- Moving a board taller than 12 rows from Build odds mode to the PC finder reduces the height to 12 rows.
- If queues of different lengths are mixed with `[ ]`, the shortest length is used.

---

## Credits

The queue input syntax and the search algorithm were considerably influenced by existing **PC solvers**. Thanks to the people who made those tools.
