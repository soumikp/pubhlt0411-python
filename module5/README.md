# Module 5 — Visualisation and description

**PUBHLT 0411 · Python for Public Health Data Analysis**
**Lecture:** Tue 11/10 · **Lab:** Thu 11/12
**Home story:** S6 — adult asthma, Pennsylvania counties (⭐ home cell)
**Packages required:** `numpy`, `pandas`, `matplotlib` + **seaborn via `micropip`** *(see below)*

## One skill

**Turn a table into an honest figure, and interrogate a distribution.**

## File manifest

```
module5/
├── class/
│   ├── module5_lecture.qmd       # lecture slides (live-revealjs)
│   ├── module5_lecture.html
│   ├── pitt-theme.scss
│   ├── data/                     # VFS copies — required, see below
│   │   ├── pa_asthma_prevalence.csv
│   │   ├── pa_counties.csv
│   │   ├── pa_maternal_infant.csv
│   │   ├── pa_covid_monthly.csv
│   │   └── pa_overdose_county_year.csv
│   └── _extensions/r-wasm/live/
├── lab/
│   ├── module5_lab.qmd           # Py Lab 5 (live-revealjs, smaller: true)
│   ├── module5_lab.html
│   ├── pitt-theme.scss
│   ├── data/                     # same five files
│   └── _extensions/r-wasm/live/
└── README.md
```

**Five CSVs, both locations.** All five are listed under `pyodide: resources:` in each `.qmd`.
Per §7, a CSV dropped in `data/` that is *not* listed there never reaches the browser, and the
lecture needs its **own** `class/data/` copy. All ten copies verified byte-identical to `data/`
by MD5.

**VFS manifest verified after rendering** by base64-decoding the `<script type="vfs-file">`
block in both HTML files. Each lists all five paths.

## Rendering

```bash
cd class && quarto render module5_lecture.qmd --to live-revealjs
cd lab   && quarto render module5_lab.qmd     --to live-revealjs
```

## The cold open and its payoff

**Open:** One in ten Pennsylvania adults has asthma, and the rate barely moves between
counties — 9.7% to 11.6%, the whole state inside two percentage points. What else moves with it?

**Payoff:** **Smoking, at r = +0.659** — the strongest relationship in the course. Asthma
prevalence has a standard deviation of **0.38**; smoking's is **2.32**, six times wider. The
closing figure annotates the two ends: **Chester** (10.6% smoking, 9.7% asthma, lowest in the
state on both) and **Sullivan** (21.1% smoking, 11.3% asthma).

The lecture also pays off Module 4's closing promise in its second slide: the child poverty ×
infant mortality scatter, **+0.646** across the **44** reporting counties, with the 44 stated
in the title.

## ⚠️ §7 — this is the highest-risk module, because it is all figures

Accessibility is taught **as course content**, not bolted on. Three requirements, each with a
slide and a lab check:

1. **Alt text on every figure.** 18 `fig-alt` entries in the lecture, 15 in the lab. Every
   figure-producing cell in both documents carries one.
2. **Marks ≥ 3:1 against the background.** The locked palette, all measured with
   `scratchpad_contrast.py`:

   | Colour | Ratio on white | Role |
   |---|---|---|
   | `#003594` royal | **10.84 : 1** | primary series |
   | `#8a6400` dark gold | **5.38 : 1** | second series |
   | `#6b7280` gray | **4.83 : 1** | annotation, source notes |

3. **Colour is never the only channel.**

### ⚠️ The measurement that justifies rule 3 — do not lose this

**No two palette colours separate from *each other* by 3:1.** Royal against dark gold is
**2.02:1**. Each is perfectly legible against the page and they are *not* reliably
distinguishable from one another. This is why the second channel is a rule rather than a
preference, and the lecture states the number explicitly.

Lab **Exercise 6** enforces it with an assert that inspects marker path geometry, so a
colour-only figure fails the check. **Verified that the assert actually discriminates**: the
correct two-marker figure yields 2 distinct path shapes, the colour-only version yields 1.

### ⚠️ seaborn's default palettes all fail 3:1 — measured for this build

| Palette | Lowest ratio | Failing colours |
|---|---|---|
| `deep` (the default) | 2.80 : 1 | `#DD8452`, `#55A868` |
| `colorblind` | **2.61 : 1** | `#DE8F05`, `#CA9161` |
| `muted` | 2.01 : 1 | `#EE854A`, `#6ACC64` |
| `tab10` | 2.53 : 1 | `#FF7F0E` |

**Even the palette named `colorblind` fails**, because it solves a different problem —
keeping hues separable under colour vision deficiency, not contrast against the background.
This is a full slide in the lecture, and it is why **every seaborn call in this module passes
an explicit `palette=`.**

## ⚠️ §6 trap 1 — counts track population, taught deliberately

**COVID deaths × population = +0.946.** The lecture teaches this with a two-panel figure whose
two lists **share no counties**:

| Top 5 by total deaths | Top 5 by deaths per 100,000 |
|---|---|
| Philadelphia, Allegheny, Montgomery, Lancaster, Bucks | Potter, McKean, Juniata, Jefferson, Armstrong |

The left list is, in order, Pennsylvania's largest counties. Potter tops the rate list at
**556.3 per 100,000** on **89 deaths**. Dividing by population removes the artifact: the rate
correlates with population at **−0.232**.

**The rule taught:** *does this bar get longer simply because more people live there?*

Appears three times — the lecture figure, the lecture check-in (`m5_rates_counts`), and lab
Exercise 8.

**Correction to §6 while building this module.** §6 cites COVID deaths × firearm suicides at
+0.851. Measured directly from these files, **COVID deaths × total firearm deaths is +0.962**;
the lecture notes quote the figure that was measured. Both illustrate the same point — two
counts correlate because both largely measure population.

## Exercises 9–11 — misleading, not broken

**The lab's distinguishing feature.** Three exercises supply figures that **execute cleanly and
state something false**, because "it ran" is the check students actually apply.

| Ex | The lie | The repair |
|---|---|---|
| 9 | `set_ylim(10.2, 10.9)` makes a **0.36-point** regional spread look enormous | zero baseline |
| 10 | No labels, no title, no source, no alt text — invisible to a screen reader | label it, describe it |
| 11 | Raw overdose **counts** titled "where the crisis hit hardest" — ranks by population | rank by `rate_per_100k` |

The lab's opening callout and its closing callout both state the standard: **running is not
checking.**

## Verification performed

**Every code cell was run before rendering** (§7 standard), using a local venv with
pandas 3.0.5 / numpy 2.5.2 / matplotlib 3.11.1 / seaborn 0.13.2.

- **Lecture: 25/25 pyodide cells execute**, zero failures. 30 content slides, **every one
  carrying speaker notes** (§2 requires the notes be load-bearing).
- **§4 construct coverage confirmed**, including all four named seaborn calls: `boxplot`,
  `heatmap`, and — added after an explicit coverage check — `histplot` and `scatterplot`,
  presented as the seaborn form of two figures students had already built by hand. That slide
  teaches `style="metro"` as the one-argument second channel.
- **Lab: 12/12 solutions pass their `#| check:` asserts.**
- **Lab: 12/12 checks discriminate** — each fails against a setup-only namespace.
- **Lab: 0/12 trivially passing** — every unedited blank raises before its check is reached
  (`AttributeError`, `NameError`, `KeyError`, `IndexError`), so no check can be satisfied by
  submitting the exercise unmodified.
- **Every number on every slide was computed from the real files**, not carried over from the
  handoff. Two handoff figures were corrected against measurement (below).

### Corrections made against measurement

- The leverage demonstration was drafted as r → 0.79. **Measured: 0.827.** Slide and alt text
  now state 0.827.
- §6's +0.851 count-artifact figure is for firearm *suicides*; **total firearm deaths ×
  COVID deaths measures +0.962** in these files. The lecture quotes the measured value.

### Register audit (§2 split-register)

- **Lecture:** second person appears **only** inside `::: {.notes}` blocks. One violation found
  and fixed (a callout titled "What a correlation cannot tell you" → "cannot establish").
- **Lab:** second person confined to `**Your turn.**` task instructions and hints. The 12 audit
  hits are all the mandated `— your turn` slide suffix, which §2 prescribes.

### Structure audit (§2 labs clause / §7)

12 section dividers · 24 exercise slides (12 worked example + 12 task) · 12 `**Your turn.**`
markers · **no exercise cell appears on a worked-example slide.**

## ⚠️ §8 — descriptive only

`.corr()` is taught as **co-movement**, never cause. No p-values, no significance tests, no
regression, no `scipy.stats`, no scikit-learn — all are available in Pyodide and none are used.

The ecological-fallacy caution is load-bearing here and appears in the lecture notes, the
closing-figure notes, and a lab callout: **these are county averages, so a correlation across
counties describes counties, not the individual people in them.**

The leverage slide exists to enforce the companion habit: **draw the scatter, then read the
correlation, in that order.** One fabricated point moves r from 0.659 to 0.827. The point is
labelled as fabricated both on the figure and in the notes.

## ⚠️ §6 trap 2 — PM2.5 is not used in this module

Neither `pa_air_pollution.csv` nor `pa_air_quality.csv` is mounted here. PM2.5 runs *opposite*
to disadvantage in Pennsylvania (child poverty × PM2.5 = −0.421, negative inside every
urbanicity stratum), so it must never carry a disparity claim. It belongs to M3, where it is a
threshold-masking exercise against a regulatory standard.

## Calendar

**The Tue/Thu pattern inverts at M5** (§3): lecture **Tue 11/10**, lab **Thu 11/12** — a
two-day gap, unlike M4's six-day gap across Election Day. The restatement load here is
therefore normal rather than maximal, though every lab exercise still restates its construct
in a worked example before the task uses it.

## seaborn runtime dependency — know this before teaching

seaborn is **not bundled** in Pyodide. It installs at runtime:

```python
import micropip
await micropip.install("seaborn")
import seaborn as sns
```

The `await` **must** be at cell top level. Two practical consequences:

- **It needs the network** to reach `files.pythonhosted.org`, a second CDN dependency on top of
  the one serving Pyodide itself. Run the install cell early in the session so the download
  happens while talking.
- **There is a real fallback.** Everything in the seaborn slides can be drawn with matplotlib
  and pandas, which are already loaded. seaborn is convenience, not capability. In Colab it is
  already installed and the `micropip` step is unnecessary.

## Standing carry-forwards — do not re-litigate

- Lab `#| check:` assert values across M1–M5 still need one confirmation against **live
  Pyodide** (Solution → Run in a real browser). Deferred to the pre-publish pass. The asserts
  here were verified against local pandas 3.0.5; Pyodide ships 2.3.1, and while nothing used
  here differs between them, the browser run is the authoritative check.
- In-browser keyboard/focus, 200% zoom, and UDOIT checks are deferred to pre-publish. **This
  module carries the most figure content in the course**, so its 200% zoom pass matters more
  than any other — figures at `figsize` widths of 9–9.5in are the ones to check first.
- Run `PREPUBLISH_CHECKLIST.md` before publishing.

## Handoff to Module 6

M5 closes by pointing at M6's cold open (S5): **everyone knows gun deaths are an urban problem,
and Pennsylvania's data disagrees** — nonmetro **18.5** per 100,000 against metro **14.1**, with
the suicide share (62% vs 55%) explaining it. M6 is a single combined session on Tue 11/17
where students build the full pipeline and write it up; the writeup is **30% of the grade**, and
M5's closing notes flag that early rather than in the last week.
