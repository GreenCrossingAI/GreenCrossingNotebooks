# Contributing

This describes how the GreenCrossingAI team works on this repo. If you've forked this project for
your own unrelated use, none of this is required — it's here for anyone actually committing
changes back to this repository.

## Notebook conventions (versioning, naming, updating)

**Edit in place — don't save a new copy.** Don't create `_V1`/`_V2`/dated/`-Copy1`/`DEBUG-`
variants when unsure about a change. Edit the existing canonical notebook and commit — git history
*is* the version record, and it's far more reliable than a pile of similarly-named files. This is
the direct fix for a mess that actually happened: both this repo's `old-Notebooks/` and a
JupyterHub server's home directory independently accumulated a scatter of ad hoc copies with no
clear "this one is current" signal.

**One canonical notebook per pipeline, per platform**, living under `Notebooks/<platform>/`:

- `Notebooks/linux/Batch-Video_Processing_2026.ipynb` — video processing, Linux/JupyterHub
- `Notebooks/linux/BatchProcessing_with_filedialog.ipynb` — image processing, Linux/JupyterHub
- `Notebooks/windows/Video_Processing_Windows_2026.ipynb` — video processing, Windows/Tarazed
- `Notebooks/windows/BatchProcessing_with_filedialog.ipynb` — image processing, Windows/Tarazed

`old-Notebooks/` holds deprecated predecessors, kept only for reference — don't add new work there.

**Branch and PR every notebook change, even solo.** Keeps `main` always in a known-good state on
every machine that pulls from it (multiple machines do), and gives a review checkpoint before a
change lands — much easier than untangling a bad notebook merge after the fact.

**Pull before you start working, push/PR promptly after.** Any machine that edits notebooks
(Windows/Tarazed box, a JupyterHub server, a laptop) should `git pull` before making changes and
get finished, tested changes back to GitHub promptly — not let a local copy drift for months.

## Dev setup: `nbstripout`

Outputs are stripped from notebooks before they're committed, via
[`nbstripout`](https://github.com/kynan/nbstripout) — this keeps diffs and PR reviews readable
(otherwise every re-run changes execution counts/output blobs even when code didn't change). Set
it up once per clone:

```
pip install nbstripout
nbstripout --install --attributes .gitattributes
```

After that it's automatic — outputs still show while you work locally, they're just cleared at
commit time. If you ever need to check whether it's active: `git check-attr filter -- some.ipynb`
should print `nbstripout`.
