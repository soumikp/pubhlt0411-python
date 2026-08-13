# Module 6 — Reproducible pipeline

**PUBHLT 0411 · Python for Public Health Data Analysis**
**Session:** Tue 11/17 · **ONE COMBINED SESSION** — not a lecture/lab pair
**Home story:** S5 — firearm injury, Pennsylvania counties (⭐ home cell)
**Packages required:** `numpy`, `pandas`, `matplotlib` *(no micropip, no seaborn)*

## One skill

**Run the pipeline end-to-end, reproducibly, and write up what it means.**

## Structure — this module is shaped differently from M0–M5

Per §3 and §4 this is **one working session**, not a lecture with a lab two days later. The
document is a single file with two halves:

| Slides | Half | Surface rules |
|---|---|---|
| 1–14 | **Teaching**, ~20 minutes | §2 lecture: academic slides, conversational speaker notes |
| 15–51 | **Studio**, the rest of the session | §2 labs clause: `**Your turn.**`, hints, no speaker notes |

Because both surfaces live in one document, the register rule switches halfway. The teaching
half carries **15 speaker-notes blocks**; the studio half carries none, because nobody is
lecturing over an exercise.

## File manifest

```
module6/
├── class/
│   ├── module6_session.qmd        # the combined session (live-revealjs)
│   ├── module6_session.html
│   ├── pitt-theme.scss            # ⚠️ the module5/class copy — see below
│   ├── data/                      # VFS copies — required
│   │   ├── pa_firearm_county_year.csv
│   │   └── pa_counties.csv
│   └── _extensions/r-wasm/live/
└── README.md
```

**No `lab/` directory.** There is no separate lab; the studio exercises are in the same file.

Both CSVs are listed under `pyodide: resources:`. Per §7, a CSV dropped in `data/` that is
**not** listed there never reaches the browser. Both copies verified byte-identical to `data/`
by MD5.

**VFS manifest verified after rendering** by base64-decoding the `<script type="vfs-file">`
block. It is a plain JSON array and it reads exactly:
`["data/pa_firearm_county_year.csv","data/pa_counties.csv"]`.

## Rendering

```bash
cd class && quarto render module6_session.qmd --to live-revealjs
```

## ⚠️ The `pitt-theme.scss` in this folder is NOT the repository-root copy

The root `pitt-theme.scss` **does not contain the `.figure-description` rule.** Only the
`module5/` copies do. This module's copy was taken from `module5/class/`. **Copying the root
theme over it silently removes the styling from every figure description in the module** —
the text would still be present and readable, but unstyled.

## The cold open and its payoff

**Open:** Firearm death is understood as an urban problem, and Pennsylvania's data disagrees.
The question is left unanswered for the first twenty minutes.

**Payoff:** **nonmetro 18.5 per 100,000 against metro 14.1** (2023, pooled over the 42
reporting counties), with the **suicide share — 62.1% nonmetro against 54.7% metro** —
supplying the explanation.

Per §6 this is **the only finding in the course meant to surprise students.** Every other
module confirms what a public-health student already expects. The closing notes say so
explicitly, because that contrast is the reason this story was saved for last.

## ⚠️ THE CENTRAL FINDING OF THE BUILD — the payoff exists only if suppression is handled correctly

**This was discovered by running the pipeline both ways against the real files, and it is
more valuable pedagogically than the specification anticipated.**

| 2023, pooled | Correct (`.dropna()` first) | Naive (no `.dropna()`) |
|---|---|---|
| Metro | **14.1** | 13.6 |
| Nonmetro | **18.5** | **8.2** |

The naive pipeline **runs cleanly, raises nothing, produces a well-formed table — and reverses
the finding**, reporting nonmetro rates as dramatically *lower* and thereby confirming exactly
the stereotype the module exists to overturn.

**Mechanism.** `.sum()` skips NaN; the population column does not. A suppressed county
contributes its **full population to the denominator and zero deaths to the numerator**. This
is §6 trap 3, and in this module it is not a footnote — **it is the mechanism that produces
the entire surprise.**

**Why this matters for teaching:** a student who forgets one `.dropna()` does not get an
error. They get a plausible table that agrees with their prior belief. That is the most
dangerous failure mode in applied data analysis, and here it is reproducible in five lines.

**It is built into the module three times:** the teaching contrast slide, **Exercise 6**
(which hands students the broken pipeline and asks them to repair it), and the closing
callout. Exercise 6 is the intended difficulty spike; the speaker notes tell the instructor
not to rescue students from it too quickly.

## ⚠️ The suicide share depends on its definition — 62.1% vs 83.7%

Two defensible definitions differ by **twenty-two points**:

| Definition | Nonmetro share |
|---|---|
| Suicide NaN → 0, across counties reporting a death total | **62.1%** *(used)* |
| Restricted to counties reporting **both** columns | 83.7% |

The suicide column is suppressed **more often** than the death column (190 vs 159 cells), so
restricting to rows with both quietly discards counties. The chosen definition is **stated on
the slide next to the number**, and the callout makes the general rule explicit: *the number
moves when the definition moves, so the definition travels with the number.*

Both §4 target figures (62% / 55%) reproduce exactly under the chosen definition:
**62.1% and 54.7%.**

## Verified numbers — every figure on every slide was computed from the real files

| Quantity | Value |
|---|---|
| Nonmetro pooled rate, 2023 | **18.5** per 100,000 |
| Metro pooled rate, 2023 | **14.1** per 100,000 |
| Nonmetro suicide share | **62.1%** |
| Metro suicide share | **54.7%** |
| Naive (wrong) nonmetro / metro | **8.2** / **13.6** |
| Counties reporting a 2023 total | **42** of 67 |
| Suppressed cells, all years | **159** of 402 (39.6%) |
| Suppressed 2023, nonmetro / metro | **19** of 30 / **6** of 37 |
| Deaths behind the nonmetro rate | **116** across 11 counties |
| Deaths behind the metro rate | **1,567** across 31 counties |
| Reported deaths, 2023 total | **1,683** |

**The finding holds in five of the six years** 2019–2024 (the exception is 2020), which is
drawn as the trend figure and used as the robustness sentence in the model report paragraph.

| Year | Metro | Nonmetro |
|---|---|---|
| 2019 | 11.6 | **18.0** |
| 2020 | **13.8** | 13.4 |
| 2021 | 15.1 | **16.3** |
| 2022 | 15.3 | **16.1** |
| 2023 | 14.1 | **18.5** |
| 2024 | 12.4 | **15.7** |

## Verification performed

Local venv: pandas 3.0.5 / numpy 2.5.2 / matplotlib 3.11.1.

- **Full top-to-bottom run: 33/33 cells execute, zero failures.** This simulates the browser
  path — every narrative cell plus every exercise *solution*, in document order, in one shared
  namespace. The final cell emits the §4 payoff sentence verbatim.
- **9/9 exercise solutions pass their `#| check:` asserts.**
- **9/9 checks discriminate** — each fails against a namespace with no student work.
- **9/9 unedited blanks raise before their check is reached** (`AttributeError`, `SyntaxError`,
  `TypeError` ×2, `KeyError` ×2, `NameError`, `AssertionError`), so no check can be satisfied
  by submitting the exercise unmodified.
- **All 9 exercises carry all four cell roles** — blank, hint, solution, check.
- **Artifact verified, not the source** (the M5 lesson): both `figure-description` blocks
  confirmed present in the rendered HTML by string match; 0 `<img>` and 0 `<canvas>` as
  expected; the VFS manifest decoded and both CSV paths confirmed.

### Two defects found by running the code, and fixed

1. **Exercise 2's worked example originally filtered to 2019, which has 25 suppressed cells —
   coincidentally identical to 2023's 25.** The worked example therefore appeared to be
   handing students the answer to their own task. Changed to **2021 (29 suppressed)** so the
   two numbers visibly differ.
2. **Five checks passed trivially.** The teaching half legitimately builds `counties`,
   `f2023`, `reporting`, `joined`, and `summary` — the same names Exercises 1–5 ask students
   to create. Because quarto-live shares one namespace across the whole document, a student
   could skip those five exercises entirely and every check would still pass on the lecture's
   leftover variables. **Fixed by renaming the teaching half's variables to a `demo_` prefix**
   (`demo_2023`, `demo_reporting`, `demo_joined`, `demo_summary`, `demo_share`, `demo_wrong`)
   and having Exercise 1 build `county_reference` rather than `counties`. The teaching slides
   still show the real pipeline; the studio now genuinely starts from nothing.

   **This is a general hazard for any single-document module** and is worth checking in the
   HW notebooks: in a shared namespace, a check tests the namespace, not the student.

### quarto-live enforces `______` itself — found in the extension source

`live-runtime.js` runs a `blankCheck` before grading: any submitted cell still matching
`/_{6}_*/g` is rejected with *"Please replace ______ with valid code."* **Six-underscore
blanks are therefore enforced by the runtime**, and an assert like `"______" not in answer` is
redundant (one was dropped from Exercise 8).

The same source shows the check cell receives `user_code`, so a check can inspect the
student's source, not only their output.

⚠️ **A blank must be six or more underscores to be caught. `____` (four) is not.** M6 uses
six throughout. **The M1–M5 labs use four**, and a count of their `{pyodide}` cells finds
**73 blanks the runtime does not enforce**:

| Lab | Non-enforced `____` blanks |
|---|---|
| M1 | 19 |
| M2 | 15 |
| M3 | 12 |
| M4 | 15 |
| M5 | 12 |

**Consequence, and it is minor.** `____` is a **valid Python identifier**, so those cells
compile — verified: all 12 M5 lab blank cells compile cleanly. An unedited submission
therefore fails at *runtime* (`NameError`, or `AttributeError` where the blank is a method
name) rather than being intercepted by the runtime's *"Please replace ______ with valid
code."* The exercises still discriminate — M5's README records that every unedited blank
raises before its check — so nothing grades incorrectly. The student just gets a raw Python
error instead of a sentence telling them what to do.

**Not changed, because it spans five shipped modules and is the instructor's call.** The fix
is mechanical (widen `____` to `______` in the lab `{pyodide}` cells, and update the one line
of M5 lab framing text that tells students to look for `____`) and belongs with the pre-publish
pass if it is wanted at all.

### Register audit (§2 split-register)

- **Teaching half:** second person appears **only** inside `::: {.notes}` blocks. Zero
  violations in slide prose.
- **Studio half:** second person confined to `**Your turn.**` task instructions (including
  their wrapped continuation lines) and hints, per the §2 labs clause.
- **Litotes scan: 0 hits** across both surfaces.
- **Structure:** 10 section dividers · 51 content slides · 15 notes blocks · 9 `**Your turn.**`
  markers · 9 `— your turn` slide headings · **no exercise cell on a worked-example slide.**

## §4 coverage

| Required | Where |
|---|---|
| import → clean → join → group → summarise → visualise → report | The seven-stage table, then Exercises 1–9 in that order |
| Notebook hygiene, imports at top, restart-and-run-all, seeds, relative paths | The hygiene slide; restart-and-run-all repeated in the closing callout and the studio notes |
| **Writing the report (30% of the grade)** | The five-move report slide, the model paragraph slide, and Exercise 9 |
| **Interpreting uncertainty verbally** — small counts, unstable rates, why suppression exists | The uncertainty slide (Juniata worked example in the notes) and the suppression slide |
| No p-values (§8) | Stated as a callout on the uncertainty slide |

**The 30% writeup weight is stated twice on slides and twice more in the notes**, per M5's
closing promise to flag it early rather than in the last week.

## ⚠️ §8 — descriptive only

No p-values, no significance tests, no confidence intervals, no `scipy.stats`, no
scikit-learn. **Uncertainty is treated verbally**, which is the §8 mitigation for the LO4
accreditation risk: the reported estimate always travels with its event count and coverage.

The uncertainty slide's notes carry the concrete worked example — **Juniata County: 10 deaths,
23,243 people, 43.0 per 100,000, the highest county rate in the state.** Moving two of those
ten deaths into an adjacent year drops it to roughly 34. That makes instability tangible
without a distributional assumption.

## Accessibility

- **Two figures, both carrying a visible `::: {.figure-description}` block.** `fig-alt` is
  **not** used and does not work in this format — see `module5/README.md` for the full
  finding (`live.lua` knows only fig-width/fig-height; `live-runtime.js` never sets `.alt`).
- **Colour is never the only channel.** The trend figure distinguishes its two series by
  **marker shape as well as colour** (triangles for nonmetro, circles for metro), because the
  two palette colours separate from each other at only **2.02:1**.
- **Palette:** `#003594` royal (10.84:1) and `#8a6400` dark gold (5.38:1), both measured.
- **Zero baselines** on both bar charts; Exercise 7's check asserts `ax.get_ylim()[0] == 0`.
- **Exercise 8 asks the student to write a figure description themselves**, and its check
  requires the chart type and both values — the accessibility habit is practised, not just
  described.

## Studio design

**Four of the nine supply no code.** Exercises 5, 7, 8, and 9 state a question in plain
English with explicit deliverables and leave the cell empty; the hints carry the scaffolding
that would otherwise be in the cell. The studio slide warns students that the jump is coming,
and the speaker notes tell the instructor to expect hands up at Exercise 5 — the first blank
page — with the unsticking move being *describe the finished table before writing it*.

| Ex | Stage | Form | Teaches |
|---|---|---|---|
| 1 | Import | 4 blanks | Load; confirm 67 rows **and one row per county** |
| 2 | Filter | 4 blanks | One year out of six; coverage **as a proportion** |
| 3 | Clean | 4 blanks | `.dropna(subset=)` vs bare `.dropna()`, and **which 23 counties** the careless form discards |
| 4 | Join | 5 blanks | `.merge()`; unmatched keys **and** row-count preservation |
| 5 | Group | **open-ended** | `.groupby().agg()` and the rate — **produces 18.5 / 14.1** |
| 6 | **The silent error** | 5 blanks | The broken pipeline repaired, **and the damage quantified** |
| 7 | Visualise | **open-ended** | A figure meeting stated requirements: zero baseline, labels, palette |
| 8 | Describe | **open-ended** | A description a screen-reader user could rebuild the figure from |
| 9 | Report | **open-ended** | The paragraph, with every number computed rather than typed |

**Exercise 6 now quantifies the error rather than only correcting it:** **792,973** nonmetro
residents sat in the denominator with no deaths above them, understating the nonmetro rate by
**10.3 per 100,000** — a larger error than the metro rate itself. Both values measured.

**Exercise 3 is quietly load-bearing.** A bare `.dropna()` leaves **19** counties rather than
42, because it also discards counties whose *homicide* or *suicide* cell is suppressed while
the total is known. The task now makes students **print the 23 names**, which is the same
class of error as Exercise 6 met earlier and more gently.

⚠️ **Two claims about those 23 counties were wrong on first writing and corrected by
measurement.** Philadelphia is **not** among them — it reports all three columns. And they are
**not** predominantly small rural counties: the list includes **Bucks, Chester, Westmoreland,
and Dauphin**, among the largest in the state. Losing them would gut the metro group, which is
a better lesson than the one originally drafted. The assert checks for Bucks and Chester.

**Exercise 9 enforces the anti-fabrication rule** from the hygiene slide. The check requires
`18.5`, `14.1`, `42`, `67`, `116`, and `11` in the output, plus the words *metropolitan* and
*exclud*, plus a 45-word minimum so it is prose rather than a list of numbers. A student who
types the numbers by hand still passes — no check can detect that from the output alone — so
the hint gives them the test that does: **change the year to 2022 and re-run.** An f-string
paragraph updates itself; a typed one keeps claiming 18.5 while the table above it disagrees.

### A check that would have passed the wrong answer

Exercise 8's check originally asserted `"metro" in description` and `"nonmetro" in
description`. **`"metro"` is a substring of `"nonmetro"`,** so a description naming only the
nonmetro group satisfied both. It now strips `nonmetro` before counting `metro`, and the fix
was verified by running a deliberately one-sided description through it.

## Handoff — what remains

**M6 completes the seven-module sequence (§10 step 2).** The remaining build work is §10
steps 3 and 4:

1. **The HWs and the final-project spec** (`.ipynb`, authored for Colab). M6 is explicitly the
   project skeleton, so the project spec should reuse its seven stages and its five-move
   report structure verbatim.
2. **Re-run `tools/audit.py`** across M0–M6 to confirm §4 concept coverage is real rather than
   intended, then run `PREPUBLISH_CHECKLIST.md`.

### Standing carry-forwards — unchanged, do not re-litigate

- Lab `#| check:` assert values across M1–M6 still need one confirmation against **live
  Pyodide** (Solution → Run in a real browser). The asserts here were verified against local
  pandas 3.0.5; Pyodide ships 2.3.1. Nothing in this module uses behaviour that differs
  between them, but the browser run is the authoritative check.
- **VoiceOver pass** on quarto-live's Run/Solution/Hint buttons and whether cell output is
  announced. Not fixable from a `.qmd` — it is extension behaviour. Documented in
  `module5/README.md`.
- In-browser keyboard/focus and 200% zoom checks deferred to pre-publish.

### Open items the instructor has not decided — still open, still not acted on

- **R bridge callouts are absent from all of M0–M6.** §12 asks for one-line bridges at the
  first parallel concept. This module had two natural sites (`.merge()` against `left_join()`,
  and the pipeline against a `dplyr` chain) and **neither was added**, because the decision
  spans all seven modules and is the instructor's call.
- **No chart-choice guidance** (question → data shape → mark). §4 lists it; M5 teaches four
  chart types and when each is wrong. M6 needs it most, since students now choose their own
  figures for the project, but adding a decision procedure here alone would leave it
  undiscoverable in M5 where the chart types are actually taught.
