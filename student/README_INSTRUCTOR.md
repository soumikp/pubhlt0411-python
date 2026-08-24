# The student distribution folder

Everything in this folder goes to students. **This is the folder that becomes
`python.zip` on Canvas**, extracting to a folder named `python` inside the
`PUBHLT0411` folder the R half already established. Nothing else in the
repository is distributed.

| File | Purpose |
|---|---|
| `SETUP.md` | Install instructions, Windows and macOS, with troubleshooting |
| `pyproject.toml` | Pins Python 3.12 and the four course packages |
| `uv.lock` | Pins the exact version of all ~100 resolved packages |
| `module0_lab.ipynb` | The Module 0 lab — prints `SETUP OK` when the install is correct |
| `data/` | The seven CSVs the course reads, copied from the repository root at packaging time |

**Labs 1–6 are NOT in the zip.** Each `moduleN_lab.ipynb` is attached to its
Canvas lab assignment and posted the morning that lab runs. Students save it
into `python/`, beside `module0_lab.ipynb`, where `data/...` resolves without
a relative-path prefix. This keeps students from working ahead and lets a lab
be corrected up to the hour it runs without reissuing the download.

## Building the zip

From the repository root:

```bash
# 1. Refresh the copies. Only the seven CSVs the course actually reads —
#    pa_air_quality.csv and pa_hiv_county_year.csv are used by no deck or
#    lab, so they stay in the repo and out of the download.
rm -rf student/data && mkdir student/data
for f in pa_overdose_county_year pa_counties pa_covid_monthly \
         pa_air_pollution pa_maternal_infant pa_asthma_prevalence \
         pa_firearm_county_year; do cp "data/$f.csv" student/data/; done
cp data/SOURCES.md student/data/
cp module0/lab/module0_lab.ipynb student/module0_lab.ipynb
rm -rf student/.venv

# 2. Build python.zip, which extracts to a folder named `python`.
#    The name matters: the deck's folder trees and Step 4's `cd python`
#    both depend on it.
rm -rf /tmp/python && cp -r student /tmp/python
# Strip anything that must not reach a student. README_INSTRUCTOR.html is the
# one that matters: rendering the .md produces an .html that is NOT covered by
# excluding the .md by name, and it carries these build commands.
# Each on its own line: in zsh an unmatched glob (e.g. no *.html present)
# aborts the whole command, and anything listed after it is silently skipped.
rm -f  /tmp/python/README_INSTRUCTOR.md
rm -rf /tmp/python/.venv
find /tmp/python -name '*.html' -delete
find /tmp/python -type d -name '*_files' -exec rm -rf {} + 2>/dev/null
find /tmp/python -name '.DS_Store' -delete
(cd /tmp && zip -r python.zip python -x '.*' -x '__MACOSX/*' -x '*/.*')
mv /tmp/python.zip . && rm -rf /tmp/python

# 3. Confirm what actually shipped — should be exactly 14 files.
unzip -l python.zip
```

The result is about 340 KB. Everything else a student needs is downloaded by
`uv sync` on their own machine.

Note the exclusion of this file — `README_INSTRUCTOR.md` stays in the repository
and does not ship.

## Rules

**`data/` here is a COPY.** The master is the repository root `data/`, because
`tools/make_notebooks.py` reads from there. Edit the root copy, then re-copy —
never edit `student/data/` directly.

**`module0_lab.ipynb` here is a COPY.** The master is `module0/lab/module0_lab.ipynb`,
which keeps it beside every other module's lab. Edit that one, then re-copy. It ships
in the zip *root* rather than in a `lab/` subfolder because students run it next to
`data/`, and `SETUP.md` Step 6 walks them to it there.

**Never ship `.venv/`.** It is machine-specific and roughly 300 MB, and it
hardcodes absolute paths from the machine that built it — it would be broken on
every student laptop even if the size were acceptable. Students build their own
with `uv sync`; that is what `uv.lock` is for. The build commands delete it, and
`.gitignore` excludes it, but it reappears whenever `uv sync` is run inside
`student/` — which happens when testing the zip. **Check before every build.**

**This folder should contain exactly six items.** Anything else is litter:

```
student/
├── SETUP.md
├── module0_lab.ipynb
├── pyproject.toml
├── uv.lock
├── data/
└── README_INSTRUCTOR.md   (excluded from the zip)
```

**`uv.lock` is committed on purpose.** It is what makes every student's
environment identical. Regenerating it mid-semester changes package versions
under students who have already installed — if you must, re-run the module
notebooks against the new versions first.

## Verifying before distribution

Copy the folder somewhere outside the repository and run the whole student
sequence:

```bash
cd /tmp && rm -rf ziptest && cp -r <repo>/student ziptest && cd ziptest
uv sync
uv run jupyter lab      # open module0_lab.ipynb, Run All, expect SETUP OK
```

If that prints `SETUP OK`, the zip is good.
