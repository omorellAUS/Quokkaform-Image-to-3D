<p align="left"><img src="app/assets/quokkaform_logo.png" alt="QuokkaForm Images To Mesh Expert" height="110"></p>

# QuokkaForm – Images To Mesh

**QuokkaForm Images To Mesh Expert** (version 1.0.0, Expert edition)

A free, open-source, local image-to-3D tool for Windows. Upload a photo of a real object (or several photos from different sides, or a short walk-around video) and get a downloadable, printable mesh on your own PC: background removal, then Microsoft TRELLIS on your NVIDIA GPU.

The app opens in its **own native window** (pywebview / Microsoft Edge WebView2). It does not open a web browser.

> Free to use, but some parts it depends on (nvdiffrast, the Inria gaussian rasteriser) are **non-commercial / research only**. See [LICENCE_FINDINGS.md](LICENCE_FINDINGS.md) before using it for paid work.

## Current status

**1.0.0 Expert edition** (November 2026). Single-image and experimental multi-image modes use `microsoft/TRELLIS-image-large` on an NVIDIA CUDA GPU. The original alpha used TripoSR and ran in a browser; that path has been replaced.

The current build keeps the version number at **1.0.0**. Later CAD and edge-quality work (quad target up to 100,000, hard-surface sharpening, VRAM offload before decode) is included in this same edition.

The usual way to install or update is **QuokkaForm-Setup-1.0.0.exe**. Run it straight over an existing 0.9.x or 1.0.0 install — you do not need to uninstall first. Models, settings, History and outputs stay in `%LOCALAPPDATA%\QuokkaForm` and `Documents\QuokkaForm\outputs`.

## Features

- **Native Windows window** (pywebview / WebView2), Start menu and optional desktop shortcut, own taskbar icon. No browser, no console in the installed app.
- **TRELLIS-image-large** generation on NVIDIA CUDA (PyTorch 2.5.1 + CUDA 12.4), targeted at an **RTX 3060 12 GB** (TRELLIS officially recommends 16 GB).
- **Automatic background removal** with a choice of model: BiRefNet lite (default), BiRefNet full, IS-Net, or U2-Net, plus mask clean-up and a **Preview cutout** before you generate.
- **Single image**, or **multi-image** (experimental TRELLIS conditioning) with quality presets: **Draft** (1 image), **Standard** (up to 3) and **High** (4–8). You can upload more photos (up to 40); the preset keeps the sharpest, most varied views. Beyond about 8 views the gains are small.
- **Video input**: a 10–15 s walk-around MP4 or MOV. Frames are scored for sharpness, blurry and near-duplicate frames are dropped, and evenly spread frames are shown in a gallery so you can untick any you do not want.
- **Known scale** (Create tab and Batch, saved in History):
  - **Measure object** (default): enter one real-world size in mm — **Height** by default, or width, depth or longest side.
  - **Turntable reference**: enter the turntable diameter in mm. QuokkaForm fits an ellipse to the disc in the original photo. You can also enter a **known object height**; that sets the scale, with the turntable estimate as a cross-check (a warning if they differ by more than about 5%).
  - After every run the real-world bounding box (width × depth × height in mm) is shown.
- **AI Cleanup** before generation, each step with its own on/off switch, a before/after preview, and applied automatically in Batch: edge refinement and feathering, shadow reduction, and auto brightness / contrast / white balance.
- **Select object** (SAM 2): click to keep the object and remove e.g. a hand holding it, or use **Auto-select** to find and remove a hand or sleeve.
- TRELLIS controls: seed, sampling steps and guidance for both stages, **Simplify** (0.50–0.98, default 0.95; lower keeps more faces and detail but is slower), texture size.
- **Texture bake**: Fast (default, one pass), Quality (slower, about 4 GB more VRAM) or None (untextured GLB).
- Downloads: **GLB** (textured, metres, Y-up), **OBJ / STL / PLY** (millimetres, Z-up, sitting on the build plate).
- Optional **target triangle count** for STL/OBJ/PLY (0 = off, or 500 to 500,000). The GLB keeps its texture and the Simplify level.
- **Flatten base**: slice a cut height (default 1 mm) off the bottom of STL/OBJ/PLY and close it with a flat face at Z = 0.
- **Re-export**: change size, triangle count, flatten, quads or hard-surface options and re-export in seconds without regenerating.
- **CAD quad OBJ for Fusion 360**:
  - Target **200 to 100,000** quads (default 20,000), with preset buttons for 1,000, 5,000, 20,000, 50,000 and 100,000.
  - Remeshed with Instant Meshes (BSD) into a watertight, all-quad mesh in mm (Known scale).
  - Quads are built from a second, less-simplified TRELLIS mesh (Simplify 0.8), then projected onto the original surface — not from the already-simplified print mesh.
  - **Align quads to features** (on by default, feature angle 30°): quad edges follow sharp edges, creases and the borders of flat faces. This feature-aligned remesh is available up to **30,000** quads; above that, uniform quads are used and the Export info says so.
  - **Group flat faces** and **Adaptive density**: flat areas become OBJ groups (`flat_001` …, `curved_001` …), with bigger quads on flats and smaller ones on curves. A coloured region preview (`model_quads_groups.glb`) is saved.
  - A fold-unfold pass tries to un-fold quads that have folded back at thin tips.
  - In Fusion 360: *Insert Mesh*, then *Convert Mesh* (quad faceted / T-spline).
- **NURBS / planar export (.3dm)** (off by default): `model_nurbs.3dm` via rhino3dm / openNURBS (MIT), in mm, which Fusion 360 and Rhino open. Flat groups become planar trimmed faces; curved groups become degree-3 NURBS patches. Faces are separate objects (rhino3dm cannot join them): in Fusion use *Surface > Stitch*. Export info reports deviation in mm, or **deviation n/a** when nothing was actually fitted.
- **Hard-surface / sharp edges** (off by default, strength 0–1, default 0.5): flattens wavy panels and restores creases on mechanical models. It is applied to STL/OBJ/PLY and the quads; the **GLB is left as the textured TRELLIS mesh**.
- **Turntable export**: an MP4 plus an optional looping GIF, with a choice of length, resolution, background colour and render mode (colour, normals, or both).
- **Batch**: queue photos, videos or a folder; each subfolder can be one multi-image object; optional per-job settings; the batch carries on if a job fails; on-screen summary plus a **CSV report** in `outputs\batch_reports\`.
- **History** tab: every run can be saved to `Documents\QuokkaForm\outputs\<date>_<time>_<name>\` (inputs, cut-outs, exports, video, settings, log).
- Remembered settings (with **Reset settings to defaults**), plain-language errors, a **Setup** tab with **System check** and model download, per-stage timings and peak VRAM in the Export info (including the build id).
- VRAM handling aimed at 12 GB cards: background models run on the CPU and are released before TRELLIS; GPU memory is freed after each job; sampling (flow) models are moved to system RAM and the CUDA cache is emptied **before decode**.

## Requirements

**Installed app (QuokkaForm-Setup-1.0.0.exe)**

- Windows 10 or 11 (64-bit). The app is Windows-only; a macOS port was considered and dropped, because TRELLIS depends on NVIDIA CUDA.
- NVIDIA GPU with a current driver (**551.61 or newer**, CUDA 12.4). An **RTX 3060 12 GB** is the card this edition is tested and tuned for. 8 GB is a practical minimum; 16 GB is what TRELLIS officially recommends.
- About 15 GB free disk for the first-run Python runtime and models (models are about 4.3 GB).
- Microsoft Edge WebView2 Runtime (already on Windows 11; the installer offers it on Windows 10).
- No system Python, Git, CUDA toolkit or admin rights required.

**Developer / from-source setup**

- **Python 3.10 only** (64-bit, add to PATH). The prebuilt flash-attn and related wheels used here are `cp310`, built for PyTorch 2.5.1 + CUDA 12.4.
- CUDA 12.4 toolkit and Visual Studio 2022 C++ Build Tools (to build nvdiffrast and diff-gaussian-rasterization on first install, unless prebuilt wheels are bundled).
- Git (to clone TRELLIS).

## Install (Windows)

1. Close QuokkaForm if it is open.
2. Run **QuokkaForm-Setup-1.0.0.exe**. If you already have 0.9.x or 1.0.0 installed, run the new setup **over the top** — do not uninstall first, so the models are kept.
3. If Windows SmartScreen warns (the installer is not code-signed), click **More info > Run anyway**.
4. The first run on a new PC sets up its own Python 3.10 and packages (10–30 minutes) and can download the models. Later updates only check what is already there, which takes a few minutes.
5. Open **QuokkaForm** from the Start menu or the desktop shortcut. It opens in its own window.

The Expert edition uses the same installer ID, the same install folder (`%LOCALAPPDATA%\Programs\QuokkaForm`) and the same data folder (`%LOCALAPPDATA%\QuokkaForm`) as 0.9.x.

Where things live:

- Program: `%LOCALAPPDATA%\Programs\QuokkaForm`
- Python + packages: `%LOCALAPPDATA%\QuokkaForm\runtime`
- Models: `%LOCALAPPDATA%\QuokkaForm\models` (older Hugging Face / torch / `.u2net` caches are reused if they already have the files)
- Logs: `%LOCALAPPDATA%\QuokkaForm\logs`
- Settings: `%APPDATA%\QuokkaForm`
- Outputs: `Documents\QuokkaForm\outputs\<date>_<time>_<name>\`

Uninstalling removes the program, Python runtime and logs. It **keeps** models, settings and outputs unless you tick them.

Full steps, repair and uninstall: [INSTALL_WINDOWS.txt](INSTALL_WINDOWS.txt).

If you have the portable ZIP instead: extract `QuokkaForm-1.0.0-portable.zip` and start `QuokkaForm.cmd`. It reuses the same runtime and models folder.

## From source (developers)

1. Install Python 3.10 from [python.org](https://www.python.org/downloads/) and check **Add python.exe to PATH**.
2. Clone this repo (it keeps its older GitHub name):

```bat
git clone https://github.com/omorellAUS/Image-to-3D_Meshtastic.git
```

3. Double-click `start.bat` (the window title and banner say *QuokkaForm Images To Mesh Expert*).
   - It creates a Python 3.10 virtual environment
   - Clones TRELLIS into `TRELLIS\` if needed
   - Installs PyTorch 2.5.1 (CUDA 12.4) and the other dependencies (first run only; `start.bat --reinstall` forces a reinstall)
   - Starts the Gradio UI in your **browser** at http://127.0.0.1:7860 (local only; set `IMG3D_HOST=0.0.0.0` to reach it from other PCs on your network)

To open the same native window the installed app uses:

```bat
start.bat --desktop
```

First run takes longer while packages download. Then download the models (about 4.3 GB) **before** the first generation, so it does not sit silently for many minutes:

```bat
start.bat --download-models
```

or, in the app, **Setup > Models > Download missing models**. If models are missing, a banner at the top of the app says so.

### Manual start (optional)

```bat
py -3.10 -m venv venv
venv\Scripts\activate
git clone --recurse-submodules https://github.com/microsoft/TRELLIS.git TRELLIS
pip install torch==2.5.1 torchvision==0.20.1 --index-url https://download.pytorch.org/whl/cu124
pip install --no-build-isolation -r requirements.txt
set ATTN_BACKEND=flash_attn
set SPCONV_ALGO=native
python app/main.py
```

`start.bat` sets `ATTN_BACKEND=flash_attn` and `SPCONV_ALGO=native`. If flash-attn does not work on your GPU, install xformers (`pip install xformers==0.0.28.post3 --index-url https://download.pytorch.org/whl/cu124`) and set `ATTN_BACKEND=xformers` before running `start.bat`.

### Models (download up front)

| Model | Used for | Licence | Download | Stored in |
|-------|----------|---------|----------|-----------|
| TRELLIS-image-large (Microsoft) | 3D generation | MIT | ~2.8 GB | Hugging Face cache (`%USERPROFILE%\.cache\huggingface`, or `HF_HOME`; installed app: `%LOCALAPPDATA%\QuokkaForm\models`) |
| DINOv2 ViT-L/14 with registers (Meta) | Image encoder inside TRELLIS | Apache-2.0 | ~1.2 GB | torch hub cache (`%USERPROFILE%\.cache\torch\hub`, or `TORCH_HOME`) |
| BiRefNet lite | Background removal (default) | MIT | ~214 MB | `%USERPROFILE%\.u2net` (or `U2NET_HOME`) |
| SAM 2.1 Small (Meta) | Select object / Auto-select | Apache-2.0 | ~176 MB | Hugging Face cache |

- `start.bat --download-models` downloads whatever is missing, shows progress and the size of each model, then exits.
- `python -m app.download_models --check` only reports (exit code 1 if anything is missing); `--only trellis,sam2` limits it to some models.
- Other background models (BiRefNet full, IS-Net, U2-Net) still download the first time you choose them.

### Building the installer

On a Windows PC with the dev setup above (its `venv` with torch is used to build one wheel):

```bat
powershell -ExecutionPolicy Bypass -File installer\build_zip.ps1
powershell -ExecutionPolicy Bypass -File installer\build_zip.ps1 -Installer
```

The first makes `dist\QuokkaForm\` and `dist\QuokkaForm-<version>-portable.zip`. `-Installer` also compiles `installer\Output\QuokkaForm-Setup-<version>.exe` with [Inno Setup 6](https://jrsoftware.org/isdl.php). `installer/build_zip.py` also runs on Linux. Version and publisher come from `app/version.py` and the `#define`s at the top of `installer/QuokkaForm.iss`.

## Recommended settings

UI defaults stay conservative (for example sampling steps default to 25, Simplify to 0.95, Hard-surface off, 20,000 quads). For better edges and shape, quality over speed, these starting points have worked well in testing. They do not use extra VRAM for the extra sampling steps; they just take longer.

| | Mechanical (robots, parts, hard edges) | Organic (creatures, rounded forms) |
|---|---|---|
| Hard-surface / sharp edges | On, strength **0.6**, or **1.0** for the crispest edges | Off |
| Quad count | **30,000 or fewer** if you want quads that follow the edges (feature-aligned remesh). Use up to 100,000 if you want the most uniform detail instead | **50,000–100,000** |
| Photos | 4–8 clear views on a plain background, Quality preset **High**, if you have them | Same |
| Sampling steps (both stages) | **40–50** | **40–50** |
| Scale | Enter the real model height in mm if you know it | Same |
| AI Cleanup | Edge clean-up and shadow reduction on | Same |
| Exports | Quad OBJ, optional .3dm, and the GLB | Same |

Notes:

- Hard-surface can shrink thin tips slightly (about **1 mm** on a 100 mm model).
- The GLB is never sharpened, so you always have the original TRELLIS surface to compare.
- Simplify 0.95 is fine for a first print mesh. CAD quads already use a less-simplified source (0.8) in the background.

## Background removal and mask clean-up

TRELLIS needs the object cut out: the cut-out is cropped to 518 × 518 and placed **on black**. That black background is normal. A wrong mask is what spoils results — leftover haze or a kept backdrop inside openings becomes a grey sheet, and TRELLIS builds a surface there.

| Model | Download (first use) | CPU time per photo | Notes |
|-------|----------------------|--------------------|-------|
| BiRefNet lite (default) | ~220 MB | ~10 s | Clean masks on the test photos (white and grey backdrops) |
| BiRefNet full | ~970 MB | ~15–20 s | Slightly finer edges, slowest |
| IS-Net general | ~180 MB | < 1 s | Good on white backdrops; can keep a grey backdrop inside openings |
| U2-Net | ~180 MB | < 1 s | What early versions used; leaves haze in openings |

These models run on the CPU with onnxruntime. BiRefNet needs about 6–7 GB of system RAM while it runs; the app releases it before TRELLIS starts. If the PC has 16 GB of RAM or less and runs out of memory, use IS-Net or U2-Net with **Clean up mask**.

- **Clean up mask** (on by default): values below 50% become fully transparent, the rest fully opaque, with a slight feather. This alone removes grey haze.
- **Remove enclosed background** (on by default): large, smooth openings that match the backdrop colour are made transparent. **Background colour tolerance** (default 30) sets how close is close.
- Neither applies to a PNG that already has a transparent background: its alpha is used as is.

## Select object (SAM 2)

Use this when a hand, stand or other foreground object is in the frame. Click **keep** points on the object and **remove** points on what should go, or draw a box. **Auto-select** seeds SAM 2 from the BiRefNet mask and tries to drop a hand or sleeve automatically. Photos without clicks keep the automatic cut-out.

## Multi-image and video

Multi-image mode is still **experimental** in TRELLIS. Two conditioning modes:

- **Stochastic** (default): each sampling step is conditioned on one photo in turn. Same speed and VRAM as one image.
- **Multidiffusion**: every step averages the model's predictions for all photos. Usually more consistent, but the two sampling stages take roughly N times longer for N photos.

A walk-around video is turned into frames, then follows the same multi-image path.

## Project structure

```text
├── app/
│   ├── main.py                 # Gradio UI
│   ├── desktop.py              # Native QuokkaForm window (pywebview / WebView2)
│   ├── version.py              # App name, version 1.0.0, Expert edition, publisher
│   ├── stubs/kaolin/           # Tiny stand-in for the one kaolin function TRELLIS uses
│   ├── ui_theme.py             # Theme, CSS, logo header, favicon
│   ├── assets/                 # Logo, icon (.ico + PNG sizes)
│   ├── download_models.py      # Model preload: python -m app.download_models
│   ├── selection_ui.py         # Select-object editor callbacks
│   └── backend/
│       ├── model.py            # TRELLIS loading, generation, CAD-source simplify
│       ├── background.py       # Background models, mask clean-up, framing
│       ├── selection.py        # SAM 2 clicks and auto-select
│       ├── processor.py        # Preprocess, preview, generate, re-export
│       ├── exporter.py         # Z-up, mm scaling, decimation, quads, sharpen
│       ├── texturing.py        # Texture bake modes (fast / quality / none)
│       ├── runtime.py          # GPU memory clean-up, stage timings
│       ├── errors.py           # Plain-language error messages
│       ├── batch.py            # Batch queue, object folders, per-job settings, CSV
│       ├── scale.py            # Known scale: measure object / turntable reference
│       ├── views.py            # Quality presets, sharp + diverse view picking
│       ├── video_frames.py     # Video input: sampling, blur / duplicate filter
│       ├── cleanup.py          # AI Cleanup: colour, shadows, edge feathering
│       ├── cad_quads.py        # CAD quad OBJ (Instant Meshes, watertight all-quad)
│       ├── cad_features.py     # Feature-aligned patches, flat-face groups
│       ├── cad_rebuild.py      # Watertight rebuild of raw TRELLIS meshes
│       ├── cad_nurbs.py        # NURBS / planar faces, .3dm (rhino3dm)
│       ├── sharpen.py          # Hard-surface / sharp-edges (STL/OBJ/PLY and quads)
│       ├── history.py          # Output history
│       ├── jobs.py             # One generation and its history folder
│       ├── settings_store.py   # Remembered settings
│       ├── envcheck.py         # Startup system check
│       ├── paths.py            # Per-user data / models / logs folders
│       ├── instance.py         # Port choice, single-instance window
│       └── api.py              # Optional FastAPI backend
├── TRELLIS/                    # Cloned by start.bat (not committed)
├── installer/                  # Inno Setup script, lock file, build_zip, first-run setup
├── tests/                      # Unit tests
├── QuokkaForm.cmd              # Portable launcher
├── QuokkaForm Setup.cmd        # Repair / reinstall the runtime
├── start.bat                   # Developer launcher (browser; --desktop for the window)
├── requirements.txt
├── INSTALL_WINDOWS.txt
├── THIRD_PARTY_LICENSES.md
├── LICENCE_FINDINGS.md
├── README.md
└── LICENSE
```

## Development history

All dates below are in Australia/Perth time (AWST, UTC+8), which is also how the git history in this repo is dated.

**22 October 2026 — Alpha, "Free Image-to-3D".** The public repo started as an MIT-licensed local image-to-3D tool. A single photo went through rembg and **TripoSR**, the UI was Gradio in a browser at `http://localhost:7860`, and `start.bat` created a venv and installed dependencies. Exports were GLB, OBJ, STL and PLY. It was aimed at Windows 10/11 and about 6–8 GB of VRAM (or a CPU path in early tests). The README still called this **Alpha (v0.1)**, with TRELLIS listed only as a planned beta.

**23–24 October 2026 — TRELLIS beta.** The generator was switched to **Microsoft TRELLIS-image-large** on NVIDIA CUDA (PyTorch 2.5.1 + CUDA 12.4), after much better results on an RTX 3060 12 GB. The UI gained real TRELLIS controls (seed, sampling steps, guidance, simplify, texture size), print-ready Z-up / millimetre exports, a target triangle count, an early Instant Meshes quad OBJ, re-export without regenerating, texture bake modes (Fast / Quality / None), experimental multi-image input, Flatten base, a choice of background models (BiRefNet, IS-Net, U2-Net) with mask clean-up and Preview cutout, and an optional 360° preview video. TripoSR remained the earlier alpha path only; it is no longer the app's generator.

**25–26 October 2026 — Desktop app and branding.** SAM 2 **Select object** and Auto-select, remembered settings, an output **History** tab, **Batch** mode, friendlier errors and a startup **System check**, and a model-download page were added. The project was renamed **QuokkaForm – Images To Mesh** (publisher: Perth Resoles). A licence audit documented third-party terms. The UI moved into a **native pywebview / WebView2 window** that never opens a browser. Per-user data went to `%LOCALAPPDATA%\QuokkaForm`, with an Inno Setup installer so a system Python was no longer required.

**26 October 2026 — 0.9.0.** First Windows installer, `QuokkaForm-Setup-0.9.0.exe`: per-user, no admin, Start menu shortcut, optional desktop icon, licence page with the non-commercial notice, optional WebView2 runtime, first-run setup and model download. Models and outputs are kept on uninstall unless ticked.

**26–27 October 2026 — 0.9.1.** Fixed Windows **error 448**, where uv's Python 3.10 folder junction was blocked (often by OneDrive Files On-Demand or security software). Setup no longer needs that link.

**27 October 2026 — 0.9.2.** Fixed **"Connection errored out"** on every action in the desktop window (the web framework's job queue was not finishing startup). This was the last pre-Expert release. Later 1.0.0 installers install over 0.9.2 and reuse its runtime and models.

**1 November 2026 — macOS dropped.** A MacBook Pro port was asked about and then scrapped. TRELLIS depends on NVIDIA CUDA libraries, so the app stays Windows-only.

**3 November 2026 — 1.0.0, "QuokkaForm Images To Mesh Expert".** Built on 0.9.2 without bumping past 1.0.0 later. New in this edition: **Known scale** (measure a side, or a turntable diameter, plus optional known height), multi-image **Draft / Standard / High** presets, **walk-around video** input, **AI Cleanup**, richer turntable export (resolution, background colour, render mode, optional GIF), a **CAD quad OBJ** for Fusion 360 (then 200–20,000 quads), and Batch polish (a subfolder can be one object, videos as jobs, per-job settings, CSV report).

**3 November 2026 — CAD follow-up (still 1.0.0).** Feature-aligned quad remesh, flat-face grouping, adaptive density, and **NURBS / planar .3dm** export via rhino3dm 8.35.0 (MIT).

**11 November 2026 — CAD quality update (still 1.0.0).** The version number was deliberately left at 1.0.0. The quad target was raised to **100,000** (default 20,000). Feature-aligned remesh is capped at **30,000** quads. Raw TRELLIS meshes are rebuilt watertight before remeshing; quads come from a less-simplified source mesh (Simplify 0.8) and are projected onto the original surface; a fold-unfold pass helps thin tips. **Hard-surface / sharp edges** (bilateral normal filter, vertex update, plane snapping) was added for mechanical models. Sampling flow models are moved to RAM and the CUDA cache is cleared before decode, to keep VRAM nearer 12 GB.

## Roadmap and known limitations

What is already in 1.0.0 Expert is listed under Features above. Still open or limited:

- **Thin tips** can shrink by about 1 mm on a 100 mm model when Hard-surface is on, and a few tip folds can still remain.
- **`.3dm` fits are not yet tight.** Neighbouring faces may need *Surface > Stitch* in Fusion; some patches stay as mesh objects; on organic shapes expect mostly free-form NURBS patches. The Export info now reports an honest deviation, or **n/a** when nothing was fitted.
- **VRAM peak** on a 12 GB card is still being tuned. Texture bake **Quality** is the usual extra load (about 4 GB). Use Fast or None if you are close to the limit.
- **Feature-aligned quads** only go up to 30,000. Higher targets use uniform quads.
- **Multi-image** TRELLIS conditioning is still experimental.
- **STEP / BRep / T-spline solid export** was looked at and then **shelved**. A separate program is planned for STEP solid bodies and metrology. For now, export the quad OBJ and convert it in Fusion 360.
- **Last Builder** (a tab for a shoe-last library, measurements, photo fitting and split-last export) is an idea only; it is not in this edition.
- A **Fast / Detailed mode selector** and further low-VRAM options for 8–12 GB cards are still on the old beta list.
- The installer is **not code-signed**, so SmartScreen shows "More info > Run anyway".
- For paid / commercial use, the non-commercial GPU components (nvdiffrast and the Inria gaussian rasteriser) would need a commercial-safe swap; see [LICENCE_FINDINGS.md](LICENCE_FINDINGS.md).

## Troubleshooting

| What you see | What to try |
|---|---|
| No NVIDIA GPU / CUDA not available / CPU-only PyTorch | An NVIDIA card with a current driver is required, or run `start.bat --reinstall` / **QuokkaForm Setup (repair)** |
| Driver too old | Install NVIDIA driver 551.61 or newer (CUDA 12.4) |
| GPU too old for flash-attn | Set `ATTN_BACKEND=xformers` |
| Model download failed or was interrupted | Check the connection and retry (finished files are kept), or run `start.bat --download-models` |
| Damaged model file | Repair with `start.bat --download-models` |
| Bad image (not an image, under 64 px, nothing left after background removal) | Use a normal JPG/PNG, try another background model or Select object |
| "Connection errored out" in an older desktop window | That was the 0.9.1 bug; install **0.9.2** or **1.0.0** over the top |
| System RAM or disk full | Close other apps / free disk (BiRefNet wants about 6–7 GB of RAM while it runs) |
| SmartScreen warning | **More info > Run anyway** (the installer is not code-signed) |

At startup the app checks the PC (GPU, VRAM, NVIDIA driver, PyTorch/CUDA build, Python, RAM, free disk for the models) and prints the result in the console (`[env]` lines). The **Setup** tab shows it, and if something stops generation from working at all, a red banner at the top says so.

## License

QuokkaForm's own code is MIT. See also [LICENSE](LICENSE).

MIT License

Copyright (c) 2026 Oliver Ross Morell

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

The project is distributed free and **non-commercial** because nvdiffrast and diff-gaussian-rasterization are research / non-commercial only. See [THIRD_PARTY_LICENSES.md](THIRD_PARTY_LICENSES.md) and [LICENCE_FINDINGS.md](LICENCE_FINDINGS.md).

The GitHub repository and the Python package `app` keep their older names. Model weights are downloaded separately on first use.

On licensing of the mesh tools: the CAD remesh path (quads, feature alignment, watertight rebuild) does **not** use GPL code — MeshFix / pymeshfix is deliberately not used there. TRELLIS itself still depends on **pymeshfix 0.17.0** (GPL-3.0) for hole filling, pinned at 0.17 or lower, plus a few other GPL/LGPL packages listed in the licence documents (igraph, plyfile, easydict, and the ffmpeg binary bundled with imageio-ffmpeg).
