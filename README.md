# GreenCrossingAI — Notebook quick start

This repo holds notebooks that run [MegaDetector](https://github.com/agentmorris/MegaDetector)
over camera-trap footage — video and still images — to detect animals and produce annotated
output for review.

Want to contribute changes back to this repo? See [CONTRIBUTING.md](CONTRIBUTING.md) for the
team's conventions first.

## Repo structure

```
Notebooks/
  linux/     - notebooks for Linux / JupyterHub environments
  windows/   - notebooks for Windows machines with an NVIDIA GPU (a.k.a. "Tarazed")
old-Notebooks/ - deprecated predecessors, kept for reference only — not maintained
```

Each platform has two notebooks: one for video, one for still images.

| Platform | Video | Images |
|---|---|---|
| Windows / Tarazed | `Notebooks/windows/Video_Processing_Windows_2026.ipynb` | `Notebooks/windows/BatchProcessing_with_filedialog.ipynb` |
| Linux / JupyterHub | `Notebooks/linux/Batch-Video_Processing_2026.ipynb` | `Notebooks/linux/BatchProcessing_with_filedialog.ipynb`¹ |

¹ Currently a straight copy of the Windows images notebook and **not yet adapted for headless
use** — it still relies on `tkinter` file-picker dialogs, which need a display. Use the video
notebook on Linux for now if you need headless/no-display operation.

## Setup — Windows / Tarazed (NVIDIA GPU)

Platform: Windows with an NVIDIA GPU. Match CUDA/cuDNN to the ML package you install.

1. Python 3.9+ recommended.
2. Create and activate a conda/venv environment.
3. Install packages:
	- `pip install jupyterlab jupyter numpy pandas opencv-python pillow tqdm matplotlib`
	- `pip install megadetector` (pulls in a compatible `torch`/`torchvision`/YOLO stack automatically)
4. Model weights:
	- Stage MegaDetector model weights (e.g. `md_v5a.0.1.pt`) at a fixed local path rather than
	  relying on the package's auto-download — this avoids per-profile download/certificate issues
	  on shared machines, and avoids re-downloading after every reboot.
5. Run:
	- `jupyter lab`
	- Open the notebook for what you're processing (see table above) and run top-to-bottom. Folder
	  and model selection happen via file-picker dialogs.

## Setup — Linux / JupyterHub

Platform: any Linux JupyterHub instance (e.g. NAIRR Jetstream2, TLJH) with GPU access recommended.

1. Prefer conda if available; venv works too.
2. Install packages:
	- `pip install jupyterlab jupyter numpy pandas opencv-python pillow tqdm matplotlib`
	- `pip install megadetector`
3. Data & storage:
	- Use scratch or project persistent storage for large video/frame sets. Update `DATA_DIR`/
	  `OUTPUT_DIR` in the notebook's top cells accordingly.
4. Model weights:
	- Same advice as above — stage weights at a fixed, persistent local path (not a temp directory)
	  and point the notebook at that exact file, rather than an auto-downloading shorthand name.
5. Run:
	- Open the video-processing notebook (see table above) in JupyterLab, edit the top cells to set
	  paths, then run cells top-to-bottom.

## Common notes

- If a `requirements.txt` exists, prefer: `pip install -r requirements.txt`
- Always edit the top cells in each notebook to set `DATA_DIR`, `OUTPUT_DIR`, `MEGADETECTOR_PATH`/
  `MODEL_FILE` before running — they won't point at your data by default.
- For GPU usage, make sure your `torch` build matches your NVIDIA driver's supported CUDA version
  (check `nvidia-smi`) — a mismatch silently falls back to CPU rather than erroring.
