# Setting up Python for PUBHLT 0411

You will install two things: **uv**, which manages Python for you, and the
**course folder**, which contains the data and notebooks.

You do **not** need to download Python. You do **not** need Anaconda. If you
already have either one installed, leave it alone — nothing here will touch it.

Budget fifteen minutes. Read each step before typing it.

---

## Step 1 — Open a terminal

**Windows.** Press the Start button, type `powershell`, and open **Windows
PowerShell**. A blue window with a `PS C:\Users\yourname>` prompt appears.

**macOS.** Press `Cmd + Space`, type `terminal`, and press Return. A window with
a `yourname@macbook ~ %` prompt appears.

That prompt is where you type commands. Type them exactly, and press Return
after each one.

---

## Step 2 — Install uv

Copy the line for your computer, paste it into the terminal, and press Return.

**Windows (PowerShell):**

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

**macOS (Terminal):**

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

This downloads about 35 MB and takes a few seconds. You do not need
administrator rights, and you will not be asked for a password.

**Now close the terminal window completely and open a new one.** The install
changes a setting that only new windows pick up. Skipping this is the single
most common reason the next step fails.

Check that it worked by typing:

```
uv --version
```

You should see something like `uv 0.11.14`. If instead you see "not
recognized" or "command not found", go to Troubleshooting below.

---

## Step 3 — Download the course folder

Download `python.zip` from Canvas and **unzip it into the `PUBHLT0411` folder
you already made for the R half.** It extracts to a folder named `python`:

```
PUBHLT0411/                <- the folder from the R half
├── PUBHLT0411.Rproj
├── data/                  <- R data
├── lecture/  lab/  final-project/
└── python/                <- python.zip extracted here
    ├── data/              <- Python data
    ├── module0_lab.ipynb
    ├── pyproject.toml
    └── uv.lock
```

If you do not have a `PUBHLT0411` folder — a late add, or you missed the R
half — just make an empty one and extract into it. Put it somewhere you can
find again and that you control, such as `Documents`.

Three warnings, all of which have cost students real time:

- **`python` goes inside `PUBHLT0411`, not beside it.** The two halves of the
  course share one folder.

- **Actually unzip it.** Windows lets you open a zip file and look inside
  without extracting. That is not the same thing, and Python cannot work in
  there. Right-click the zip and choose *Extract All*.
- **Avoid OneDrive, Dropbox, and Google Drive folders.** Syncing software
  interferes with the files Python creates, in ways that produce confusing
  errors later. Documents is fine as long as it is not being synced.

---

## Step 4 — Point the terminal at the `python` folder

In your terminal, type `cd `, then a space, then drag the **`python`** folder
from your file browser onto the terminal window and press Return. Dragging
pastes the correct path, which saves you typing it.

Drag `python`, not `PUBHLT0411`. Dragging the outer folder is the most common
mistake here, and it makes Step 5 fail for a reason two steps back.

You can confirm you are in the right place:

- **Windows:** type `dir`
- **macOS:** type `ls`

Either way you should see `pyproject.toml` and a `data` folder in the list. If
you see `PUBHLT0411.Rproj` instead, you are one level too high — type
`cd python` and check again.

---

## Step 5 — Build the environment

```
uv sync
```

This is the step that installs Python and every package the course needs. The
first run downloads roughly 200 MB, so it takes a minute or two on a good
connection and longer on classroom wifi. You will see a long list of package
names scroll past. That is normal.

When it finishes, a `.venv` folder appears inside the course folder. That
folder is your Python installation. It belongs to this course only, and
deleting the course folder removes it completely.

---

## Step 6 — Check that it worked

```
uv run jupyter lab
```

Your browser opens to JupyterLab. In the file list on the left, double-click
`module0_lab.ipynb`, then from the menu choose **Run → Run All Cells**.

If the notebook prints **SETUP OK**, you are finished. Bring your laptop to
class.

---

## After today — the routine for every session

Steps 1–5 never happen again. From now on, three steps:

1. Open a terminal
2. `cd` into your `PUBHLT0411/python` folder
3. `uv run jupyter lab`

**Where lab notebooks go.** Each lab is posted to Canvas the morning it runs.
Download `moduleN_lab.ipynb` and save it into `python/` — the same folder as
`module0_lab.ipynb`, next to `data/`. A notebook saved anywhere else cannot
find the data files and fails with `FileNotFoundError`.

To stop JupyterLab when you are done, return to the terminal and press
`Ctrl + C` twice.

---

## Every time after this

You only do Steps 1–5 once. From then on, each time you work:

1. Open a terminal
2. `cd` into the course folder (Step 4)
3. `uv run jupyter lab`

---

## Troubleshooting

**`uv` is not recognized / command not found.**
You almost certainly did not open a *new* terminal window after installing.
Close every terminal window and open a fresh one. If that does not fix it,
restart the computer and try `uv --version` again.

**Windows: "running scripts is disabled on this system."**
Use the exact command in Step 2 — it includes `-ExecutionPolicy ByPass`, which
handles this. If you typed a shortened version, retype the full line.

**macOS: "operation not permitted."**
Your terminal is being denied access to that folder. Move the course folder to
`Documents` and try again.

**`uv sync` fails, or stops partway.**
Usually the network dropped. Run it again — it resumes rather than starting
over. On Pitt wifi, try `PITTNET` rather than the guest network.

**`No such file or directory: 'data/...'` inside a notebook.**
You started JupyterLab from the wrong folder. Stop it with `Ctrl + C`, `cd`
into the course folder, and start it again. The notebooks find data by
relative path, so where you launch from matters.

**Nothing above worked.**
Bring the laptop to office hours. Do not spend an hour on this alone, and do
not fall behind over a setup problem — it is not what the course is assessing.
Email `soumik@pitt.edu` with a screenshot of what the terminal shows and we
will sort it out.
