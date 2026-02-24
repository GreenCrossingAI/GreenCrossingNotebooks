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

