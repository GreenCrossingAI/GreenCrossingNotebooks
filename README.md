# GreenCrossingAI — Notebook quick start

This README covers the notebooks in this repo. There are two distinct workflows:

- Tarazed notebooks — intended for Windows machines with NVIDIA GPUs.
- Video_Processing_Current.ipynb — intended for use on NAIRR Jetstream2 JupyterHub instances.

## Tarazed notebooks — Windows GPU machines

Platform: Windows with an NVIDIA GPU. Match CUDA/cuDNN to the ML package you install.

1. Python: 3.8+ recommended.
2. Create and activate a venv (PowerShell):
	- python -m venv .venv
	- .\\.venv\\Scripts\\Activate.ps1
3. Install packages (adjust TensorFlow/PyTorch to match your CUDA):
	- pip install jupyterlab jupyter numpy pandas opencv-python pillow tqdm matplotlib
	- Install a GPU ML package per official instructions (e.g. tensorflow or torch built for your CUDA version).
4. Models & paths:
	- Download MegaDetector/model weights and set notebook variables (MEGADETECTOR_PATH, MODEL_FILE, DATA_DIR, OUTPUT_DIR) in the top cells.
5. Run:
	- jupyter lab
	- Open `Notebooks/Tarazed_*.ipynb` and run top-to-bottom.

## Video_Processing_Current.ipynb — NAIRR (Jetstream2) JupyterHub

Platform: NAIRR Jetstream2 JupyterHub instance (use instance compute/storage).

1. Prefer conda if available; venv works too. Example (bash on the instance):
	- python3 -m venv .venv
	- source .venv/bin/activate
2. Install packages:
	- pip install jupyterlab jupyter numpy pandas opencv-python pillow tqdm matplotlib
3. Data & storage:
	- Use $SCRATCH or project persistent storage for large video/frame sets. Update DATA_DIR/OUTPUT_DIR in the notebook accordingly.
4. MegaDetector / models:
	- Ensure model files and any MegaDetector code are available on the instance and point notebook variables to those paths.
5. Run:
	- Open the JupyterHub web UI, start `Video_Processing_Current.ipynb`, edit the top cells to set paths, then run cells top-to-bottom.

## Common notes

- If a requirements.txt exists, prefer: `pip install -r requirements.txt`
- Always edit the top cells in each notebook to set DATA_DIR, OUTPUT_DIR, MEGADETECTOR_PATH, and MODEL_FILE before running.
- For GPU usage, install GPU‑compatible TensorFlow or PyTorch per your CUDA/cuDNN and OS setup.

## Notebook conventions (versioning, naming, updating)

**Edit in place — don't save a new copy.** Don't create `_V1`/`_V2`/dated/`-Copy1`/`DEBUG-`
variants when unsure about a change. Edit the existing canonical notebook and commit — git history
*is* the version record, and it's far more reliable than a pile of similarly-named files. This is
the direct fix for the mess that motivated writing this section down: both this repo's
`old-Notebooks/` and a JupyterHub server's home directory had independently accumulated a scatter
of ad hoc copies with no clear "this one is current" signal.

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

**Outputs are stripped from notebooks before they're committed**, via
[`nbstripout`](https://github.com/kynan/nbstripout) — this keeps diffs and PR reviews readable
(otherwise every re-run changes execution counts/output blobs even when code didn't change). Set
it up once per clone:

```
pip install nbstripout
nbstripout --install --attributes .gitattributes
```

After that it's automatic — outputs still show while you work locally, they're just cleared at
commit time. If you ever need to see whether it's active: `git check-attr filter -- some.ipynb`
should print `nbstripout`.

