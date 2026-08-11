# Module 1 — Conditions and Sequences

**PUBHLT 0411 · Python for Public Health Data Analysis**
**Lecture:** Thu 10/8 · **Lab:** Tue 10/13
**Home story:** S1 — drug overdose deaths, Pennsylvania counties (⭐ home cell)
**Packages required:** none (`packages: []`)

## One skill

**Branch on a condition and hold a sequence of values.**

## File manifest

```
module1/
├── class/
│   ├── module1_lecture.qmd       # lecture slides (live-revealjs)
│   ├── module1_lecture.html      # rendered output
│   ├── pitt-theme.scss
│   └── _extensions/r-wasm/live/
├── lab/
│   ├── module1_lab.qmd           # Py Lab 1 (live-revealjs, smaller: true)
│   ├── module1_lab.html          # rendered output
│   ├── pitt-theme.scss
│   └── _extensions/r-wasm/live/
└── README.md
```

**No `data/` directory and no `pyodide: resources:` block.** §4 specifies inline county
records for M1, so nothing is loaded from the VFS. Every number is nonetheless the real
2023 value from `data/pa_overdose_county_year.csv` — see the verification section.

## Rendering

```bash
cd class && quarto render module1_lecture.qmd --to live-revealjs
cd lab   && quarto render module1_lab.qmd     --to live-revealjs
```

**Labs are `live-revealjs`, not `live-html`** (§7). The v1 READMEs say `live-html` and are
wrong. `smaller: true`, one exercise per slide.

The `{{< include _extensions/r-wasm/live/_knitr.qmd >}}` line must sit directly after the
YAML in both files, or the `{pyodide}` cells render as dead text.

## The cold open and its payoff

**Open:** Module 0 established the two rates but never said which county was which. M1
opens on that gap — *which is which, and how would that be determined without being told?*

**Payoff:** Schuylkill classifies **High** (41.7, above the state rate of 25.6); Centre
classifies **Low** (4.4, below the Healthy People 2030 target of 13.1). The code decides,
from the numbers, using the two thresholds.

**The structural explanation is on the payoff slide**, per §5's framing rule: overdose
mortality concentrates in adults 35–64, Centre's population is young because of the
university, and Schuylkill is an older post-anthracite county. **Age composition and
economic history, not individual conduct.**

## The three real thresholds

Every classification in this module uses published anchors, not invented cut-points:

| Anchor | Value | Source |
|---|---|---|
| Pennsylvania overdose mortality rate | **25.6** per 100,000 | KFF/CDC NCHS |
| Healthy People 2030 target | **13.1** per 100,000 | HHS |
| Schuylkill / Centre 2023 | 41.7 / 4.4 | PA DOH, verified against the file |

Applied to all 60 unsuppressed counties in 2023, these bands give **34 High, 21 Moderate,
5 Low** — a usable spread, which is why they work as teaching thresholds.

## Counties used, and why each one

| County | Deaths | Population | Rate | Role |
|---|---|---|---|---|
| Schuylkill | 60 | 143,786 | 41.7 | ⭐ the High case, carried from M0 |
| Centre | 7 | 157,795 | 4.4 | ⭐ the Low case, carried from M0 |
| Franklin | 30 | 157,854 | 19.0 | **exercises the `elif`** — the only middle case in lecture |
| Beaver, Cambria, Adams, Lebanon, Fayette, Blair, Lawrence | — | — | — | lab exercises |

**Franklin is doing real work.** Its population (157,854) is within 60 people of Centre's
(157,795), and its rate falls between the two thresholds — so it is both a second
near-identical-population comparison and the only value that reaches the `elif` branch.

## Verification status

**Lecture — all 18 `{pyodide}` cells run.**

```bash
cd class && python3 ../../tools/run_cells.py module1_lecture.qmd
```

The `IndentationError` checkpoint raises as designed. The `m1_classify` blank is skipped;
its solution prints `Franklin: 19.0 per 100,000 — Moderate`.

**Lab — all 12 exercises pass, and all 12 checks discriminate.**

```bash
cd lab && python3 ../../tools/verify_lab.py module1_lab.qmd
```

`discriminates` means the check **fails** against setup-only with no student work, per §7.
A check that passes trivially grades nobody. 27 asserts total across 12 exercises.

**Every county figure was verified against the real file, and this caught an error.**
Exercise 12 originally used Lawrence County at 60 deaths / 83,111 population — **invented
numbers.** The real 2023 row is **61 deaths / 84,472**. Both give a rate of 72.2 to one
decimal, so the error would have survived any spot-check of the printed output. Fixed, and
the assert now targets 72.2133.

**The two-stage error in Exercise 12 was confirmed by compilation**, not assumed:

```
SyntaxError: expected ':' (line 4)          <- first
IndentationError: expected an indented block <- after the colon is fixed
```

The hint tells students to expect exactly this sequence.

## Teaching devices

- **Two checkpoints** (§2 error-callout form — message, cause, fix on the slide; candor in
  the notes):
  - **A silent error** — comparing `overdose_deaths > 25.6`, a count against a rate. It
    runs, prints a plausible answer, and raises nothing. **Verified: the bug produces the
    right answer for Schuylkill and Centre and only fires on Butler** (43 deaths, rate
    21.7 — the count says "above," the rate says "below"). The notes tell the instructor to
    switch to Butler live.
  - **A loud error** — `IndentationError` from an unindented block.
- **One fill-in-the-blank live demo** (`m1_classify`), Franklin County, blank on the 13.1
  threshold.
- **Twelve lab exercises**, each worked-example-then-your-turn, with hint, solution, and a
  discriminating check.
- **Exercise 5 carries a real statistical point**: the four-county combined rate (21.2) is
  not the average of the four county rates (21.7). Averaging rates weights counties
  equally; combining counts weights by population. This is the rates-vs-counts thread that
  becomes a full check-in in M5 (§6, trap 1).
- **Exercise 8 uses the real `"Mc Kean"` / `"McKean"` mismatch** — the actual spelling
  disagreement between County Health Rankings and the Census that breaks a join in M4.

## Voice sheet compliance (§2, amended 2026-08-11)

| Surface | Second person | Result |
|---|---|---|
| Lecture slides | 0 instances | clean |
| Lecture notes | 18 blocks, 1,633 words | load-bearing |
| Lab prose, callouts, explanations | 0 instances | clean |
| Lab task instructions (`**Your turn.**`) | 10 instances | **expected — see below** |

**§2 gained a "Labs — the third surface" clause while this module was built.** A lab has no
speaker-notes surface, so the slide/notes split does not apply. Task instructions to a
student performing an action are second person and imperative by nature; rewriting
*"Compute its rate"* as *"the rate is to be computed"* is the padded register §2 forbids.

**Audit a lab differently from a lecture:** confirm second person appears **only** in
`**Your turn.**` task instructions and in hints — never in framing prose, callouts, or
error explanations. M1's lab passes that test.

## Accessibility (WCAG 2.2 AA)

- **No figures in M1** — text, tables, and code only. No alt-text obligation.
- The index ruler on the string-indexing slide is a fenced code block, so it reads as
  preformatted text rather than as an image.
- Inherits the audited `pitt-theme.scss`. No new colors.
- **Run `PREPUBLISH_CHECKLIST.md`** before publishing.

## Concept coverage (§4)

Comparisons · `and` / `or` / `not` · `if` / `elif` / `else` · **the colon and the indent** ·
`+=` · strings as sequences · indexing · **zero-indexing** · slicing `[a:b]`, `[-1]`,
`[::2]` · `len()` · `.split()` · `.strip()` · `.replace()` · `.upper()` · lists ·
`.append()` · `in`

**All present in the lecture** (verified by grep against both documents). The lab exercises
all of them except `.upper()`, `[::2]`, and `!=`, which appear in lecture demos only.

## Instructor TODO

- **Timing is untested.** 18 slides plus two checkpoints for a 75-minute session. If the
  room runs long, the compressible slide is `.upper()` / string methods — the lab covers
  `.strip()` and `.replace()` in Exercise 8 regardless.
- **Run both checkpoints broken on the projector.** The silent one is the more valuable of
  the two, and it only teaches if students see it print a plausible wrong-reasoned answer
  before the bug is named.
- **Switch the silent checkpoint to Butler County live** (43 deaths, population 198,413,
  rate 21.7) to make the bug actually fire. The notes say so.
- The lab has 12 exercises for a 75-minute session. **Exercises 11 and 12 are the
  synthesis** — if time is short, they are the ones worth protecting, and 6–7 (indexing
  and slicing) are the ones to cut.
