# Santa 2026 Public Frontier Tracker

**One input for any Santa 2026 notebook.** The shortest verified public program for each of the 1,000 puzzles of [Santa 2026 - The Elf Calculator](https://www.kaggle.com/competitions/santa-2026), merged from every public notebook output and every public dataset of the competition (~500 sources), replayed under the official rules and checked four times a day. A new version appears whenever the public frontier moves, so a notebook that attaches it starts from the current best.

## Why use it
- **Start from the best public score.** `submission.csv` is ready to submit; `puzzles.csv` and `elf_verify.py` are included, so the notebook needs no other input.
- **Improve on the current frontier.** Feed `best_programs.csv` to any optimizer (beam search, LNS, CP-SAT, peephole, register allocation) and merge back only what is shorter.
- **Spend compute where it pays.** `puzzle_stats.csv` has the gap to a trivial lower bound, instructions per target and how many sources already sit at the frontier.
- **See what is moving.** `frontier_history.csv` logs every improvement with time, old and new length and source.
- **Know when to stop recombining.** For 228 of 232 studied puzzles an exact CP-SAT bound shows that no splice of two public lineages is shorter, so new intermediate values are needed there.
- **Use it beyond the leaderboard.** 1,000 exact arithmetic program synthesis instances with verified solutions and their history.

## Content
| file | what |
|---|---|
| `submission.csv` | `id,program`: the public frontier, ready to submit |
| `best_programs.csv` | per puzzle: length, program, source, source update time, sources at this length |
| `puzzle_stats.csv` | per puzzle: sizes, trivial bound, gap, recombination verdict |
| `frontier_history.csv` | one row each time a puzzle's frontier moves |
| `sources.csv` | every source at the frontier and on how many puzzles it was first |
| `history.csv` | one row per version: total, puzzles changed, files read |
| `puzzles.csv`, `elf_verify.py` | official puzzles, unchanged, and the validator used for every program |
| `l9_summary.csv`, `l9_lineages.json` | the recombination study (232 puzzles, two lineages, CP-SAT results) |

Programs are lists of `[op, i, j, d]`; `README.md` documents every column.

```python
import pandas as pd
base = "/kaggle/input/santa-2026-two-program-lineages/"
best = pd.read_csv(base + "best_programs.csv")
print(best.instructions.sum())   # the public frontier score = last total in history.csv
stats = pd.read_csv(base + "puzzle_stats.csv")
stats.sort_values("gap_to_trivial_bound", ascending=False).head(20)
```

Only public work, and only programs that replay correctly. `source` is the earliest-updated source holding the length: read it as where to find it, not as authorship.

## Inspiration
1. Run your optimizer on the 50 puzzles with the largest `gap_to_trivial_bound` and compare your total with `history.csv`.
2. Join `frontier_history.csv` with `sources.csv` to see which sources still move the frontier and how often.
3. Train a model to predict `frontier` from `puzzles.csv` and find the puzzles that are longest for their size.

## Acknowledgements
All programs come from public Kaggle notebooks and datasets (Apache 2.0); `sources.csv` names each one. Analysis: [Santa 2026: Why the First Target Costs 8x More](https://www.kaggle.com/code/georgymamarin/santa-2026-why-the-first-target-costs-8x-more).

## Columns

### best_programs.csv (one row per puzzle)
| column | meaning |
|---|---|
| id | Puzzle id, as in puzzles.csv. |
| instructions | Length of the program (number of instructions); the score of this puzzle. The sum over all puzzles is the public frontier score. |
| source | owner/slug of the earliest-updated public notebook or dataset whose output holds this length. Where to find it, not proof of authorship: merges copy the same program. |
| source_type | notebook or dataset. |
| source_updated_utc | Last update of that source on Kaggle (UTC) when it was read. |
| n_sources_at_length | How many public sources hold a program of exactly this length. 1 means a single source is ahead of everyone else on this puzzle. |
| program | Program as JSON: a list of [op, i, j, d] (op in + - * /, registers 0..47, starting values in registers 0..k-1). |

### puzzle_stats.csv (one row per puzzle)
| column | meaning |
|---|---|
| id | Puzzle id, as in puzzles.csv. |
| n_start | Number of starting values. |
| n_targets | Number of targets. |
| new_targets | Targets that are not among the starting values. Each needs at least one instruction, so this is a trivial lower bound on the length. |
| value_limit | Largest absolute value a register may hold (from puzzles.csv). |
| frontier | Length of the best public program (= instructions in best_programs.csv). |
| gap_to_trivial_bound | frontier - new_targets. An upper limit on what any program can still gain. |
| instructions_per_target | frontier / n_targets. |
| n_sources_at_length | How many public sources hold a program of the frontier length. |
| recombination_verdict | Only for the 232 puzzles of the l9 study, empty elsewhere. "no shorter splice": no program built from the values of the 22-23 Nov public lineages is shorter than their live program; "open": the bound did not reach it. |
| recombination_lower_bound | CP-SAT lower bound on the length of any splice of those lineages (= m_lower_bound in l9_summary.csv). A frontier below it was reached with new values. |

### frontier_history.csv
| column | meaning |
|---|---|
| build_utc | Time of the build that saw the new length (UTC). |
| id | Puzzle id. |
| instructions | New frontier length. |
| previous | Frontier length before; empty on the first build. |
| source | Source holding the new length (see best_programs.csv). |

### sources.csv
| column | meaning |
|---|---|
| source | owner/slug on Kaggle. |
| source_type | notebook or dataset. |
| puzzles_first_at_frontier | Puzzles where this source is the earliest-updated holder of the frontier length (the one named in best_programs.csv). |
| puzzles_at_frontier | Puzzles where this source holds the frontier length at all. |
| instructions_first_at_frontier | Sum of frontier lengths over puzzles_first_at_frontier. |

### history.csv
| column | meaning |
|---|---|
| build_utc | Build time (UTC). |
| total | Sum of frontier lengths over all 1,000 puzzles: the public frontier score. |
| puzzles_changed | Puzzles whose frontier moved since the previous build. |
| files_read | Submission files read from public sources in this build. |
| sources | Sources holding the frontier somewhere (rows of sources.csv). |

### l9_summary.csv (232 rows, one per puzzle)
| column | meaning |
|---|---|
| puzzle | Puzzle id from the competition's puzzles.csv. |
| set | X: the night program shares less than 70% of its values with the old one (153 puzzles); N: 70% or more (79). |
| family | Our label for how the puzzle's targets relate: mixed, multiplicative, shared, independent. |
| live_len | Instructions in the live program (merged public pool of 23 Nov 2026). |
| old_len | Instructions in the old program (merged public pool of 22 Nov 2026). |
| night_len | Instructions in the night program that replaced it. |
| shared_values | Share of the night program's distinct values that the old program also builds. |
| verdict | Arm M: "no shorter splice" when the CP-SAT lower bound reached live_len, otherwise "open". |
| m_lower_bound | CP-SAT lower bound of arm M on the number of values, hence on the length of any splice. |
| s_passed | Arm S found an acyclic solution no longer than night_len (the control worked on this puzzle). |

### l9_lineages.json
- `about`: one-paragraph description.
- `X`, `N`: puzzle ids of each set.
- `puzzles[id]`: `family`; `old`, `night`, `live`: programs as lists of `[op, i, j, d]` (op in + - * /, registers 0..47,
  starting values in registers 0..k-1, the competition's rules); `results`: one entry per arm `A`, `M`, `S` with
  `thr` (length the arm is compared with), `lb` (CP-SAT lower bound), `status` (BOUND = bound reached thr;
  OPTIMAL/FEASIBLE = solved; ROUNDS = cycle cuts ran out), `acyclic` and `nvals` (whether a cycle-free choice of
  recipes was found and how many values it builds), `nv`/`nr` (values and recipes in the graph), `rounds`, `sec`.

## Recombination study (files l9_*)
In the night of 22 to 23 Nov 2026 a second, independent family of programs entered the public pool. For 153 puzzles
(set X) the new "night" program shares less than 70% of its values with the "old" one it replaced; set N (79 puzzles,
70% or more shared) is the control. A CP-SAT model took every value either program builds, allowed every recipe
`a op b = v` among those values, and searched for the shortest program inside that graph. Its lower bound reached the
live program's length on 228 of the 232 puzzles, so no splice of the lineages is shorter there.
Arms: A = recipes of the live program only (control of the solver); M = live + old + night (the question);
S = old + night with the solver started from the old program (must find the night length; a control that can fail).
Every arm adds all cross recipes `a op b = v` whose three numbers are values of the graph. Settings: 20 s per solve,
up to 40 rounds of lazy cycle cuts, 8 CP-SAT workers.
Limits: the graph has no bridges (values outside both programs), registers are ignored (the bound only gets weaker),
and "live" is the merged public pool of 23 Nov (130,890 instructions), not today's frontier. The frontier has since gone
below the bound on many of these puzzles, which is the point: it needed values neither lineage had.
Full analysis: section 4 of the notebook linked above.
