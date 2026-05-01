# Aggression: Game Analysis Suite
**Python algorithmic analysis of the Aggression board game — optimal solvers, AI opponents, and strategic opening heatmaps**

*Research project — Department of Mathematics, Berea College (May–Oct 2025)*

> **Related repo:** [`Aggression_Game_For_m-n`](https://github.com/adhikarisamundra/Aggression_Game_For_m-n) — browser-based 3D implementation of the same game with Three.js, MCTS AI, and a multithreaded simulation lab. This repo is the Python-side algorithmic research.

---

## Overview

This suite investigates the Aggression board game from a combinatorial game theory perspective. The core research questions:

- Under optimal play, which player has a guaranteed win for a given m×n grid?
- How does AI search depth interact with play quality, and can deliberate imperfection produce more realistic opponents?
- Which opening positions maximize P1's win probability across different troop allocation strategies?

**Game rules (brief):** Two players alternate placing troops on an m×n grid, each starting with n² units. The player who depletes their reserve first earns initiative for the attack phase. A cell is captured if the sum of adjacent enemy troop counts exceeds its defender count. Victory is determined by cells controlled, then surviving troop count, then initiative.

---

## Repository Contents

| File | Purpose |
|------|---------|
| `Grid_Map_Aggression_Game_Solver.ipynb` | Full alpha-beta solver — determines the game-theoretically optimal outcome for any grid size |
| `Aggression_Human_vs_AI.ipynb` | Interactive Human vs AI game with three difficulty levels and calibrated AI imperfection |
| `strategic_opening_analyzer.py` | Exhaustive opening analysis — scores every P1 starting position and generates a win-rate heatmap |
| `Grid_Map_Game_Simulator.ipynb` | Earlier iteration of the Human vs AI notebook — kept for version history |

---

## Notebooks in Detail

### `Grid_Map_Aggression_Game_Solver.ipynb` — Optimal Game Solver

Determines which player wins under perfect play for a given grid size. Implements full minimax with alpha-beta pruning and memoization keyed on `(grid, p1_troops, p2_troops, turn_player)`.

- Alpha-beta pruning eliminates branches provably worse than already-found alternatives
- Heuristic move ordering (fewer troops first) maximizes early pruning effectiveness
- Memoization caches results by canonical state tuple to avoid re-analyzing transpositions

**Output** — reports the guaranteed winner, number of unique states analyzed, and computation time:

```
Grid Size: 3x3
Unique States Analyzed: 14,832
Time Taken: 2.3471 seconds
Result: Player 1 has a guaranteed win under optimal play.
```

**Practical limits:** Tractable up to ~9 cells (3×3). Grids above this trigger a confirmation prompt — the exponential state space makes larger grids infeasible without sampling heuristics beyond alpha-beta.

---

### `Aggression_Human_vs_AI.ipynb` — Human vs AI

Interactive terminal game against an AI at three configurable difficulty levels.

| Level | Search depth | Imperfection |
|-------|-------------|--------------|
| Easy | 2 | 25% chance of playing 2nd-best move |
| Challenging | 4 | 15% chance of playing 2nd-best move |
| Complex | ∞ (full tree) | None — plays optimally |

The imperfection logic sorts scored moves post-search from best to worst, then with a configurable probability selects the second-best move rather than the optimal one. This produces a more realistic and educational opponent than random noise.

Key functions:
- `find_best_ai_move()` — alpha-beta minimax with depth limit and memoization; returns both the score and the best move object
- `evaluate_final_state()` — simulates both attack phases from a terminal state; returns +1 (P1 win) or −1 (P2 win) using three-tiered tiebreak (cells → troop count → initiative)
- `run_simulated_attack()` — applies 8-neighbor adjacency attack rule, including the 3×3 special case restricting center-cell diagonal attacks
- `display_grid()` — ANSI color-coded terminal rendering (P1 in blue, P2 in red)

**How to run:** Open in Google Colab, execute the single cell, and follow the prompts to configure grid size, difficulty, and player assignment.

---

### `strategic_opening_analyzer.py` — Opening Heatmap Generator

Exhaustively evaluates every possible P1 first move and scores each cell by how many troop allocation strategies lead to a P1 win under subsequent optimal play by both sides.

For each cell `(r, c)`, three opening amounts are tested: `1`, `n²//2`, and `n²` (all-in). For each combination the cache is cleared, the opening move is applied, and `minimax()` runs from the resulting state with P2 to move. Cell score = number of allocations that produced a P1 win (range: 0–3).

```
Score interpretation:
3 — P1 wins regardless of opening troop allocation
2 — P1 wins with most allocations
1 — P1 wins with only one allocation
0 — P1 loses with all tested allocations from this position
```

**Output:** A matplotlib heatmap using the `'hot'` colormap, annotated with raw scores per cell, saved as `strategic_heatmap.png`. Brighter (hotter) cells indicate more strategically robust opening positions.

**Note:** Each cell requires a full minimax solve with a cleared cache. For a 3×3 grid this means 27 separate full-tree evaluations. Grids above 3×3 include a confirmation prompt due to computation time.

---

## Technical Notes

**Memoization:** All solver functions use a global `memoization_cache` dictionary. The cache **must be cleared** between independent analyses — the notebooks handle this explicitly. Failure to clear produces incorrect results when analyzing multiple grid sizes in sequence.

**Move generation pruning:** Rather than enumerating every troop count from 1 to n², `get_possible_moves()` restricts options to `{1, n²//2, n²}` for reserves above 6 and the full range for reserves ≤ 6. This is a heuristic that substantially reduces branching factor at the cost of theoretical completeness for large reserves.

**State representation:** The grid is a list of lists of `None | (player, troop_count)` tuples. Canonical form converts this to nested tuples for dictionary hashability.

---

## Running the Notebooks

All notebooks run in Google Colab with no dependencies beyond the Python standard library (`sys`, `time`, `math`, `random`) and `matplotlib` / `numpy` for the heatmap analyzer. Click the **Open in Colab** badge at the top of each notebook and run the cell.

---

## Relationship to the Browser Implementation

[`Aggression_Game_For_m-n`](https://github.com/adhikarisamundra/Aggression_Game_For_m-n) implements the same game as a browser-based 3D app using Three.js. That repo uses **Monte Carlo Tree Search (MCTS)** rather than minimax, enabling play on boards up to 12×12 where exhaustive search is infeasible, and includes a multithreaded simulation lab running parallel Web Workers to generate empirical win-rate heatmaps at scale.

The two repos are complementary: this suite provides exact game-theoretic analysis on small grids; the browser repo provides scalable simulation and playability on larger ones.
