# Module 3 — NumPy

**PUBHLT 0411 · Python for Public Health Data Analysis**
**Lecture:** Thu 10/22 · **Lab:** Tue 10/27
**Home story:** S3 — PM2.5 air pollution, Pennsylvania counties (⭐ home cell)
**Packages required:** `numpy` (`packages: [numpy]`)

## One skill

**Replace loops with whole-array arithmetic, then select with masks.**

## File manifest

```
module3/
├── class/
│   ├── module3_lecture.qmd       # lecture slides (live-revealjs)
│   ├── module3_lecture.html
│   ├── pitt-theme.scss
│   ├── data/                     # VFS copies — required, see below
│   │   ├── pa_air_pollution.csv
│   │   └── pa_overdose_county_year.csv
│   └── _extensions/r-wasm/live/
├── lab/
│   ├── module3_lab.qmd           # Py Lab 3 (live-revealjs, smaller: true)
│   ├── module3_lab.html
│   ├── pitt-theme.scss
│   ├── data/                     # same two files
│   └── _extensions/r-wasm/live/
└── README.md
```

**This is the first module to use the Pyodide VFS.** Both CSVs are listed under
`pyodide: resources:` in each `.qmd`. Per §7, a CSV dropped in `data/` that is *not* listed
there never reaches the browser, and the lecture needs its **own** `class/data/` copy — a
`data/` directory at the project root is not visible to it. Both copies are byte-identical
to `data/`.

## Rendering

```bash
cd class && quarto render module3_lecture.qmd --to live-revealjs
cd lab   && quarto render module3_lab.qmd     --to live-revealjs
```

## The cold open and its payoff

**Open:** The EPA annual PM2.5 standard is 9.0 µg/m³. Twenty-two of Pennsylvania's 67
counties exceed it. One exceeds it by a wide margin. The magnitude is withheld until the end.

**Payoff:** **Allegheny County, 14.1 µg/m³, z = +4.37** — the state's only extreme outlier.

The teaching point is the *shape* of the distribution, not the single value. Remove
Allegheny and the counties run 5.5 to 11.1, an ordinary spread. Twenty-one of the 22
exceedances cluster between 9.1 and 11.1. **The gap between Allegheny and second place
(Lancaster, 11.1) exceeds the gap between second place and the state mean.** Allegheny is
not the top of a gradient; it is a separate case.

**Ask the room which county before revealing anything.** They will say Allegheny and they
will be right — most of them live there. The magnitude is what nobody predicts.

## ⚠️ The PM2.5 trap — read before teaching

**§6 trap 2: never use PM2.5 for a disparity claim.** In Pennsylvania, child poverty × PM2.5
= **−0.421**, and **stratifying by urbanicity does not reverse it** — the correlation is
negative inside every stratum. PM2.5 here is an industrial and density exposure, and the
poorest counties are rural with clean air. County averages cannot demonstrate environmental
justice, which operates at neighbourhood scale.

This module treats PM2.5 as **a regulatory threshold story only**. The lecture notes
instruct the instructor to block the drift explicitly, and the payoff slide's notes name the
resolution limit out loud: the Mon Valley is not Sewickley, and a county-wide annual average
cannot speak to either.

## The z-score discrepancy — resolved

**HANDOFF §4, §5, and §6 all record Allegheny's z as +4.34. The documents teach +4.37, and
+4.37 is what the taught code prints.**

The handoff figure was computed with `ddof=1` (the sample standard deviation, σ = 1.2929).
§4 specifies `.std()` as the method to teach, and **NumPy's `.std()` defaults to `ddof=0`**
(the population standard deviation, σ = 1.2832), which yields **+4.372**.

Teaching `.std()` and then printing 4.34 would be an error a student running the cell would
catch. The lecture therefore uses +4.37 throughout and **makes the discrepancy itself a
teaching point** — a callout states that NumPy's default divides by *n* while R's `sd()` and
most statistics courses use *n* − 1, and names `ddof=1` as the sample version. This matters
because the course runs alongside a statistics sequence.

**The handoff's +4.34 should be corrected to +4.37 in §4, §5, and §6, or annotated with the
`ddof` it assumes.** Left as-is it will resurface as a contradiction in M5.

## Verification status

**Lecture — 18 `{pyodide}` cells run; 0 raise.**

```bash
cd class && python3 ../../tools/run_cells.py module3_lecture.qmd
```

All 15 runnable cells return `OK`. The `m3_zscore` blank is skipped as unrunnable; its
solution prints `Highest PM2.5: Allegheny / concentration: 14.1 / z-score: +4.37`.

**Lab — all 13 exercises pass, and all 13 checks discriminate.**

```bash
cd lab && python3 ../../tools/verify_lab.py module3_lab.qmd
```

36 asserts across 13 exercises. `discriminates` means each check **fails** against
setup-only, per §7.

**Slide structure: one *task* per slide** (§2 labs clause, amended 2026-08-11). The worked
example and the `**Your turn.**` task are separate slides sharing a heading, the task
suffixed `— your turn`; a trailing teaching callout stays on the task slide. Exercise 13
("Find the error") has no worked example, so it remains a single slide. 12 of the 13
exercises are therefore three-slide units: divider, example, task.

**The deliberately-broken cell was executed, per §7's verification standard.** Exercise 13's
`pm25 > 9.0 and pm25 < 11.0` raises the exact message the callout quotes:

```
ValueError: The truth value of an array with more than one element is ambiguous.
Use a.any() or a.all()
```

This is a **loud** failure, so the run-it-broken lesson holds. §7 records two v1 cases where
an assumed error did not fire; this one was confirmed rather than assumed.

**Every number was verified against the real files.** `pa_air_pollution.csv` (67 rows) and
`pa_overdose_county_year.csv` (402 rows): mean 8.4896, median 8.50, σ 1.2832, 22 counties
over 9.0 (32.8%), mean above 9.80 / below 7.85, 20 counties in (9.0, 11.0). Zero mismatches.

**Every lecture cell is self-contained.** `run_cells.py` executes each in a fresh namespace,
which is what caught M2's cross-cell `NameError`. Every cell here re-imports NumPy and
re-reads or re-declares its data, so a student jumping to any single cell in the browser
gets a working cell.

## Teaching devices

- **The views-versus-copies slide is the module's real bug source.** `pm25[:3]` is a *view*;
  writing through it mutates the original. The identical syntax on a list produces a copy.
  The failure is silent — a value changes in an array the code never appears to touch. The
  notes give the one-character fix (`.copy()`) and the asymmetry worth knowing: **slices are
  views, boolean masks are copies.**
- **`axis=` gets the module's longest note** because it is the hardest idea here. The rule
  taught is *`axis=` names the dimension removed, not the one kept*, with the shape check as
  the diagnostic: from `(4, 5)`, `axis=1` removes the 5 and leaves 4 numbers. The wrong axis
  usually still returns an array and stays silent.
- **`&` versus `and`** appears twice by design — as a slide callout during masking, and as
  the Exercise 13 debug. It is the most common NumPy error and it returns in M4 with pandas,
  where the parentheses trip people harder.
- **One fill-in-the-blank live demo** (`m3_zscore`), blank on `.mean()`. It introduces
  `.argmax()`, and the notes draw the `.max()`/`.argmax()` distinction — value versus
  position, where the position is what indexes the names array.
- **Parallel arrays are taught as a liability, not a technique.** `counties[12]` and
  `pm25[12]` describe the same county only by convention; nothing enforces it, and sorting
  one without the other is silently wrong. A lab callout after Exercise 3 states this
  directly. **This is the honest motivation for the DataFrame in M4**, and the two clumsy
  `genfromtxt` calls are left visibly clumsy on purpose.
- **Missing data is framed as a judgment, not a syntax.** `np.nanmean()` describes the
  counties that reported; suppression concentrates in small rural counties, so the omitted
  counties are not a random sample. Both documents require the denominator in the writeup.
  This continues M2's `.get(key, 0)` thread and seeds M4's NaN material.

## Data used

**Lecture.** `pa_air_pollution.csv` read live with `np.genfromtxt()` — the first real file
of the course, and worth marking as a milestone. The 2-D `axis=` material uses a 4×5 array
of overdose rates (Allegheny, Philadelphia, Schuylkill, Centre × 2019–2023) transcribed from
`pa_overdose_county_year.csv`. The `np.nan` slides use a 7-element array reflecting the real
2023 suppressions.

**Lab.** Both files, read from the VFS. Exercises 3–9 and 13 read `pa_air_pollution.csv`
live; 10–11 use the 4×5 overdose array; 12 uses a 15-element array with four suppressions.

**Callbacks, not re-taught:** Schuylkill and Centre return from M1/M2 with their real 2023
overdose rates (41.7 vs 4.4 — the S1 payoff). Philadelphia and Allegheny return from M2.
Suppression is named here and **taught** in M4.

## Real-data facts embedded in the documents

| Fact | Value |
|---|---|
| Counties over the 9.0 standard | 22 of 67 (32.8%) |
| Allegheny PM2.5 / z-score | 14.1 / **+4.37** |
| Next highest (Lancaster) | 11.1 (z = +2.03) |
| State mean / median / σ | 8.49 / 8.50 / 1.28 |
| Mean above vs below standard | 9.80 vs 7.85 |
| Over 9.0 but under 11.0 | 20 counties |
| Philadelphia PM2.5 | 8.7 — **under** the standard, z = +0.16 |
| 2023 overdose suppressions | 7 of 67 counties |
| 2023 overdose mean / median (60 reporting) | 30.1 / 26.8 — right-skewed |

**Philadelphia being under the standard is a deliberate detail.** It separates the two
questions a z-score conflates: how does this county compare to its neighbours, and does it
meet the rule.

## Voice sheet compliance (§2, amended 2026-08-11)

| Surface | Second person | Result |
|---|---|---|
| Lecture slides | 0 instances | clean |
| Lecture notes | 19 blocks, 2,199 words | load-bearing |
| Lab prose, callouts, explanations | 0 instances | clean |
| Lab `**Your turn.**` instructions | 12 instances | expected per the labs clause |

Audited with code-cells and `.notes` blocks excluded from the slide count, per §2's
labs clause. The only second-person match in the lecture source is the YAML comment stating
the rule.

## Accessibility (WCAG 2.2 AA)

- **No figures in M3** — text, tables, and code only. Visualisation is M5.
- All callouts carry titles.
- Inherits the audited `pitt-theme.scss`. No new colors.
- **Run `PREPUBLISH_CHECKLIST.md`** before publishing.

## Concept coverage (§4)

Arrays vs lists and **why** · `np.array()` · `dtype` · `.shape` · `.size` · indexing ·
slicing · 2-D `[row, col]` · vectorised arithmetic · `.mean()` · `.std()` · `.median()` ·
`np.percentile()` · **`axis=`** · **boolean masking** · `np.where()` · `np.nan` ·
`np.nanmean()` · `np.random` + **seed** · **views vs copies**

**All present in the lecture** (verified by grep against both documents). The lab exercises
everything except `np.percentile()`, `np.random`/seeding, and views-vs-copies, which are
lecture-only. `np.genfromtxt()` is taught in both and is additional to §4 — it is what makes
"all-real data" possible in a numpy-only module, one week before pandas.

## Instructor TODO

- **Correct the handoff's +4.34 to +4.37** (§4, §5, §6), or annotate it with `ddof=1`. See
  the discrepancy section above. Left alone it will contradict M5.
- **Test the VFS on classroom wifi before 10/22.** This is the first module whose cells fail
  without a successful resource fetch, and the failure mode in the room is an unhelpful
  file-not-found rather than an obvious network error.
- **Demonstrate the view-mutation bug live.** Running it, then adding `.copy()` and running
  again, is the whole lesson and it takes ninety seconds.
- **Block the environmental-justice drift** if the room raises it. The correct response is
  that the data cannot answer it and the county-level correlation runs the other way — not
  that the question is uninteresting. See the trap section.
- The lab has 13 exercises. **Exercises 9 and 12 are the synthesis** — protect those.
  Exercise 1 (`np.array()` from a list) is the one to cut for time.
