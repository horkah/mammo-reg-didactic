# Multi-Scale Mammogram Registration — Didactic Example

A self-contained Jupyter notebook that registers a left mammogram to a right one
(or vice versa) using a **coarse-to-fine Gaussian pyramid** and **gradient-free
optimization**. Every step is visible — no specialized registration library is
hidden behind the scenes.

## What it does

Bilateral mammography produces left/right image pairs of the same patient.
Comparing regional breast texture between the two sides is a useful screening
feature, but the images are mirror-images of each other and were acquired
separately, so they must be aligned first.

This notebook:

1. Mirrors the moving image horizontally so both breasts point the same way.
2. Builds a Gaussian image pyramid (6 levels → 32× downsampling at the
   coarsest level).
3. Estimates a transform at the coarsest level using Powell's method on mean
   squared error.
4. Refines the estimate level-by-level, using the previous result as the warm
   start.
5. Applies the final transform and shows fixed / moving / warped / difference
   panels.

A single `MODE` flag selects between **rigid** (translation + rotation, 3
parameters) and **affine** (6 parameters).

## Why the coarse-to-fine strategy helps

At full resolution a mammogram can be 3000 × 2000 pixels. The MSE surface is
highly non-convex: small misalignments look like large gradients, and the
optimizer easily gets trapped. At 32× downsampling the image is only ~90 × 60
pixels — the cost surface is much smoother and the basin of attraction of the
global minimum is wide enough for Powell to find it reliably. Each finer level
then refines the solution within that basin.

## Dependencies

| Package | Purpose |
|---------|---------|
| `numpy` | array maths |
| `scipy` | `gaussian_filter`, `affine_transform`, `minimize` |
| `matplotlib` | visualization |
| `Pillow` | loading PGM / PNG / JPEG / TIFF images |
| `pydicom` | *(optional)* loading DICOM files |

Install with:

```bash
pip install numpy scipy matplotlib Pillow pydicom
```

## Supported image formats

`load_image()` accepts:

- **PGM** (P2 ASCII and P5 binary) — common for raw mammogram datasets such as
  DDSM and VinDr-Mammo
- **PNG, JPEG, TIFF, BMP** — via PIL/Pillow
- **DICOM (`.dcm`)** — via `pydicom`

> **Why not `matplotlib.image.imread`?**  
> Matplotlib's reader does not support PGM. The previous version of the notebook
> used it as a fallback and silently crashed on any `.pgm` file. The fix
> replaces that fallback with PIL/Pillow, which handles PGM natively.

## Quick start

### With your own images

Edit the two path constants at the top of the configuration cell:

```python
FIXED_PATH  = '/path/to/right.pgm'   # reference image
MOVING_PATH = '/path/to/left.pgm'    # will be mirrored & registered
```

### Offline / smoke test (no real mammograms needed)

Uncomment the two fabricated-data lines in the loading cell:

```python
fixed  = np.zeros((3000, 2000), np.float32)
fixed[800:2400, 400:1600] = np.linspace(0, 1, 1600)[None, :]
moving = np.roll(np.fliplr(fixed), shift=(40, -25), axis=(0, 1))
```

This creates a synthetic breast phantom with a known 40 px / −25 px shift so
you can verify that the optimizer recovers the ground-truth transform.

### On Kaggle

The notebook was designed to run on Kaggle. Set `FIXED_PATH` and `MOVING_PATH`
to the dataset input paths and hit *Run All*.

## Tuning tips

| Problem | Try |
|---------|-----|
| Optimizer stuck in local minimum | Increase `N_LEVELS` (e.g. 7–8) |
| Slow on large images | Reduce `N_LEVELS` or crop to breast ROI |
| Cost rises at the finest level | More levels, or switch `MODE` to `'rigid'` |
| Images from different scanners | Replace MSE with mutual information |

## Limitations and next steps

This is a **teaching tool**, not a production-grade registration pipeline.
Known limitations and suggested extensions are listed in the final notebook cell:

- **Better metric:** mutual information or normalized cross-correlation for
  cross-modality or inter-scanner pairs.
- **Breast masking:** evaluate cost only inside an Otsu-thresholded breast mask
  to avoid the air background dominating.
- **Non-rigid step:** add a B-spline or Demons stage after this affine
  alignment to capture local anatomical differences.
- **Robustness:** random restarts with the lowest final MSE.

## License

MIT License — see [`LICENSE`](LICENSE).

This code was written with heavy assistance from large language models (Claude
by Anthropic). The MIT license is the conventional choice for AI-assisted
educational and research code: it is maximally permissive, requires only
attribution, imposes no patent or copyleft obligations, and is accepted by
every major open-source ecosystem. It lets students, researchers, and
practitioners freely adapt the code while making the provenance transparent.

## Citation / attribution

If you use or adapt this notebook in your own work or course, a brief
acknowledgement is appreciated:

```
Multi-Scale Mammogram Registration — Didactic Example
https://github.com/horkah/mammo-reg-didactic
```
