# Module 2 — Collections, Loops, and Functions

**PUBHLT 0411 · Python for Public Health Data Analysis**
**Lecture:** Thu 10/15 · **Lab:** Tue 10/20
**Home story:** S2 — COVID-19 deaths, Pennsylvania counties (⭐ home cell)
**Packages required:** none (`packages: []`)

## One skill

**Walk collections with loops and package work into your own functions.**

## File manifest

```
module2/
├── class/
│   ├── module2_lecture.qmd       # lecture slides (live-revealjs)
│   ├── module2_lecture.html
│   ├── pitt-theme.scss
│   └── _extensions/r-wasm/live/
├── lab/
│   ├── module2_lab.qmd           # Py Lab 2 (live-revealjs, smaller: true)
│   ├── module2_lab.html
│   ├── pitt-theme.scss
│   └── _extensions/r-wasm/live/
└── README.md
```

**No `data/` and no `pyodide: resources:`.** §4 specifies inline dicts for M2. Every value
was computed from the real `data/pa_covid_monthly.csv` (3,283 rows) and verified — see
below.

## Rendering

```bash
cd class && quarto render module2_lecture.qmd --to live-revealjs
cd lab   && quarto render module2_lab.qmd     --to live-revealjs
```

## The cold open and its payoff

**Open:** 49 months, 67 counties, 3,283 county-months. Which month was the worst? Doing it
by hand is 49 comparisons for the state.

**Payoff:** **January 2022 — 4,294 deaths**, the deadliest month of the pandemic in
Pennsylvania. Not March 2020.

The point is not the date but its lateness: the peak arrived **22 months in**, after
vaccines were available, during Omicron. Cumulative deaths across the seven-month teaching
window crossed 10,000 during that same January.

**Ask the room to guess before revealing anything.** Most say spring 2020 — the lockdown
period, the sirens, the refrigerated trucks. That guess is wrong, and being wrong for a
memorable reason is what makes the answer stick. The notes say so.

## The counts-versus-rates result

The `sorted(key=)` material carries §6's trap 1, and the real data makes it unusually
stark. Ranking all 67 counties by COVID death **count** versus by **rate**:

| Rank | By count | By rate |
|---|---|---|
| 1 | Philadelphia (4,052) | Potter (556.3) |
| 2 | Allegheny (2,179) | McKean (493.4) |
| 3 | Montgomery (1,092) | Juniata (473.3) |
| 4 | Lancaster (1,064) | Jefferson (431.1) |
| 5 | Bucks (998) | Armstrong (422.9) |

**The two lists share no members.** Counting ranks the five largest counties; the rate
ranking returns five rural counties, none above 65,000 residents.

**Potter County recorded 89 deaths against Philadelphia's 4,052, and has more than double
Philadelphia's rate** — because 16,000 people live there. Lab Exercise 9 and 11 both land
on this.

## Verification status

**Lecture — 21 `{pyodide}` cells run; 1 raises by design.**

```bash
cd class && python3 ../../tools/run_cells.py module2_lecture.qmd
```

The `KeyError` checkpoint raises as designed. The `m2_worst` blank is skipped; its solution
prints `Worst month: 2022-01 with 4,294 deaths`.

**Lab — all 13 exercises pass, and all 13 checks discriminate.**

```bash
cd lab && python3 ../../tools/verify_lab.py module2_lab.qmd
```

23 asserts across 13 exercises. `discriminates` means each check **fails** against
setup-only, per §7.

**Every number was verified against `data/pa_covid_monthly.csv`.** Nine county totals with
populations, and six months of Allegheny data: **zero mismatches.** Two authoring errors
were caught and fixed:

1. **A `NameError` in the lecture.** The `.items()` demo relied on `covid_deaths` defined in
   the previous cell. Cells run in fresh namespaces, so a student jumping straight to that
   cell in the browser would have hit the error. The dictionary is now redefined in the cell.
2. **Fayette County appeared as both 325 and 323** on consecutive slides. The real total is
   **325**.
3. **The statewide COVID rate was asserted as "about 220" in a speaker note.** The real
   figure across the 49 months is **212.3 per 100,000**. Both the note and the function
   threshold in `summarize_county()` now use 212.3.

## Teaching devices

- **Two checkpoints:**
  - **`KeyError`** — a loud, friendly error naming the missing key. Paired immediately with
    `.get(key, default)`.
  - **An indentation checkpoint that is *correct as written*.** The lesson is what
    indentation controls: moving the final `print()` inside the loop reports a partial count
    as though it were the answer, with no error. The notes tell the instructor to indent it
    live and ask which of the seven printed lines a hurried reader would copy.
- **One fill-in-the-blank live demo** (`m2_worst`), blank on `.items()`.
- **Thirteen lab exercises**, worked-example-then-your-turn **across two slides**, with hint, solution, and a
  discriminating check.
- **Exercise 13 is a `NameError` debug** — an accumulator used before initialization, which
  is the most common loop bug students write.
- **The `.get(key, 0)` judgment call is taught, not just the syntax.** A default of zero
  asserts nobody died that month. That is right when the month is outside the reporting
  period and **wrong** when nobody reported. This is the seed of M4's NaN material, and it
  appears in both the lecture notes and a lab callout.

## Objects (2 slides, per §4)

The goal is recognition, not fluency — students need to *read* a class, not design one.

- **Slide 1:** everything is an object; `"text".upper()`, `counties.append()`, and
  `covid_deaths.items()` are the same construction.
- **Slide 2:** one `County` class with `__init__`, `self`, an attribute, and a method.
- **A third slide closes the loop**, combining the class with `sorted(key=)` and a lambda
  calling a method — then names what M4 introduces: `df.head()` is the same dot.

**The missing-parentheses bug is flagged in the notes** — `schuylkill.rate_per_100k`
without `()` returns the bound method, prints something like `<bound method ...>`, and is
always truthy in an `if`. §7 records this as a real v1 error.

## Voice sheet compliance (§2, amended 2026-08-11)

| Surface | Second person | Result |
|---|---|---|
| Lecture slides | 0 instances | clean |
| Lecture notes | 21 blocks, 1,903 words | load-bearing |
| Lab prose, callouts, explanations | 0 instances | clean |
| Lab `**Your turn.**` instructions | 12 instances | expected per the labs clause |

## Accessibility (WCAG 2.2 AA)

- **No figures in M2** — text, tables, and code only.
- All callouts carry titles.
- Inherits the audited `pitt-theme.scss`. No new colors.
- **Run `PREPUBLISH_CHECKLIST.md`** before publishing.

## Concept coverage (§4)

Dicts · `d[k]` · `.get()` · `.keys()` · `.values()` · **`.items()`** · `for` · `range()` ·
**the accumulator pattern** · `while` · **`while True:` + `break`** · `continue` · `def` ·
parameters · `return` · docstrings · multiple returns · tuples + unpacking ·
`sorted(key=)` · **objects: attributes vs methods, `__init__`, `self`**

**All present in the lecture** (verified by grep against both documents). The lab exercises
everything except `while`, `while True:`, `continue`, `.keys()`, `.values()`, and
standalone tuple unpacking, which appear in lecture demos only. Exercise 10 covers
unpacking implicitly, via a two-value `return`.

## Instructor TODO

- **This is the densest module so far** — 23 slides for 75 minutes, and the outcomes list
  is long. If the room saturates, dictionaries and `.items()` are what must land; the
  object slides are the compressible ones, since M4 reintroduces the dot anyway.
- **Warn about infinite loops before the `while` slide.** In Colab a runaway loop hangs the
  notebook, and the fix is `Runtime > Interrupt execution`. Point at that menu item before
  they need it.
- **Demonstrate print-vs-return live** in the functions section. Replacing `return` with
  `print` then assigning the result is the bug that costs students the most time, and it
  fails one line away from its cause.
- The lab has 13 exercises. **Exercises 11 and 12 are the synthesis** — protect those. 7
  (`range`) is the one to cut for time.
