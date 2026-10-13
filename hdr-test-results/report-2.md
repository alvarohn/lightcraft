# Photo Merge ▸ HDR on real brackets — round 2 (the fix build)

LightCraft **0.7.1-a7iii-hdr** (`lightcraft-cli 0.7.1`), the release `a7iii-hdr-20261013-a1f3e59` of
alvarohn/lightcraft: upstream 0.7.1 + `53631935 LR-MERGE-HDR: keep the reference frame's lens corrections in HDR
merges`. It does **not** include storytold/lightcraft#1123 (raws without colour matrices keep their colour in merges),
which reached the branch (`eece99ba`) after the tag. 2026-10-13. Same four sets and same method as round 1
([`report.md`](report.md)); the CLI merged fresh copies in `<set>\cli2\`, the user merged each set again in the app
(Auto Align on, Deghost None). All numbers are in [`results-2.json`](results-2.json).

## Summary

1. **Fixed on all four sets, in the app and the CLI.** The merged DNGs now carry `OpcodeList3: WarpRectilinear`.
   They open with the lens profile on, like their source (`develop.get`: `lens_profile: true`; in round 1 it was off),
   and the correction now changes the HDR render: mean 4.5–7.9 levels, 91–95 % of pixels (round 1: 0, bit-identical).
   The reference frame changes by 6.9–9.1 levels; the HDR's smaller pixel difference comes from its darker,
   lower-contrast default render, not from a smaller warp: the geometry below shows the warp is the same.
2. **Geometry: the corrected HDR now sits on the corrected reference frame, at the level of the control.**

   | Set | ref ON ↔ HDR ON, round 1 | ref ON ↔ HDR ON, round 2 | control: ref OFF ↔ HDR OFF |
   |---|---|---|---|
   | sony-01150-living | 10.95 / 33.93 px | **0.41 / 1.83 px** | 0.41 / 1.88 px |
   | sony-01159-comedor | 12.88 / 24.36 px | **0.60 / 2.45 px** | 0.62 / 2.83 px |
   | sony-01171-cocina | 11.28 / 23.67 px | **0.76 / 2.40 px** | 0.77 / 2.51 px |
   | dji-0107-edificios | 6.03 / 20.28 px | **0.15 / 0.62 px** | 0.15 / 0.63 px |

   (median / p95, CLI merges; the app's are the same to ±0.02 px, see the tables below.)
3. **The drone HDR now matches Lightroom's geometry:** Lightroom bakes the drone's correction into its HDR, and
   LightCraft's HDR with the correction on now lands on it: **0.18 / 0.62 px** (round 1: 6 / 20 px). On the Sony sets
   Lightroom's HDR keeps the uncorrected geometry (it corrects later from its own lens profile), so it stays 11–13 px
   from LightCraft's corrected HDR, as expected.
4. **Nothing else changed.** With the correction off, the round-2 HDR renders are **bit-identical** to round 1's on
   all four sets, and the reference frames render bit-identically too. So round 1's findings on tone, magenta fringes
   (0.85 / 2.85 / 4.0 % of window pixels), ghosts with Deghost None and translation-only alignment all still hold.
5. **Still open: white balance and colour tags of ARW-sourced HDRs.** They still open at As Shot 6502 K / +10
   (reference ARWs: 4813 / 5268 / 3932 K), and the Sony HDR DNGs still have no colour tags. That is expected, since
   this build predates #1123. A build from `eece99ba` or later would need a short check (round 2b: merge, read
   `develop.get` and the tags).
6. **Order dependence, now pinned down.** App and CLI files are byte-identical when the frames come in the same order
   (living, dining room). The app passed the kitchen frames as DSC01171, DSC01173, DSC01172: a CLI merge in that order
   is byte-identical to the app's file, while the CLI's 71 / 72 / 73 order differs on 0.58 % of pixels (max 102 levels).
7. **Speed:** about 12 s per app merge (the user's estimate). The CLI took 14.3–21.7 s this round, and an idle re-run
   took 16.7 s (living) and 13.5 s (drone), against 12.3 s and 8.0 s in round 1. 0.7.1 may be slower, but the load on
   this PC was not controlled.

## Results per set

### sony-01150-living

| Frame | Time | Shutter | f | ISO | EV comp | Focal | Lens |
|---|---|---|---|---|---|---|---|
| DSC01150.ARW (ref) | 2025:11:08 12:17:51 | 1/160 | 8.0 | 500 | +0.0 | 16 mm | FE 16-35mm F4 ZA OSS |
| DSC01151.ARW | 2025:11:08 12:17:51 | 1/160 | 8.0 | 125 | -2.0 | 16 mm | FE 16-35mm F4 ZA OSS |
| DSC01152.ARW | 2025:11:08 12:17:51 | 1/160 | 8.0 | 2000 | +2.0 | 16 mm | FE 16-35mm F4 ZA OSS |

CLI merge: 21.7 s, 6000×4000, EV measured [0.0, -1.994, 1.983] (EXIF [0.0, -2.0, 2.0]), alignment translation 1.84 px, translation 1.87 px.

**4a — correction on vs off** (8-bit levels)

| File | mean | p99 | max | pixels > 1 level | identical |
|---|---|---|---|---|---|
| reference frame | 6.89 | 72.02 | 255 | 97.0 % | no |
| LightCraft HDR (CLI) | 4.52 | 54.77 | 255 | 92.9 % | no |
| LightCraft HDR (app) | 4.52 | 54.77 | 255 | 92.9 % | no |
| Lightroom HDR | 0.00 | 0.00 | 0 | 0.0 % | yes |

**4b — geometry** (px of the full-size image; similarity fit)

| Comparison | median | p95 | core median / p95 | scale | rotation | shift (x, y) | points |
|---|---|---|---|---|---|---|---|
| ref ON ↔ CLI HDR ON | 0.41 | 1.83 | 0.39 / 1.33 | 1.00018 | +0.003° | +0.23, +0.04 | 4681 |
| ref ON ↔ app HDR ON | 0.41 | 1.83 | 0.39 / 1.33 | 1.00018 | +0.003° | +0.23, +0.04 | 4681 |
| ref OFF ↔ CLI HDR OFF (control) | 0.41 | 1.88 | 0.39 / 1.33 | 1.00013 | +0.003° | +0.25, +0.02 | 4499 |
| ref OFF ↔ app HDR OFF (control) | 0.41 | 1.88 | 0.39 / 1.33 | 1.00013 | +0.003° | +0.25, +0.02 | 4499 |
| ref ON ↔ ref OFF (the correction itself) | 11.75 | 32.02 | 11.75 / 31.39 | 1.00078 | +0.003° | +3.70, -2.38 | 5914 |

ref ON ↔ CLI HDR ON by radius (0 = centre, 1 = corner): 0.00-0.25: 0.3 px (-0.2 radial), 0.25-0.50: 0.3 px (-0.0 radial), 0.50-0.75: 0.6 px (+0.1 radial), 0.75-0.90: 1.0 px (+0.2 radial), 0.90-1.00: 1.1 px (-2.8 radial)

Lightroom HDR geometry: vs ref OFF median 0.40 / p95 2.18 px; vs ref ON median 10.97 / p95 29.03 px; vs LightCraft HDR ON median 11.32 / p95 33.92 px.

**4d — tone** (median log2 linear luminance; regions from the reference frame)

| Render | mids | shadows − mids | windows − mids | windows clipped | window local SD |
|---|---|---|---|---|---|
| LightCraft, default | -4.20 | -1.80 | 4.20 | 84.7 % | 0.000 |
| Lightroom, default | -4.01 | -1.80 | 4.01 | 90.9 % | 0.000 |
| LightCraft, −3 EV | -7.17 | -1.82 | 5.18 | 0.0 % | 0.137 |
| Lightroom, −3 EV | -6.98 | -1.82 | 5.28 | 0.0 % | 0.122 |

Lightroom ↔ LightCraft after radial alignment: median 0.31 / p95 1.22 px. Disagreement at −3 EV (exposure-normalised, 4 px blur): median 0.009 EV, > 0.5 EV on 0.00 % of pixels, > 1 EV on 0.00 %.
Magenta in the windows at −3 EV: LightCraft 0.85 %, Lightroom 0.01 % of window pixels.
Frames vs reference: DSC01151.ARW: scale 1.00000, rotation -0.008°, shift +0.58, +0.33 px (≈ 0.5 px at the corners beyond a translation); DSC01152.ARW: scale 1.00027, rotation +0.013°, shift +1.40, +0.35 px (≈ 1.3 px at the corners beyond a translation).
App vs CLI HDR (renders, correction off): mean 0.000, max 0 levels, 0.00 % of pixels > 1 level (bit-identical).

- App: `DSC01150-HDR.dng` (first photo DSC01150, as the CLI): byte-identical to the CLI's `cli2/DSC01150-HDR.dng`.

### sony-01159-comedor

| Frame | Time | Shutter | f | ISO | EV comp | Focal | Lens |
|---|---|---|---|---|---|---|---|
| DSC01159.ARW (ref) | 2025:11:08 12:21:40 | 1/640 | 8.0 | 500 | +0.0 | 16 mm | FE 16-35mm F4 ZA OSS |
| DSC01160.ARW | 2025:11:08 12:21:40 | 1/640 | 8.0 | 125 | -2.0 | 16 mm | FE 16-35mm F4 ZA OSS |
| DSC01161.ARW | 2025:11:08 12:21:40 | 1/640 | 8.0 | 2000 | +2.0 | 16 mm | FE 16-35mm F4 ZA OSS |

CLI merge: 19.2 s, 6000×4000, EV measured [0.0, -1.994, 1.986] (EXIF [0.0, -2.0, 2.0]), alignment translation 1.56 px, translation 2.19 px.

**4a — correction on vs off** (8-bit levels)

| File | mean | p99 | max | pixels > 1 level | identical |
|---|---|---|---|---|---|
| reference frame | 8.98 | 103.14 | 255 | 98.1 % | no |
| LightCraft HDR (CLI) | 5.33 | 68.39 | 255 | 93.3 % | no |
| LightCraft HDR (app) | 5.33 | 68.39 | 255 | 93.3 % | no |
| Lightroom HDR | 0.00 | 0.00 | 0 | 0.0 % | yes |

**4b — geometry** (px of the full-size image; similarity fit)

| Comparison | median | p95 | core median / p95 | scale | rotation | shift (x, y) | points |
|---|---|---|---|---|---|---|---|
| ref ON ↔ CLI HDR ON | 0.60 | 2.45 | 0.58 / 1.88 | 0.99990 | -0.040° | -0.29, -0.73 | 4025 |
| ref ON ↔ app HDR ON | 0.60 | 2.45 | 0.58 / 1.88 | 0.99990 | -0.040° | -0.29, -0.73 | 4025 |
| ref OFF ↔ CLI HDR OFF (control) | 0.62 | 2.83 | 0.59 / 1.91 | 0.99995 | -0.043° | -0.27, -0.81 | 3923 |
| ref OFF ↔ app HDR OFF (control) | 0.62 | 2.83 | 0.59 / 1.91 | 0.99995 | -0.043° | -0.27, -0.81 | 3923 |
| ref ON ↔ ref OFF (the correction itself) | 14.26 | 24.31 | 14.26 / 24.31 | 1.00374 | -0.050° | +2.55, -3.31 | 6002 |

ref ON ↔ CLI HDR ON by radius (0 = centre, 1 = corner): 0.00-0.25: 0.5 px (-0.2 radial), 0.25-0.50: 0.6 px (-0.3 radial), 0.50-0.75: 0.6 px (+0.1 radial), 0.75-0.90: 1.0 px (+0.5 radial), 0.90-1.00: 1.8 px (+0.4 radial)

Lightroom HDR geometry: vs ref OFF median 0.50 / p95 2.47 px; vs ref ON median 13.00 / p95 24.62 px; vs LightCraft HDR ON median 13.25 / p95 24.97 px.

**4d — tone** (median log2 linear luminance; regions from the reference frame)

| Render | mids | shadows − mids | windows − mids | windows clipped | window local SD |
|---|---|---|---|---|---|
| LightCraft, default | -3.69 | -1.27 | 2.78 | 7.7 % | 0.780 |
| Lightroom, default | -3.44 | -1.31 | 2.84 | 16.5 % | 0.738 |
| LightCraft, −3 EV | -6.65 | -1.30 | 2.97 | 0.0 % | 0.853 |
| Lightroom, −3 EV | -6.39 | -1.34 | 3.05 | 0.0 % | 0.828 |

Lightroom ↔ LightCraft after radial alignment: median 0.35 / p95 1.29 px. Disagreement at −3 EV (exposure-normalised, 4 px blur): median 0.017 EV, > 0.5 EV on 0.00 % of pixels, > 1 EV on 0.00 %.
Magenta in the windows at −3 EV: LightCraft 2.85 %, Lightroom 0.00 % of window pixels.
Frames vs reference: DSC01160.ARW: scale 0.99991, rotation -0.015°, shift +0.73, -1.29 px (≈ 1.0 px at the corners beyond a translation); DSC01161.ARW: scale 1.00009, rotation -0.058°, shift +1.05, -1.01 px (≈ 3.6 px at the corners beyond a translation).
App vs CLI HDR (renders, correction off): mean 0.000, max 0 levels, 0.00 % of pixels > 1 level (bit-identical).

- App: `DSC01159-HDR-2.dng`: byte-identical to the CLI's `cli2/DSC01159-HDR.dng`.

### sony-01171-cocina

| Frame | Time | Shutter | f | ISO | EV comp | Focal | Lens |
|---|---|---|---|---|---|---|---|
| DSC01171.ARW (ref) | 2025:11:08 12:28:14 | 1/50 | 8.0 | 640 | +0.0 | 16 mm | FE 16-35mm F4 ZA OSS |
| DSC01172.ARW | 2025:11:08 12:28:14 | 1/50 | 8.0 | 160 | -2.0 | 16 mm | FE 16-35mm F4 ZA OSS |
| DSC01173.ARW | 2025:11:08 12:28:14 | 1/50 | 8.0 | 2500 | +2.0 | 16 mm | FE 16-35mm F4 ZA OSS |

CLI merge: 16.6 s, 6000×4000, EV measured [0.0, -1.979, 1.999] (EXIF [0.0, -2.0, 1.966]), alignment translation 2.28 px, translation 2.77 px.

**4a — correction on vs off** (8-bit levels)

| File | mean | p99 | max | pixels > 1 level | identical |
|---|---|---|---|---|---|
| reference frame | 8.00 | 72.41 | 255 | 97.6 % | no |
| LightCraft HDR (CLI) | 5.08 | 54.83 | 255 | 94.7 % | no |
| LightCraft HDR (app) | 5.12 | 54.85 | 255 | 94.9 % | no |
| Lightroom HDR | 0.00 | 0.00 | 0 | 0.0 % | yes |

**4b — geometry** (px of the full-size image; similarity fit)

| Comparison | median | p95 | core median / p95 | scale | rotation | shift (x, y) | points |
|---|---|---|---|---|---|---|---|
| ref ON ↔ CLI HDR ON | 0.76 | 2.40 | 0.74 / 2.10 | 1.00102 | -0.019° | -0.01, -0.72 | 7835 |
| ref ON ↔ app HDR ON | 0.75 | 2.40 | 0.74 / 2.10 | 1.00104 | -0.020° | -0.07, -0.72 | 7925 |
| ref OFF ↔ CLI HDR OFF (control) | 0.77 | 2.51 | 0.75 / 2.11 | 1.00102 | -0.020° | -0.00, -0.69 | 7656 |
| ref OFF ↔ app HDR OFF (control) | 0.77 | 2.49 | 0.75 / 2.08 | 1.00101 | -0.019° | -0.02, -0.70 | 7600 |
| ref ON ↔ ref OFF (the correction itself) | 12.13 | 23.86 | 12.13 / 23.86 | 1.00349 | +0.080° | +0.46, -6.14 | 10810 |

ref ON ↔ CLI HDR ON by radius (0 = centre, 1 = corner): 0.00-0.25: 1.0 px (+0.0 radial), 0.25-0.50: 0.7 px (+0.5 radial), 0.50-0.75: 0.7 px (-0.0 radial), 0.75-0.90: 2.1 px (-1.9 radial), 0.90-1.00: 5.3 px (-5.1 radial)

Lightroom HDR geometry: vs ref OFF median 0.73 / p95 2.39 px; vs ref ON median 11.94 / p95 24.24 px; vs LightCraft HDR ON median 11.17 / p95 23.24 px.

**4d — tone** (median log2 linear luminance; regions from the reference frame)

| Render | mids | shadows − mids | windows − mids | windows clipped | window local SD |
|---|---|---|---|---|---|
| LightCraft, default | -4.05 | -1.89 | 3.96 | 59.9 % | 0.163 |
| Lightroom, default | -3.95 | -1.88 | 3.95 | 80.6 % | 0.113 |
| LightCraft, −3 EV | -7.02 | -1.91 | 4.24 | 1.7 % | 0.509 |
| Lightroom, −3 EV | -6.92 | -1.90 | 4.37 | 2.8 % | 0.466 |

Lightroom ↔ LightCraft after radial alignment: median 0.27 / p95 1.05 px. Disagreement at −3 EV (exposure-normalised, 4 px blur): median 0.008 EV, > 0.5 EV on 0.01 % of pixels, > 1 EV on 0.00 %.
Magenta in the windows at −3 EV: LightCraft 4.01 %, Lightroom 0.04 % of window pixels.
Frames vs reference: DSC01172.ARW: scale 1.00056, rotation -0.020°, shift +2.23, -0.10 px (≈ 2.4 px at the corners beyond a translation); DSC01173.ARW: scale 1.00152, rotation -0.025°, shift +2.63, -0.97 px (≈ 5.7 px at the corners beyond a translation).
App vs CLI HDR (renders, correction off): mean 0.112, max 102 levels, 0.58 % of pixels > 1 level.

- App: `DSC01171-HDR.dng`, frames passed as DSC01171, DSC01173, DSC01172; byte-identical to a CLI merge in that order (`cli2/DSC01171-HDR-3.dng`). The CLI's 71 / 72 / 73 order differs on 0.58 % of pixels.
- The user first merged this set with Deghost Low by mistake (`DSC01173-HDR-3.dng`): not measured.

### dji-0107-edificios

| Frame | Time | Shutter | f | ISO | EV comp | Focal | Lens |
|---|---|---|---|---|---|---|---|
| DJI_0107.DNG (ref) | 2025:11:08 13:08:04 | 1/1250 | 2.8 | 100 | -0.7 | 4.5 mm | FC3170 |
| DJI_0108.DNG | 2025:11:08 13:08:05 | 1/2000 | 2.8 | 100 | -1.3 | 4.5 mm | FC3170 |
| DJI_0109.DNG | 2025:11:08 13:08:05 | 1/800 | 2.8 | 100 | +0.0 | 4.5 mm | FC3170 |

CLI merge: 14.3 s, 4000×3000, EV measured [0.0, -0.639, 0.684] (EXIF [0.0, -0.678, 0.644]), alignment translation 0.55 px, translation 0.56 px.

**4a — correction on vs off** (8-bit levels)

| File | mean | p99 | max | pixels > 1 level | identical |
|---|---|---|---|---|---|
| reference frame | 9.11 | 83.34 | 255 | 95.1 % | no |
| LightCraft HDR (CLI) | 7.86 | 81.66 | 255 | 90.7 % | no |
| LightCraft HDR (app) | 7.86 | 81.65 | 255 | 90.7 % | no |
| Lightroom HDR | 0.00 | 0.00 | 0 | 0.0 % | yes |

**4b — geometry** (px of the full-size image; similarity fit)

| Comparison | median | p95 | core median / p95 | scale | rotation | shift (x, y) | points |
|---|---|---|---|---|---|---|---|
| ref ON ↔ CLI HDR ON | 0.15 | 0.62 | 0.15 / 0.59 | 1.00000 | -0.001° | -0.01, +0.01 | 15187 |
| ref ON ↔ app HDR ON | 0.15 | 0.62 | 0.15 / 0.59 | 1.00001 | -0.001° | +0.00, +0.00 | 15162 |
| ref OFF ↔ CLI HDR OFF (control) | 0.15 | 0.63 | 0.15 / 0.60 | 1.00001 | -0.001° | -0.00, -0.01 | 14880 |
| ref OFF ↔ app HDR OFF (control) | 0.15 | 0.63 | 0.15 / 0.60 | 1.00001 | -0.001° | -0.00, -0.01 | 14897 |
| ref ON ↔ ref OFF (the correction itself) | 6.04 | 20.25 | 5.90 / 17.87 | 1.00681 | -0.005° | +0.18, -0.97 | 13729 |

ref ON ↔ CLI HDR ON by radius (0 = centre, 1 = corner): 0.00-0.25: 0.1 px (-0.0 radial), 0.25-0.50: 0.1 px (+0.0 radial), 0.50-0.75: 0.1 px (-0.0 radial), 0.75-0.90: 0.2 px (+0.0 radial), 0.90-1.00: 0.2 px (+0.1 radial)

Lightroom HDR geometry: vs ref OFF median 6.02 / p95 20.54 px; vs ref ON median 0.26 / p95 0.96 px; vs LightCraft HDR ON median 0.18 / p95 0.62 px.

**4d — tone** (median log2 linear luminance; regions from the reference frame)

| Render | mids | shadows − mids | windows − mids | windows clipped | window local SD |
|---|---|---|---|---|---|
| LightCraft, default | -2.75 | -4.54 | – | – | – |
| Lightroom, default | -2.72 | -4.49 | – | – | – |
| LightCraft, −3 EV | -7.62 | -2.69 | – | – | – |
| Lightroom, −3 EV | -7.60 | -2.64 | – | – | – |

Lightroom ↔ LightCraft after radial alignment: median 0.21 / p95 0.75 px. Disagreement at −3 EV (exposure-normalised, 4 px blur): median 0.008 EV, > 0.5 EV on 0.03 % of pixels, > 1 EV on 0.00 %.
Frames vs reference: DJI_0108.DNG: scale 1.00000, rotation -0.002°, shift -0.02, -0.04 px (≈ 0.1 px at the corners beyond a translation); DJI_0109.DNG: scale 1.00000, rotation -0.001°, shift -0.09, -0.14 px (≈ 0.1 px at the corners beyond a translation).
App vs CLI HDR (renders, correction off): mean 0.011, max 4 levels, 0.00 % of pixels > 1 level.

- App: `DJI_0109-HDR-3.dng` (first photo DJI_0109); practically the same as the CLI's (max 4 levels).
- **LightCraft's corrected HDR now matches Lightroom's HDR, which has the correction baked in: 0.18 / 0.62 px** (the 'vs LightCraft HDR ON' figure of the 'Lightroom HDR geometry' line).

## The DNG files of this round (4c)

CLI and app HDRs alike: DNG 1.4, LinearRaw, 3 × 16-bit float, Deflate, DefaultCrop = whole image, the same
BaselineExposure as round 1 (1.96 / 1.44 / 2.09 / 1.01), and now **`OpcodeList3: WarpRectilinear`** (the drone's
`GainMap` stays applied to the pixels, as before). Colour tags: still none on the Sony files; AsShotNeutral,
ColorMatrix1/2, ProfileHueSatMapDims and ProfileToneCurve on the drone file. The extra kitchen merge the user first ran
with Deghost Low by mistake (`DSC01173-HDR-3.dng`) also carries the opcode; it was not measured.

## Still open after this round (for the cloud session)

In order of what a user would notice first:

1. **White balance and colour tags of ARW-sourced HDRs** (round 1, item 3): probably addressed by #1123; needs a
   round 2b on a build that includes it.
2. **Magenta fringes and patches in bright windows** (round 1, item 4): unchanged (0.85 / 2.85 / 4.0 % of window
   pixels against ≤ 0.04 % in Lightroom; none in the darkest frame alone).
3. **Ghosts with the default Deghost None**: moving cars on the drone set; Deghost Medium fixes them. Possibly a
   default worth reconsidering, or a hint in the dialog.
4. **Translation-only alignment** leaves up to 5.7 px at the corners of the +2 EV frames (scale 1.0015, rotation
   0.06°).
5. **Order dependence**: same frames and options in a different order give a slightly different file.
6. **File size**: about 4× Lightroom's (Deflate half-float vs JPEG XL).

## Clean-room note

Same as round 1. Lightroom's HDR DNGs were only read as files (sizes, sample format, compression, DefaultCrop,
BaselineExposure, which tags and opcode *types* exist), and their geometry was measured from the rendered pixels. No
coefficient values, lens-profile data or camera-profile data were recorded. Lightroom, its catalog and the original
photos were not touched; no photos, DNGs or crops are committed.
