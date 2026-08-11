# Course datasets — provenance

**Every value in these files is real, from a public source. Nothing is simulated,
smoothed, or invented.** Extracted 2026-08-11.

This replaces the 12-row representative teaching files used in the v1 build. Those
remain in `module*/lab/data/` and are superseded.

---

## `pa_counties.csv` — the reference table (67 rows)

The join target for every other file. One row per Pennsylvania county.

| Column | Meaning | Source |
|---|---|---|
| `county` | County name, no "County" suffix | — |
| `fips` | 5-digit FIPS code | Census |
| `population_2023` | Vintage 2023 population estimate | [Census CO-EST2023](https://www2.census.gov/programs-surveys/popest/datasets/2020-2023/counties/totals/co-est2023-alldata.csv) |
| `region` | PA DOH health district grouping (6 regions) | PA DOH district definitions |
| `urbanicity_code` | NCHS 2013 code, 1–6 | [NCHS Urban–Rural Classification Scheme 2013](https://www.cdc.gov/nchs/data/data_acces_files/NCHSURCodes2013.xlsx) |
| `urbanicity` | Label for the code above | NCHS |
| `metro` | Metro (codes 1–4) / Nonmetro (5–6) | derived from NCHS |

**Grouping variables.** `region` gives 6 groups (8–14 counties each); `metro` gives a
37/30 binary split. Both are real classifications, not teaching constructs. `region` and
`metro` cross-tabulate usefully — that is the natural `pd.crosstab()` example.

---

## `pa_overdose_county_year.csv` — S1 overdose (402 rows = 67 × 6)

Accidental and undetermined drug overdose deaths, 2019–2024.

**Source:** [PA DOH, *Estimated Drug Overdose Deaths CY 2012–Current*](https://data.pa.gov/resource/azzc-q64m.json)
(Socrata dataset `azzc-q64m`, data.pa.gov). Rates computed here as
`deaths / population_2023 * 100000`.

| Column | Notes |
|---|---|
| `overdose_deaths` | **Blank where PA DOH suppressed the cell** (count 1–5) |
| `rate_per_100k` | Blank wherever the count is suppressed |

### Why this file is worth more than a clean one

**57 of 402 cells (14.2%) are genuinely suppressed**, across 23 counties, because PA
DOH does not publish counts of 1–5. This is a real disclosure rule, not manufactured
missingness.

**Forest County is the teaching case.** Its 2019–2021 values are blank (suppressed) and
its 2022–2024 values are `0` (a real zero). *Missing* and *zero* sit in the same column,
with different meanings, in real data. That single county justifies `.isna()`,
`.fillna()`, and the "missing is not zero" lesson without any invention.

**Verification against the published anchor:** summing 2024 county counts (state row:
3,336) over total PA population gives a crude rate of **25.7 per 100,000**, against the
cited anchor of **25.6** (KFF/CDC NCHS, age-adjusted). The data reproduces the anchor to
within rounding and the difference between crude and age-adjusted.

**Note on 2024:** PA DOH marks 2024 as provisional — overdose death certificates are
delayed 3–6 months. The apparent 2024 decline is partly real and partly incomplete
reporting. This is worth stating in class rather than hiding.

---

## `pa_covid_monthly.csv` — S2 infectious disease, time series (3,283 rows)

COVID-19 deaths by county by month, March 2020 – March 2024 (49 months × 67 counties).

**Source:** [PA DOH, *COVID-19 Aggregate Death Data*](https://data.pa.gov/resource/fbgu-sqgp.json)
(`fbgu-sqgp`). Published daily; **aggregated here to monthly** — the daily file is 99,824
rows, too large for a teaching file, and monthly is the right granularity for a trend.

**This is the dated file.** `month` is a real ISO date (`2020-03-01`), so `pd.to_datetime()`,
`.dt.year`, and `.dt.month` have something genuine to parse. Every other file uses a plain
integer year.

The January 2022 Omicron peak (4,294 deaths statewide) is clearly visible, as is the
December 2021 Delta wave. Students recognise this history, which makes the trend figure
land.

---

## `pa_hiv_county_year.csv` — S2 infectious disease, cross-section (330 rows)

HIV new diagnoses and prevalence by county, **2012–2016**.

**Source:** [PA DOH](https://data.pa.gov/resource/buk2-94cb.json) (`buk2-94cb`).

⚠️ **These years do not overlap the other files.** PA's portal has no newer county-level
HIV table in machine-readable form. Use this file on its own, not joined to the 2019–2024
data. For a joinable infectious-disease story, use the COVID file instead.

---

## `pa_firearm_county_year.csv` — S5 firearm injury (402 rows = 67 × 6)

Firearm deaths, firearm suicides, and firearm homicides by county, 2019–2024.

**Source:** [CDC, *Mapping Injury, Overdose, and Violence — County*](https://data.cdc.gov/resource/psx4-wq38.json)
(`psx4-wq38`).

⚠️ **Different suppression convention.** CDC marks suppressed cells with the literal
string `1-9`, where PA DOH leaves them blank. **Normalised to blank here**, and this
difference is itself worth one slide — real data does not agree on how to say "missing."
159 of 402 cells (40%) are suppressed.

### The payoff, verified by running it

| 2023, pooled | Rate per 100k | Suicide share |
|---|---|---|
| Metro | 14.1 | 55% |
| **Nonmetro** | **18.5** | **62%** |

**Nonmetro firearm death rates are higher than metro** — which contradicts what most
students expect, and the suicide share explains why. This is a finding students discover
from the data rather than a claim they are told. It matches the cited anchor
(~half of US firearm deaths are suicides; rural rates are not lower).

Crude 2023 rate over unsuppressed counties: **13.0 per 100k** against the cited anchor of
14.3. The gap is expected — 40% of cells are suppressed, so the sum undercounts. **Worth
teaching explicitly**: this is what suppression does to a total.

---

## `pa_asthma_prevalence.csv` — S6 asthma (67 rows)

Adult asthma, smoking, and obesity prevalence by county, model-based estimates.

**Source:** [CDC PLACES, 2024 release](https://data.cdc.gov/resource/d3i6-k6z5.json) (`d3i6-k6z5`).

**Note:** PLACES values are *modelled* small-area estimates, not direct counts. Real and
citable, but the modelling is worth one honest sentence in class.

---

## `pa_air_quality.csv` — S6 air quality (198 rows)

Annual air-quality summary by county, 2019–2023.

**Source:** [EPA AQS, annual AQI by county](https://aqs.epa.gov/aqsweb/airdata/).

**Only 40 of 67 counties appear** — a county needs a monitor to have data. This is real
missingness with a real cause, and it makes a left join produce genuine NaNs. The 2023
Canadian-wildfire signal is visible (Allegheny: 15 unhealthy-for-sensitive-groups days).

---

## `pa_air_pollution.csv` — S3 air pollution (67 rows)

Annual average PM2.5 and drinking-water violations by county.

**Source:** [County Health Rankings & Roadmaps 2024](https://www.countyhealthrankings.org/)
(measures v125, v124), which derives PM2.5 from EPA monitoring and modelling.

**S3's home skill is M3 NumPy — z-scores and threshold masking.** The EPA annual PM2.5
standard is **9.0 µg/m³**, which gives a real cutoff to mask against:

- 22 of 67 counties exceed the standard
- mean 8.49, sd 1.29
- **Allegheny 14.1 = z +4.34** — a dramatic real outlier, and where many students live
- metro 8.87 vs nonmetro 8.02, in the expected direction

**Do not use this file for a disparity claim.** See the correlation sweep below — PM2.5 in
PA tracks industry and density, not poverty. The poverty columns were deliberately removed
from this file so it does one job cleanly.

---

## S6 changed: asthma × smoking, not asthma × air quality

**The originally planned M5 closing figure — "asthma tracks air quality" — does not exist
in the data.** Median AQI × asthma prevalence is **0.151**, essentially nothing. Asthma
prevalence barely varies statewide (9.7%–11.6%).

**Instructor decision: S6 is now asthma × smoking**, which is real:

| Relationship | r |
|---|---|
| **Smoking × asthma** | **+0.659** ✅ |
| Obesity × asthma | +0.476 |
| Median AQI × asthma | 0.151 ❌ |

`pa_air_quality.csv` stays in the repo as a secondary file — the 2023 Canadian-wildfire
signal and the 40-of-67 monitor coverage still make it a good `.isna()` / left-join example.

---

## Correlation sweep — what students can find in these files

Every numeric county-level pair was computed (`n ≥ 15`). Use this to know in advance what
a curious student will hit. **Two traps and one deliberate surprise:**

### ⚠️ Trap 1 — anything correlates with population

| Pair | r |
|---|---|
| COVID deaths (total) × population | **+0.946** |
| Firearm suicides (count) × population | **+0.921** |
| COVID deaths × firearm suicides | **+0.851** |

These are **counts, not rates**. Big counties have more of everything. The 0.851 between
COVID deaths and firearm suicides is pure population artifact and looks alarming if a
student plots it cold.

**This is a teaching asset, not a problem** — it is the "rates vs counts" lesson (M5 item
5.15) with a real number attached. Consider making it a deliberate check-in: *"Philadelphia
has the most of everything. So what?"*

### ⚠️ Trap 2 — PM2.5 runs opposite to disadvantage

| Pair | r |
|---|---|
| PM2.5 × population | +0.450 |
| PM2.5 × median income | +0.311 |
| PM2.5 × child poverty | **−0.421** |

In PA, PM2.5 is an **industrial/density** exposure. The dirtiest counties (Allegheny 14.1,
Lancaster 11.1) are dense metros; the poorest are rural with clean air. Stratifying by
urbanicity does **not** reverse it — the negative holds inside every stratum (metro −0.296,
nonmetro −0.417, all six NCHS levels negative). County averages cannot show environmental
justice, which operates at neighborhood scale.

**Mitigation applied:** `pa_air_pollution.csv` now carries only `county`, `pm25_annual_avg`,
and `drinking_water_violation`. The poverty columns were removed so the file does one job.
Use PM2.5 for the M3 **skill only** (z-scores, threshold masking) — never for a disparity
claim.

### ⚠️ Trap 3 — suppression biases grouped rates downward

`.sum()` skips NaN, but the population denominator still counts every county. A suppressed
county therefore contributes population and no deaths, pulling the group rate down.

2023 overdose rate by region, computed both ways:

| Region | Naive (all counties in denominator) | Correct (suppressed excluded from both) | Bias |
|---|---|---|---|
| Southwest | 42.9 | 42.9 | 0.0 |
| Southeast | 41.3 | 41.3 | 0.0 |
| Northwest | 37.5 | 39.5 | **−2.0** |
| Northeast | 35.4 | 36.4 | −1.0 |
| South Central | 22.3 | 23.0 | −0.7 |
| North Central | 20.3 | 22.1 | **−1.8** |

**The bias concentrates in rural regions**, which is exactly where suppression happens. It
is a *silent* error — the code runs, the numbers look plausible, and they are wrong. This
belongs in M4 as a check-in.

### ✅ The reliable, textbook-direction relationships

Build narrative payoffs on these:

| Pair | r | n |
|---|---|---|
| Child poverty × median income | −0.797 | 67 |
| Child poverty × teen births | +0.763 | 66 |
| Median income × smoking | −0.752 | 67 |
| Child poverty × smoking | +0.698 | 67 |
| **Asthma × smoking** (S6 home) | **+0.659** | 67 |
| Infant mortality × smoking | +0.658 | 44 |
| **Child poverty × infant mortality** (S4 home) | **+0.646** | 44 |
| Infant mortality × median income | −0.602 | 44 |

### The one deliberate surprise

**S5 firearm: nonmetro rate 18.5 vs metro 14.1, suicide share 62% vs 55%.** This is
counter-intuitive *by design* — v2 §9 names it as the story's anchor. It is the only
finding in the course that should surprise students.

---

## Still missing

Nothing. All six stories have real data.

**S3 changed topic.** Childhood lead has **no machine-readable PA county source** — checked
data.pa.gov, data.cdc.gov, and the national Socrata catalog. New York, Connecticut, Chicago
and NYC publish county/ZIP lead data; Pennsylvania publishes only PDF tables in its annual
Childhood Lead Surveillance Report. **S3 is now PM2.5 air pollution** (instructor decision),
which preserves the M3 skill — z-scores and threshold masking against a real regulatory
standard — at full 67-county coverage.

**S4 no longer needs CDC WONDER.** County Health Rankings carries real county infant
mortality. WONDER's API is blocked by Akamai, but it is no longer required.
