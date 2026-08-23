# The student distribution folder

Everything in this folder goes to students. **This is the folder that becomes
`pubhlt0411.zip` on Canvas.** Nothing else in the repository is distributed.

| File | Purpose |
|---|---|
| `SETUP.md` | Install instructions, Windows and macOS, with troubleshooting |
| `pyproject.toml` | Pins Python 3.12 and the four course packages |
| `uv.lock` | Pins the exact version of all ~100 resolved packages |
| `module0_lab.ipynb` | The Module 0 lab — prints `SETUP OK` when the install is correct |
| `data/` | The nine course CSVs, copied from the repository root at packaging time |

## Building the zip

From the repository root:

```bash
rm -rf student/data && cp -r data student/data
cp module0/lab/module0_lab.ipynb student/module0_lab.ipynb
rm -rf student/.venv
cd student && zip -r ../pubhlt0411.zip . \
    -x '.*' -x '__MACOSX/*' -x 'README_INSTRUCTOR.md' -x '.venv/*'
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

**Never ship `.venv/`.** It is machine-specific and roughly 300 MB. The commands
above delete it before zipping; `.gitignore` also excludes it.

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
