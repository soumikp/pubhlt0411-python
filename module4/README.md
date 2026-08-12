# Module 4 — pandas and wrangling

**PUBHLT 0411 · Python for Public Health Data Analysis**
**Lecture:** Thu 10/29 · **Lab:** Thu 11/5
**Home story:** S4 — maternal and infant health, Pennsylvania counties (⭐ home cell)
**Packages required:** `numpy`, `pandas` (`packages: [numpy, pandas]`)

## One skill

**Load a real table and reshape the question.**

**This is the pivotal module. It carries the R half's three wrangling modules**, and it is
the densest document in the course.

## File manifest

```
module4/
├── class/
│   ├── module4_lecture.qmd       # lecture slides (live-revealjs)
│   ├── module4_lecture.html
│   ├── pitt-theme.scss
│   ├── data/                     # VFS copies — required, see below
│   │   ├── pa_maternal_infant.csv
│   │   ├── pa_counties.csv
│   │   ├── pa_overdose_county_year.csv
│   │   ├── pa_covid_monthly.csv
│   │   └── pa_firearm_county_year.csv
│   └── _extensions/r-wasm/live/
├── lab/
│   ├── module4_lab.qmd           # Py Lab 4 (live-revealjs, smaller: true)
│   ├── module4_lab.html
│   ├── pitt-theme.scss
│   ├── data/                     # same five files
│   └── _extensions/r-wasm/live/
└── README.md
```

**Five CSVs, both locations.** All five are listed under `pyodide: resources:` in each
`.qmd`. Per §7, a CSV dropped in `data/` that is *not* listed there never reaches the
browser, and the lecture needs its **own** `class/data/` copy. All ten copies verified
byte-identical to `data/` by MD5.

**VFS manifest verified after rendering** by base64-decoding the `<script type="vfs-file">`
block in both HTML files. Each lists all five paths. Note the manifest is a JSON **array of
paths**, not the objects-with-payload structure one might assume.

## Rendering

```bash
cd class && quarto render module4_lecture.qmd --to live-revealjs
cd lab   && quarto render module4_lab.qmd     --to live-revealjs
```

## ⚠️ The calendar problem — design around it

**HANDOFF §3 fact 1: M4's lab is six days after its lecture, across Election Day** (11/3 is
a non-teaching Final Project Advancement session). **The densest module has the longest
lecture-to-lab gap**, and the spec requires that "M4's lab must restate more than any other."

**How this build satisfies that:** every one of the 16 lab exercises **restates its construct
in a worked example before the task uses it.** The lab's opening carries a callout telling
students that returning cold is expected and that rereading the slides first is unnecessary.
**This redundancy is deliberate — do not compress it in a future edit.**

The Tue/Thu rhythm also inverts at M4 (lecture Thu, lab Thu) and again at M5. §4 requires
this be stated in M0's course map so students do not track it themselves.

## The cold open and its payoff

**Open:** In twenty-three Pennsylvania counties no infant mortality rate is published. Not
because it is secret — because it is unstable. Which counties, and what do they share?

**Payoff:** **Twenty-three counties, eighteen of them nonmetro.** Median population **37,607**
against **157,825** for the counties that report. The cross-tab is metro 32/5 against nonmetro
12/18 — **sixty percent of nonmetro counties are suppressed, against 13.5% of metro counties.**

The mechanism, stated on the payoff slide: a county with ~400 births and two infant deaths
reports 5.0 per 1,000; **one more death moves it to 7.5.** Publishing that invites a
comparison it cannot support.

**The framing that matters more than the number:** suppression is not the absence of
information. That two-thirds of rural Pennsylvania counties cannot support a stable infant
mortality rate *is itself a finding*.

## ⚠️ §5 rule 4 — the framing is structural

**The maternal/infant disparity is framed structurally, never as individual choice.** The
notes name the mechanism as small populations, distance to obstetric care, and closed
maternity wards — infrastructure, not behaviour. Two lab callouts (Exercises 5 and 7)
restate that these are statements about **counties**, not about families.

The room takes its cue from how the instructor says it.

## ⚠️ §6 trap 3 — the silent error, taught deliberately

**Suppression biases grouped rates downward.** `.sum()` skips `NaN`, but the population
denominator still counts every county, so a suppressed county contributes **population and no
deaths**. No exception is raised.

Verified 2023 overdose rates by region — this build reproduces §6's table exactly:

| Region | Naive | Correct | Bias |
|---|---|---|---|
| Southwest | 42.9 | 42.9 | 0.0 |
| Southeast | 41.3 | 41.3 | 0.0 |
| Northeast | 35.4 | 36.4 | −1.0 |
| South Central | 22.3 | 23.0 | −0.7 |
| Northwest | 37.5 | 39.5 | **−2.0** |
| North Central | 20.3 | 22.1 | **−1.8** |

**The bias is largest in rural regions, exactly where suppression concentrates.** It appears
twice: as a lecture slide with the full table, and as lab Exercise 12, where students compute
the Northwest bias themselves.

**The rule taught:** *whenever a ratio is computed over a group, confirm the numerator and the
denominator cover the same rows.*

## The "Mc Kean" join bug — reconstructed, and why

**§4 and §6 require teaching the real "Mc Kean"/"McKean" mismatch. The shipped
`data/` files no longer contain it** — every file writes `McKean`, 65 occurrences, zero
`Mc Kean`. It was cleaned upstream before the CSVs landed in this repo.

**Resolution:** rather than fabricate a mismatch in the course data or silently drop the
concept, both documents **reconstruct the original spelling in a small literal DataFrame**
and join it against the real `pa_counties.csv`. The lecture slide states in a code comment
that County Health Rankings publishes "Mc Kean" while the Census writes "McKean"; the lab's
Exercise 14 repairs it with `.str.replace(" ", "")`.

This keeps the lesson real and honest about its provenance. **Do not "fix" this by editing
`pa_counties.csv` to reintroduce the bug** — the join exercises elsewhere depend on clean keys.

## Verification status

**Lecture — 29 `{pyodide}` cells; 28 run clean, 1 raises by design.**

```bash
cd class && python3 ../../tools/run_cells.py module4_lecture.qmd
```

The single `RAISE` is the deliberately-broken `KeyError` slide, and it raises **exactly** the
message the callout quotes:

```
KeyError: 'infant_mortality'
```

**The deliberately-broken cell was executed, per §7's verification standard** — confirmed,
not assumed. §7 records two v1 cases where an assumed error did not fire.

**Lab — all 16 exercises pass, and all 16 checks discriminate.**

```bash
cd lab && python3 ../../tools/verify_lab.py module4_lab.qmd
```

55 asserts across 16 exercises. `discriminates` means each check **fails** against setup-only
with no student work, per §7. Zero `TRIVIAL-PASS`.

> **Note on `run_cells.py` against the lab:** it reports `NameError` for all 16 `#| check:`
> cells. This is expected and not a defect — checks reference student variables and are run
> by `verify_lab.py` in a shared namespace, which is the correct tool. Every worked example
> and solution cell runs clean.

### Two errors caught by running the code

**Both were assumptions that execution corrected**, which is the §7 standard working:

1. **Lab Exercise 6 asserted Allegheny's 2022 firearm rate as 191 deaths / 15.59 per
   100,000.** The real values are **187 deaths / 15.27**. Caught by `verify_lab.py`; the
   assert and the hint were corrected.
2. **The lecture's `.str.title()` note claimed it would turn "McKean" into "Mckean".**
   Run against the actual input, `.str.title()` leaves `"Mc Kean"` unchanged — it capitalises
   after a space, so the space removal is what repairs the key. The note was rewritten to
   state what actually happens, and to give `"MCKEAN"` → `"Mckean"` as the case where it does
   misbehave.

Two smaller corrections: `fillna(0)` mean is **4.04**, not 4.03; reporting-county median
population is **157,825**, not 157,824.

### Every number verified against the real files

| Fact | Value |
|---|---|
| Counties with no published infant mortality rate | **23 of 67** |
| Of those, nonmetro | **18** |
| Cross-tab metro (not suppressed / suppressed) | 32 / 5 |
| Cross-tab nonmetro | 12 / 18 |
| Median population, suppressed vs reporting | **37,607** vs **157,825** |
| Infant mortality, 44 reporting counties | mean 6.15, median 6.0, range 3.4–10.2 |
| Same column filled with zero | **4.04** — a false claim about 23 counties |
| Child poverty × infant mortality | **+0.646** (§6 confirmed) |
| Median income × infant mortality | **−0.602** (§6 confirmed) |
| Other gaps in the file | child mortality 7, teen births 1 |
| Counties above 20% child poverty | 11 (Philadelphia highest, 28.3%) |
| Poverty > 20% **and** infant mortality > 7 | 4 — Fayette, Mifflin, Philadelphia, Venango |
| Highest infant mortality | Mifflin, 10.2 per 1,000 |
| Top five infant mortality | all nonmetro |
| Low birthweight | mean 7.72%, max 11.0%, 9 counties above 9% |
| Highest teen births | Fayette, 25.4 per 1,000 |
| Uninsured > 6% **and** poverty > 15% | 7 counties |
| Low birthweight by region | Northwest 8.44% high, North Central 6.94% low |
| Median household income, metro vs nonmetro | 70,751 vs 57,978 |
| COVID file | **49 months**, Mar 2020 – Mar 2024 |
| COVID peak month | **Jan 2022, 4,294 deaths** (Omicron) — §5 confirmed |
| COVID yearly totals | 975 / 10,875 / 11,109 / 3,647 / 916 |
| Allegheny 2022 firearm | 187 deaths, **15.27** per 100,000 |
| Firearm 2023 suppression | metro 6/37, nonmetro **19/30** |
| 2023 overdose, counties reporting | **60 of 67** |
| 2023 overdose by region, mean rate | Southwest 36.0 high, South Central 20.7 low |
| Regions / metro split | 6 regions · 37 metro, 30 nonmetro |

**Every lecture cell is self-contained.** `run_cells.py` executes each in a fresh namespace,
which is what caught M2's cross-cell `NameError`. Every cell re-imports pandas and re-reads
its data, so a student jumping to any single cell in the browser gets a working cell.

## A note on the §5 firearm cell

§5 assigns M4 the firearm story as a **callback** (`.value_counts()`; `crosstab()`), and it is
used that way: lab Exercise 11 cross-tabulates 2023 firearm suppression against metro status,
and the lecture uses the firearm file for the join-type slide.

**During verification, the naive metro/nonmetro firearm rate came out reversed from §5's
`nonmetro 18.5 > metro 14.1`.** This is not a data error — it is §6 trap 3 firing on a second
outcome. Nineteen of thirty nonmetro counties are suppressed for 2023, so the naive
computation gives nonmetro 8.2 against metro 13.6. **Dropping suppressed counties from both
numerator and denominator reproduces §5 exactly: nonmetro 18.5, metro 14.1.**

**This is M6's payoff, and M4 does not spend it.** M4's job is the `.merge()` that makes it
computable and the suppression discipline that makes it correct. Recorded here because the
reversal will look like a bug to anyone who recomputes it naively.

## Slide structure

**One *task* per slide** (§2 labs clause, amended 2026-08-11). The worked example and the
`**Your turn.**` task are separate slides sharing a heading, the task suffixed `— your turn`;
a trailing teaching callout stays on the task slide. Exercise 14 ("Repair a broken key") has
no worked example — it is the run-it-broken exercise — so it remains a single slide, following
M3's Exercise 13 pattern. **15 of the 16 exercises are three-slide units: divider, example,
task.**

Lecture: 30 content slides, 2 section dividers, **30 notes blocks / 4,372 words**.

## Teaching devices

- **`.info()` finds the answer to the cold open in the second minute.** The non-null count
  reads 44 where every other column reads 67, and the missing 23 *are* the question. This is
  deliberate sequencing: the tool discovers the finding before the finding is discussed.
- **`missing ≠ zero` gets a full slide with the arithmetic on it.** 6.15 skipping missing
  against 4.04 filling with zero. The notes name this as the third appearance of one rule —
  M2's `.get(key, 0)`, M3's `np.nanmean()`, and now `.fillna()` — and call it the most
  important non-syntactic idea in the course.
- **`&`/`|` with parentheses appears three times by design**: a lecture callout with the
  `TypeError` explanation, lab Exercise 5 (which blanks *both* operators), and the §4-mandated
  check-in framing. It is the most common pandas error after `KeyError`, and §4 explicitly
  asks that it be a check-in.
- **The suppression-bias slide is the module's most important**, and it is a *silent* error —
  no exception, plausible-looking output. It gets the longest note in the document.
- **`.assign()` is taught as a data check, not as syntax.** Recomputing a published rate
  column and confirming it matches is the habit; a mismatch means the denominator or the
  published rate uses different assumptions. Lab Exercise 6 does the same against the firearm
  file.
- **Joins are taught with the row-count check attached.** Every join slide prints shape before
  and after. The notes state that `inner` is the dangerous default because it deletes
  unmatched rows silently, and recommend `left` as the working default.
- **A failed join key is distinguished from missing data.** `NaN` in a borrowed column has two
  possible causes needing different fixes, and they look identical in the output.
- **One fill-in-the-blank live demo** (`m4_suppressed`), blank on `how=`. `inner` gives an
  identical answer here and is still wrong — the point is that the join should express the
  intent and survive a data update. That argument is the reason the blank is `how=` and not
  something with a distinguishable output.

## Data used

**Lecture.** `pa_maternal_infant.csv` is the spine. `pa_counties.csv` is the join target.
`pa_overdose_county_year.csv` carries the `.groupby()` and suppression-bias material,
`pa_covid_monthly.csv` the `to_datetime()`/`.dt` slide, `pa_firearm_county_year.csv` the
join-type slide. The `.melt()` and `.str` slides use small literal DataFrames, because both
need a *wide* and a *dirty* input respectively and no course file is either.

**Lab.** All five files. Exercises 1–7 and 15 use the births file; 8, 9, 12, and 16 the
overdose file; 6 and 11 the firearm file; 13 the COVID file; 9–12 and 14 the county reference.

**Callbacks, not re-taught** (§5 rule 2): the Omicron peak returns from M2 with a real date
column under it; Schuylkill and Centre return from M1 in the `.isin()` slide; the four
recurring counties keep their identity from M1 onward. Suppression was *named* in M3 and is
**taught** here, as §5 specifies.

## Voice sheet compliance (§2, amended 2026-08-11)

| Surface | Second person | Result |
|---|---|---|
| Lecture slides | 0 instances | clean |
| Lecture notes | 30 blocks, 4,372 words | load-bearing |
| Lab prose, callouts, explanations | 0 instances | clean |
| Lab `**Your turn.**` instructions | 15 instances | expected per the labs clause |
| Lab `— your turn` slide headings | 15 | §7-mandated suffix |

Audited with code cells and `.notes` blocks excluded from the slide count, per §2's labs
clause. **No litotes in either document.** All callouts carry titles.

## Concept coverage (§4)

Series · DataFrame · Index · `pd.read_csv()` · `.head()` · `.info()` · `.describe()` ·
`.shape` · `.dtypes` · `.loc` / `.iloc` · boolean filtering · **`&`/`|` with parentheses** ·
`.isin()` · `.assign()` · `.sort_values()` · **`.groupby()` / `.agg()`** · `.value_counts()` ·
`pd.crosstab()` · `.isna()` · `.dropna()` · `.fillna()` · **missing ≠ zero** · **`.merge()`**
inner/left/outer + key mismatch · `.melt()` / `.pivot()` · `.rename()` / `.drop()` /
`.astype()` · chaining · `.apply()` · `.str` cleaning · `pd.to_datetime()` / `.dt`

**All present in the lecture** (verified by grep against both documents). `.rename()` and
`.drop()` were added on a coverage re-audit after the first draft omitted them.

The lab exercises everything except `.melt()`/`.pivot()`, `.apply()`, `.fillna()`,
`.value_counts()`, `.isin()`, `.info()`, and `.describe()`, which are lecture-only. `.query()`
is taught in both and is additional to §4 — it appears **after** the parentheses rule, not
before it, so it reads as a shortcut rather than a way to avoid learning the rule.

## Accessibility (WCAG 2.2 AA)

- **No figures in M4** — text, tables, and code only. Visualisation is M5.
- All callouts carry titles (verified: zero untitled).
- Inherits the audited `pitt-theme.scss`. No new colors.
- **Run `PREPUBLISH_CHECKLIST.md`** before publishing.

## Instructor TODO

- **Test the VFS on classroom wifi before 10/29.** Five files here against M3's two, and the
  failure mode in the room is an unhelpful file-not-found rather than an obvious network error.
- **Run the `KeyError` and the unparenthesised `&` broken on the projector.** Both are thirty
  seconds and both are the errors students will actually hit.
- **Protect the suppression-bias slide if time runs short.** It is the module's most important
  content and it is two-thirds of the way in, which is exactly where a dense lecture loses its
  tail. Cut the `.melt()`/`.pivot()` slide first — it is the least load-bearing, and M5 and M6
  both revisit reshaping.
- **The lab is 16 exercises and is deliberately over-provisioned** for the six-day gap.
  Exercises 12 (suppression bias) and 16 (the chain) are the synthesis — **protect those.**
  Exercise 1 (`read_csv` + `.shape`) is the one to cut for time, then Exercise 3.
- **Watch the framing in discussion.** §5 rule 4 is structural, not individual. If the room
  drifts, the correct redirect is toward closed maternity wards and distance to care.
- Consider whether §5's firearm row for M4 should be annotated with the suppression caveat
  recorded above, so M6's build does not rediscover the reversal.
