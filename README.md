# GreenCrossingAI: Notebook Quick Start

This repo holds notebooks that run MegaDetector over camera-trap footage (video and still images) to detect animals and produce annotated output for review.

> Want to contribute changes back to this repo? See [CONTRIBUTING.md](CONTRIBUTING.md) for the team's conventions first.

## Repo Structure

```
Notebooks/
├── linux/          # notebooks for Linux / JupyterHub environments
└── windows/        # notebooks for Windows machines with an NVIDIA GPU (a.k.a. "Tarazed")
old-Notebooks/      # deprecated predecessors, kept for reference only (not maintained)
```

Each platform has two notebooks: one for video, one for still images.

| Platform | Video | Images |
|---|---|---|
| Windows / Tarazed | `Notebooks/windows/Video_Processing_Windows_2026.ipynb` | `Notebooks/windows/BatchProcessing_with_filedialog.ipynb` |
| Linux / JupyterHub | `Notebooks/linux/Batch-Video_Processing_2026.ipynb` | `Notebooks/linux/BatchProcessing_with_filedialog.ipynb`¹ |

Both video notebooks skip videos with no animal detections, but their output videos differ on purpose: the Windows notebook writes an **animal-only highlight reel** (only frames with an animal, with boxes), while the Linux notebook writes the **full video** with boxes on the animal frames. Both use a confidence threshold of 0.2, Dan Morris's recommended default for the MDv5a model.

The Windows notebooks pick folders with `tkinter` file-picker dialogs, which need a display. The Linux notebooks are headless: paths are hardcoded in their top cells instead.

¹ Despite the name, the Linux images notebook has no file dialogs; it was adapted from the Windows one with the dialogs replaced by hardcoded paths.

`Notebooks/linux/Video_Processing_Windows_headless.ipynb` is not a separate workflow. It is a headless copy of the Windows video notebook, used to verify changes to that notebook's batch loop on a Linux GPU server. When the Windows video notebook's loop changes, port the same change into this copy.

## Setup: Windows / Tarazed (NVIDIA GPU)

**Platform:** Windows with an NVIDIA GPU and a driver recent enough for CUDA 11.8 (check `nvidia-smi`). The CUDA and cuDNN libraries ship inside the `torch` wheel, so no separate CUDA toolkit install is needed.
**Python:** 3.11 (what Tarazed runs).

**Why Tarazed needs extra steps:** Tarazed is a shared machine. Each person logs in with their own account, and accounts don't share settings, caches, or environment variables. A lot of Python tooling quietly assumes a single user: Jupyter checks your own account's kernels before the machine-wide ones, MegaDetector downloads model weights into each account's own `%TEMP%`, and each account can have a different `python` first on its `PATH`. So a notebook can work for the person who set it up and fail for the next person who logs in, often with misleading errors like `CERTIFICATE_VERIFY_FAILED` or `ModuleNotFoundError: megadetector`. Most of the steps below exist to put things in one machine-wide place that every account sees.

JupyterLab itself runs from the machine's Anaconda install; the environment below only needs to provide the kernel.

1. **Create and activate** a conda environment (Tarazed's is named `megadetector`).

2. **Install packages:**

   ```bash
   pip install megadetector ipykernel  # megadetector brings torch, numpy, pandas, opencv, pillow, tqdm, matplotlib
   ```

   On Windows, PyPI's `torch` wheel is CPU-only, so replace it with a CUDA build from PyTorch's own index and confirm the GPU is visible:

   ```bash
   pip install torch torchvision --upgrade --force-reinstall --index-url https://download.pytorch.org/whl/cu118
   python -c "import torch; print(torch.__version__, torch.cuda.is_available())"   # expect ...+cu118 True
   ```

   The `cu118` index tops out at torch 2.7.1 (newer torch releases dropped CUDA 11.8), which is what Tarazed runs. Re-run this step after any `pip install --upgrade megadetector`, since an upgrade can pull the CPU-only wheel back in.

3. **Register the kernel system-wide, not per-user.** On a shared machine, `python -m ipykernel install --user ...` registers the kernel under `%APPDATA%\jupyter`, which Jupyter's data-file search order checks before the machine-wide location. As a result, a per-user kernel can silently shadow the one other accounts are meant to use. From an **admin** prompt, with the `megadetector` environment activated, install it machine-wide instead:

   ```bash
   python -m ipykernel install --name megadetector --display-name "Python (megadetector)"
   ```

   (No `--user`, no `--prefix`; this is why it needs admin rights.) This lands in `%PROGRAMDATA%\jupyter\kernels\`, so every account on the machine sees the same kernel. It also writes the environment's full interpreter path (e.g. `C:\ProgramData\mambaforge\envs\megadetector\python.exe`) into the kernelspec, so the kernel doesn't depend on which `python` is first on each account's `PATH`. If a per-user copy already exists from an earlier install, remove it; otherwise it continues to shadow the machine-wide one for any session launched from that account.

4. **Model weights:** Stage the MegaDetector model weights (e.g. `md_v5a.0.1.pt`) at a fixed, shared local path, such as `C:\ProgramData\megadetector_models\`, rather than relying on the package's auto-download. On a shared machine, the package's default cache lives under each user's own `%TEMP%`, so a different account than the one that downloaded the weights will attempt its own re-download the first time it runs. This can surface as a `CERTIFICATE_VERIFY_FAILED` error that has nothing to do with the certificate itself. Point every notebook at the same shared file: the video notebook sets it in `MODEL_PATH` (already `C:\ProgramData\megadetector_models\md_v5a.0.1.pt`) and fails early if it's missing; the images notebook asks for it in a file-picker dialog.

5. **Run:** Start JupyterLab, open the notebook for what you're processing (see table above), select the **Python (megadetector)** kernel, and run top-to-bottom. Input and output folders are chosen in file-picker dialogs.

## Setup: Linux / JupyterHub

**Platform:** Any Linux JupyterHub instance (e.g. NAIRR Jetstream2, TLJH), with GPU access recommended.

On the team's TLJH server (Altair), this is already set up: MegaDetector is installed system-wide under `/opt/tljh/user` and exposed as the **Python (MegaDetector 10)** kernel (`megadetector10`), and the model weights are at `/shared/megadetector_models/md_v5a.0.1.pt`. Skip to step 3. For a new server:

1. **Install packages** (prefer conda if available; venv works too):

   ```bash
   pip install megadetector ipykernel
   ```

   No extra torch step is needed here: PyPI's Linux `torch` wheel already includes CUDA support. The video notebook also calls the `ffmpeg` command-line tool to extract frames, so make sure `ffmpeg` is installed and on `PATH`.

2. **Model weights:** Same advice as above. Stage weights at a fixed, persistent, shared path (not a temp directory; the package's auto-download cache is under `/tmp`) and point the notebook at that exact file, rather than an auto-downloading shorthand name.

3. **Set paths:** Use scratch or project persistent storage for large video/frame sets. Edit the top cells of the notebook before running:
   - Video notebook: `DATA_DIR`, `RUN_LABEL`, and `MODEL_NAME`
   - Images notebook: `folder_path`, `RUN_LABEL`, and `model_path`

   Output goes to `./megadetector_outputs/<RUN_LABEL>_<timestamp>/`, so repeat runs never overwrite each other.

4. **Run:** Open the notebook in JupyterLab, select the MegaDetector kernel, and run cells top-to-bottom.

## Verifying Your Install

Before assuming a bug is in the code, confirm which MegaDetector the environment is actually running. Run these with the kernel's environment activated:

```bash
python -c "import megadetector as m; print(m.__file__)"
python -m pip show megadetector
```

If the printed file path does not fall inside the `Location` that `pip show` reports, something earlier in the module search path is shadowing the install. Most commonly this is a machine-wide `PYTHONPATH` entry pointing at an old clone. Check it:

```bat
:: cmd
echo %PYTHONPATH%
```

```powershell
# PowerShell
$env:PYTHONPATH
```

A shadowed install upgrades without error and without visibly changing anything, so this check is worth running after any `pip install --upgrade megadetector`, not just when something looks broken.

## Common Notes

- On Linux, the paths in the notebooks' top cells point at the last dataset someone ran, not at your data. Always check them before running.

- For GPU usage, make sure your torch build matches your NVIDIA driver's supported CUDA version (check `nvidia-smi`). A mismatch silently falls back to CPU rather than erroring.

- The video notebooks load the model once and call MegaDetector's Python API in-process. The images notebooks run it as a command; when you do that, invoke it via `subprocess.run([sys.executable, "-m", "megadetector...", ...])`, not a bare `python` or `!python`. A bare interpreter name can resolve to a different Python than the one running the notebook (e.g. an old 3.8 env still on `PATH`), which shows up as `ModuleNotFoundError: megadetector` even though the package is installed correctly in the kernel's own environment.

- The Windows video notebook (and its headless copy) skips a video that fails rather than halting the run. The summary lists failed videos separately from videos with no animals, with the reason for each failure, so a broken video isn't mistaken for an empty one. If you're extending that batch loop, preserve this behavior. The Linux video notebook does not do this yet: a failed `ffmpeg` extraction or a video that yields no frames stops the whole run.
