# Lecture finalization — running notes

One pass per module. Findings are **recorded here, not acted on**, unless the
entry says APPLIED. Verified against the real data files at the date shown.

---

## Module 0 — Notebooks and Values

Audited 2026-08-23. Deck: `module0/class/module0_lecture.qmd` (1220 lines, 89 slides).

**Verified correct** — no action needed. Checked against
`data/pa_overdose_county_year.csv`:

- 4,703 deaths in 2023; 402 rows; 67 counties
- State rate 36.7 per 100,000 (36.74 excluding Montour, 36.84 including)
- Schuylkill 60 deaths / 143,786 → 41.7 · Centre 7 / 157,795 → 4.4
- Forest 0.0 · Philadelphia 84.8 (the two labelled extremes on Figure 1)
- 7 counties suppressed in 2023, 60 reporting
- All seven arithmetic results: `4703+67=4770`, `//=70`, `%=13`, `2**10=1024`
- `60/143786*100000 = 41.728680121847745` (the "excessive precision" slide)

### OPEN 0.1 — Figure 2 hardcodes "9.5x" but the ratio is 9.4

`module0_lecture.qmd:1121` sets the span label to `"9.5x"`.

- From raw values: 41.7287 / 4.4361 = **9.4065**
- From published `rate_per_100k`: 41.70 / 4.40 = **9.4773**

Neither rounds to 9.5. Worse, the *previous* slide (line 1059) computes
`{schuylkill_rate / centre_rate:.1f}` in a live cell in front of the room and
will print **9.4**. A student who compares the two sees the deck contradict
itself.

The prose "ninefold" (line 1146) and the section heading are fine either way.

**Fix:** change the label to `9.4x`, or compute it inline so it cannot drift.

### OPEN 0.2 — Montour is silently excluded from both figures

Lines 79 and 1085 drop Montour County. The reason is sound: 20 deaths in a
population of 17,860 is **112.0 per 100,000**, a third higher than
Philadelphia, driven by a small denominator and a regional hospital that
receives cases from surrounding counties.

But Figure 2's caption says "the same 59 Pennsylvania counties" with no
explanation of why not 60, and the data slide (line 136) promises that
**every limitation is stated**. This one is not.

**Fix:** one line in the figure caption or the data table.

### OPEN 0.3 — Suppression stated two ways, never reconciled

Line 146 says "Seven counties are blank in 2023" (correct). The figures
work from the 60 reporting counties, and `module0_lab.ipynb` checks 7 CSVs.
All true; never tied together on screen. Low priority.

### APPLIED 0.4 — Module 0 lab missing from the schedule table

Slide 7's schedule showed `—` in Module 0's Lab column, but students now
run `module0_lab.ipynb` after class. Added, marked ungraded.
See the entry in this file's Applied log.

### OPEN 0.5 — "Six per module" labs count

Assessment slide (line 224) says labs are "Six per module". With Lab 0
there are seven lab notebooks. If Lab 0 stays ungraded the 40% weight is
unaffected, but the wording is now imprecise.

### OPEN 0.6 — Two orphaned CSVs ship to students

`data/pa_air_quality.csv` and `data/pa_hiv_county_year.csv` are referenced
by **no deck and no lab** (verified by grep across all seven modules). They
are in the zip and in `module0_lab.ipynb`'s expected-file list only by
omission — the check tests 7 files, not 9, which is correct as written.

**Decide:** drop them from the distribution, or keep for the final project.

### NOT A DEFECT — `int(41.7)` on the conversion slide

Line 841 shows `int(41.7)` → 41. An earlier session note recorded a change
to `int(70.2)`; the slide still shows 41.7 and that is arguably better,
since 41.7 is Schuylkill's own rate and the number is already in the room.
The speaker note offers 70.2 as a second example. Both correct.

### Syllabus cross-check — 2026-08-23

Deck slide 7 checked line by line against
`shared/20260818_PUBHLT0411-syllabus-RD.docx`.

**Matches exactly:** all seven lecture dates, all seven lab dates, assessment
weights (40/25/30/5), final-project structure (60/40, sessions 12/1 and 12/3,
packet due 11:59pm 12/7, peer attendance ungraded), and the readings on
`index.qmd` (P4E 1–2, P4E 3/5/8 + P4DA 2, …).

#### RESOLVED S.1 — Homework dates fixed in the Word document (2026-08-23)

Instructor updated the syllabus at 17:20. Re-checked: all five Python HW
dates now read 10/15, 10/22, 10/29, 11/10, 11/17 — each satisfying the
9:00am rule, and each matching the deck. No Wednesdays remain. Original
finding below for the record.

**Instructor confirmed 2026-08-23:** the rule is 9:00am on the day the next
module begins. The prose and the deck both state this correctly; only the
schedule **table** is wrong, by one class day in five places. Three of its
dates (10/14, 10/21, 10/28) fall on **Wednesdays**, when the class does not
meet — independent evidence the table drifted, not the prose.

| HW | Table says | Prose rule implies | Deck says |
|---|---|---|---|
| 1 | 10/14 | 10/15 | Thu 10/15 |
| 2 | 10/21 | 10/22 | Thu 10/22 |
| 3 | 10/28 | 10/29 | Thu 10/29 |
| 4 | 11/9  | 11/10 | Tue 11/10 |
| 5 | 11/16 | 11/17 | Tue 11/17 |

Fix belongs in the Word document — move each table date forward one class
day. No deck changes needed.

**The rule is course-wide, and the R half already follows it.** Checked
2026-08-23: the syllabus states the 9:00am rule once, in the Homework
section, with no distinction between halves. R HW 2–5 each sit in the row
for the lecture that opens the next module (9/8, 9/15, 9/22, 9/29) — correct.

R HW 1 is the single exception, listed Thu 9/3 when R Module 2 began Tue 9/1.
Reads as a deliberate concession rather than an error: R Module 1 is a
combined Lecture+Lab on 8/27, two days after Module 0, so the strict rule
would give students three days total from the first real content. 9/3 also
pairs it with R Lab 2.

Score: **R 4/5 follow the rule, Python 0/5.** The Python table being
uniformly one day early is the signature of a copy-forward that was never
adjusted, not a considered exception.

R labs are all internally consistent — each due on its own lab session date.

#### OPEN S.2 — Syllabus says Module 0 has "No HW, no LAB"

Now in tension with APPLIED 0.4, which adds an ungraded Module 0 lab to the
deck. Compatible in spirit — nothing is submitted — but the syllabus states
flatly that there is no lab. Consider "no graded lab" in the Word document.

#### OPEN S.3 — Two syllabus dates appear in no Python deck

- **11/3 — final project topic selections due.** Carries a 5% project penalty
  for lateness. Falls inside the Python half; no deck mentions it.
- **11/19 — no class, Final Project Advancement.**

Module 6 (11/17) is the natural place for both, since it is the last teaching
session before the project sessions.

#### RESOLVED S.2 — Module 0 lab (2026-08-23)

The "No HW, no LAB" cell is now empty in the updated document. No longer
contradicts the ungraded setup check on deck slide 7. Optionally the cell
could say "No HW; setup check only, not graded" — but silence does not
conflict.

#### RESOLVED S.4 — Anaconda and Colab removed from the syllabus (2026-08-23)

Re-checked: neither word appears anywhere in the document. The Required
Software paragraph now names uv and describes the Canvas download.

`index.qmd:63` in THIS repository still carries the stale Colab claim —
now the last one anywhere in the course. Separate edit, still open.

#### Original finding S.4 — Syllabus still recommends Anaconda and Google Colab

Required Software says *"For Python, the use of the Anaconda distribution is
recommended… We will be exploring the use of Google Colab to run python
scripts as well."*

Module 0 now installs Python via `uv` locally and states that Anaconda is
**not** required. Same stale claim as `index.qmd:63`.

**Replacement text supplied to the instructor 2026-08-23** for the Word
document's "Required Software" paragraph: names uv, states that a separate
Python download is not required, that Anaconda is neither required nor
recommended, that an existing install is left untouched, and that Colab is
not used. R sentences unchanged.

`index.qmd:63` still carries the same stale Colab claim and is a separate
edit, in this repository.

---

## Applied log

| Date | Module | Change |
|---|---|---|
| 2026-08-23 | 0 | Added Module 0's lab to the slide 7 schedule, marked ungraded (0.4) |
