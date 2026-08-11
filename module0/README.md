# Module 0 — Notebooks and Values

**PUBHLT 0411 · Python for Public Health Data Analysis**
**Lecture:** Tue 10/6 · **Lab:** none — M0 has no lab session (see HANDOFF v3 §3)
**Home story:** S1 — drug overdose deaths, Pennsylvania counties
**Packages required:** none (`packages: []`)

## One skill

**Run a notebook and store values in typed variables.**

M0 is a full teaching session, not orientation. It absorbs Python's basic values layer,
which is what gives every later module roughly a third of a session of headroom.

## File manifest

```
module0/
├── class/
│   ├── module0_lecture.qmd       # lecture slides (live-revealjs, interactive Pyodide cells)
│   ├── module0_lecture.html      # rendered output
│   ├── pitt-theme.scss           # revealjs theme (copied from repo root)
│   ├── _extensions/r-wasm/live/  # quarto-live extension (copied from repo root)
│   └── data/
│       └── pa_overdose_county_year.csv   # VFS copy — see note below
└── README.md
```

## Rendering

From inside `module0/class/`:

```bash
quarto render module0_lecture.qmd --to live-revealjs
```

Two mechanics that are easy to get wrong and expensive to debug:

1. **`{{< include _extensions/r-wasm/live/_knitr.qmd >}}` must sit right after the YAML.**
   It wires up the `{pyodide}` cell engine. Without it the code cells render as dead text.
2. **A CSV reaches Pyodide only if it is listed under `pyodide: resources:`.** Dropping it
   in `class/data/` is not enough, and the failure is invisible until a cell errors in the
   browser. The lecture needs its **own** copy under `class/data/` — the repo-root `data/`
   directory is not on the render path.

Verify the wiring after rendering by decoding the VFS manifest:

```bash
python3 -c "
import re, base64
h = open('module0_lecture.html').read()
print(base64.b64decode(re.findall(r'<script type=\"vfs-file\"[^>]*>(.*?)</script>', h, re.S)[0]))
"
```

It should print `["data/pa_overdose_county_year.csv"]`. **Verified 2026-08-11.**

## Structure

| Section | Slides | Content |
|---|---|---|
| Cold open | 2 | 3,336 overdose deaths, 2024; provenance and the provisional-data caveat |
| The course | 4 | Student roadmap (§9), what the course is for, calendar traps, grading, the AI policy |
| Colab | 4 | Sign in and save a copy · turning Gemini off · cells and execution order · getting a file in |
| Values | 9 | Variables · naming · types · `type()` · conversion · `TypeError` · arithmetic · f-strings · comments |
| Payoff | 4 | Open the real file · compute both counties · the gap · session outcomes |

## The cold open and its payoff

**On the slide** (declarative, per amended §2): *"Pennsylvania recorded 3,336 overdose
deaths in 2024."* The question is framed as what the session establishes — what that
number means for a single county — rather than as a promise addressed to the reader.

**In the notes** (spoken): the pause before the borough comparison, the local anchor to
pick from, and the instruction not to answer the question yet.

**Payoff:** Schuylkill **41.7** vs Centre **4.4** per 100,000 — computed live, from the
real 2023 rows, with code the students filled in. The slide states the finding and the
ninefold difference. **The rhetorical close is spoken, not printed** — *"Two counties,
nearly the same size. There's the gap. You wrote the code that found it, in your first
hour of Python"* is in the speaker notes.

The handoff's original cold-open wording (*"By the end of today you will make Python tell
you…"*) is second person and was rewritten for the slide; its content survives in the
notes.

**Hand-off to M1:** *which county is which, and on what evidence?* M1 opens on the same
two counties with the same framing and the same numbers.

## Data notes — read before teaching

**`3,336` is the published PA DOH state total for 2024. The county file sums to 3,294.**
The 42-death difference is suppression: PA DOH does not publish counts of 1–5, so 14
counties are blank in 2024. Both numbers are correct and they cannot be reconciled from
the county file alone. This is stated honestly on the provenance slide, and it is the
first quiet appearance of the suppression thread that becomes a full teaching case in M4.

**The two-county comparison uses 2023, not 2024** — Schuylkill 60 deaths / 143,786 and
Centre 7 / 157,795, matching the anchors in HANDOFF v3 §5 and §6.

**The printed ratio is `9.4x`, not 9.5×.** The handoff's 9.5 comes from dividing the
*rounded* rates (41.7 / 4.4 = 9.48); the code divides the unrounded rates and prints 9.4.
The slides say "nine times," which is true of both. If you change the code to round first,
change the prose too.

**2024 is provisional.** Death certificates lag three to six months, so the apparent 2024
decline is partly real and partly incomplete reporting. The slide says so.

## Teaching devices

- **Two fill-in-the-blank live demos** (`m0_assign`, `m0_rate`), each a complete scaffold
  with one `______` blank, wired with `#| hint:` and `#| solution:`. Fill them in with the
  class rather than assigning them as solo work — M0's blanks are about the mechanics of
  running a cell.
- **One deliberately broken cell** — `"60" + 8` raising a real `TypeError`. Verified to
  raise. The slide states the error, the cause, and the fix in technical prose; the
  instruction to **run it broken on the projector** and read the traceback's last line
  aloud lives in the speaker notes, per the amended §2.
- **One silent-wrong-answer demo** — `"60" + "8"` giving `"608"` with no error. This is the
  first time students see that the dangerous failures are the quiet ones.
- **R bridges, one line each.** Three on the slides (`<-` vs `=`; the four type names with
  `TRUE` vs `True`; "prior R coursework transfers directly"), plus `snake_case` vs R's
  conventions and `sprintf`/`glue` vs f-strings in the speaker notes. Per §12, no
  comparison lecture — one line and move on.

## Verification status

- **All 17 `{pyodide}` cells run.** `python3 ../../tools/run_cells.py module0_lecture.qmd`
  from inside `class/`. The two `______` blanks are skipped as expected; both solutions
  run; the `TypeError` cell raises as designed.
- **Every printed number was executed, not asserted from memory** — including the ugly
  `41.728680121847745` quoted on the f-string slide and the `403` line count.
- **Renders clean** to `live-revealjs`; VFS manifest decoded and confirmed.
- **No lab to verify** — `tools/verify_lab.py` does not apply to M0.

## Accessibility (WCAG 2.2 AA)

- **No figures in M0**, so no alt-text obligation. M0 is text, tables, and code.
- All 21 callouts carry titles.
- Inherits the audited `pitt-theme.scss` (contrast-remediated, visible focus, reduced
  motion). No new colors introduced.
- **Run `PREPUBLISH_CHECKLIST.md`** before publishing for the in-browser checks — keyboard
  and focus, 200% zoom, and UDOIT once it is on Canvas.

## Voice sheet compliance (§2, amended 2026-08-11)

**M0 is the reference implementation of the split-register rule.** The slide is academic
and impersonal; the speaker notes carry the conversation. M0 was originally built to the
prior rule (conversational slides) and rebuilt after the instructor read it rendered.

Audited on the current file:

| Check | Result |
|---|---|
| Second person (`you`/`we`/`our`) on slides | **0 instances** |
| Second person in speaker notes | 38 instances across 25 blocks |
| Speaker-note volume | **2,206 words** — up from ~1,300 pre-amendment |
| Litotes, emoji, mascots | none |
| Callouts carrying titles | 21 of 21 |

The notes grew because the conversational content was **relocated, not deleted** — §2 now
treats that as a defect check. Everything the slides gave up landed in the notes: the
pause before the borough line, "somebody in this room is going to lose an hour of lab work
to this," the instruction to run the `TypeError` cell broken on the projector, and the
spoken payoff line at the close.

To re-audit after any edit:

```bash
python3 -c "
import re
src = re.sub(r'^---.*?^---', '', open('module0_lecture.qmd').read(), flags=re.S|re.M)
slide = re.sub(r'::: \{\.notes\}.*?:::', '', src, flags=re.S)
slide = re.sub(r'\`\`\`\{?pyodide\}?.*?\`\`\`|\`\`\`python.*?\`\`\`', '', slide, flags=re.S)
hits = [l.strip() for l in slide.split(chr(10)) if re.search(r'\b(you|your|we|our|us)\b', l, re.I)]
print('second person on slides:', len(hits))
[print(' ', h[:90]) for h in hits]
"
```

Every slide headline is a full sentence stating its claim. Real variable names throughout
(`county`, `overdose_deaths`, `population`, `rate_per_100k`) — the one `x = 60` is the bad
example on the naming slide, shown directly against its good counterpart.

## Learning objectives

- **LO1** (computing strategy) — the assign → convert → compute → format arc.
- Course-description "basics of programming" — variables, types, arithmetic, `print()`,
  comments, and the first real error message.
- **Forward hook:** the values layer is the atoms every NumPy array holds in M3.

## Instructor TODO

- **Timing is untested.** The Colab section is four slides but runs long when done live on
  the projector, which it should be. If the room is slow, the compressible part is the
  grading table — it is on Canvas.
- **Do the Colab and Gemini steps live**, with the class following along, and ask for hands
  up when both AI toggles are off. It is the only way to know the room actually did it.
- **Pyodide's first load can be slow on classroom wifi.** Open the rendered lecture and let
  the runtime warm up before class starts.
- Confirm the Canvas setup page names **Colab** as official — the syllabus currently names
  Anaconda (HANDOFF v3 §11, open item 6).
