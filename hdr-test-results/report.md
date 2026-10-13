# Photo Merge ▸ HDR on real brackets — round 1

LightCraft **0.6.0-a7iii** (installed build, unsigned, from alvarohn/lightcraft; `lightcraft-cli 0.6.0`) · 2026-10-12 ·
first real-photo test of the HDR merge (`docs/merge.md` lists synthetic tests only). Four bracketed sets from one
shoot: three Sony ILCE-7M3 + FE 16-35 F4 ZA at 16 mm (ARW), one DJI FC3170 (Mavic Air 2, DNG). Each set was merged
by the user in the app and by `lightcraft-cli merge hdr`, and compared with the HDR DNG Lightroom made from the same
frames. All numbers are in [`results.json`](results.json).

## Summary

1. **The bug is confirmed on all four sets, in the app and the CLI: the merged HDR DNG loses the reference frame's
   lens correction.** Turning the profile correction on/off changes almost every pixel of the reference frame (mean
   6.9–9.1 of 255 levels, 95–98 % of pixels) and *nothing* in the merged HDR (the two renders are bit-identical). With
   the correction on, the HDR sits **11–13 px (median) / 24–34 px (p95)** from the corrected reference frame on the
   Sony (6000 × 4000) and **6.0 / 20.3 px** on the drone (4000 × 3000), with the radial pattern of the lens
   correction; the control (correction off on both) gives **0.15–0.77 / 0.6–2.8 px**.
2. **Lightroom's HDR DNGs carry no opcode lists at all** (no OpcodeList1/2/3), and treat the two sources differently
   (measured on the pixels): from the **Sony ARWs** the merged image keeps the **uncorrected** geometry (0.40–0.73 px
   from the uncorrected reference frame, 11–13 px from the corrected one), so Lightroom corrects it later from its own
   lens profile; from the **DJI DNGs** (whose OpcodeList3 holds `GainMap` + `WarpRectilinear`) the correction is
   **baked into the pixels** (0.26 px from the corrected reference, 6.0 px from the uncorrected one). Carrying the
   reference frame's opcode into the DNG, as planned, keeps the correction switchable in both cases; for the drone it
   should land on the same geometry Lightroom bakes in.
3. **A second bug: ARW-sourced HDRs lose the white balance and every colour tag.** LightCraft reads the reference
   ARWs as As Shot **4813 / 5268 / 3932 K**, but their merged HDRs open at **6502 K / +10** (the D65 fallback): the
   user saw a dim, orange result. Cause in the code: the ARW decoder returns `color: ColorData::default()` and keeps
   the camera white balance in `RawImage::wb_multipliers` (`crates/raw/src/vendor/arw.rs`, `RawImage { … }` near line
   731); `load_frame` keeps only `raw.color` (`crates/merge/src/frame.rs`), and `write_linear_dng` writes
   `wb_multipliers: None` (`crates/merge/src/output.rs`). The DNG ends up with no `AsShotNeutral` and no
   `ColorMatrix1` (its IFD0 has no colour tags at all; the DNG spec requires `ColorMatrix1` for colour images, so other
   DNG readers may reject or misrender it). The drone HDR is fine: its source DNG carries the tags.
4. **Merge quality against Lightroom** (both rendered by LightCraft at default settings, Lightroom's embedded edits
   suppressed): luminance agrees closely: median disagreement 0.008–0.017 EV, under 0.03 % of pixels off by more
   than 0.5 EV; shadows relative to mid-tones within 0.05 EV; recovered window highlights within 0.08–0.13 EV
   (LightCraft slightly lower). The differences found are:
   - **Magenta fringes and patches in bright windows:** 0.85 / 2.85 / 4.0 % of window pixels in LightCraft against
     ≤ 0.04 % in Lightroom. The darkest source frame rendered on its own has none (≤ 0.01 %), and in the dining room
     it is barely clipped (0.1 %), so the merge introduces them. A hypothesis: the weights near saturation (in camera
     RGB the green channel clips first).
   - **Ghosts of moving cars** with the default *Deghost: None* (drone set). *Deghost: Medium* removes them.
   - **Translation-only alignment:** the +2 EV frames differ from the reference by up to 1.0015 in scale and 0.06° in
     rotation (≤ 5.7 px at the corners). The translation model the merge chose (rms 1.6–2.8 px) leaves that in, and the
     Deghost overlay lights up those edges. The crops inspected show no visible double edges.
   - **The result depends on the input order:** same options with a different first photo (app vs CLI) gives
     bit-identical output in one set and 5 % of pixels more than one level apart (max 155) in the kitchen set.
5. **Speed:** app preview (Ctrl+H) about 3 s, app merge 10–14 s (user's estimate); CLI 12–17 s per 24 MP set, 8 s
   for the 12 MP drone set; the Deghost Medium preview takes 3.2 s. LightCraft's HDR DNGs are about 4× larger than
   Lightroom's (Sony 80–88 MB vs 18–21 MB, drone 42 MB vs 12 MB): Deflate half-float against JPEG XL.

## How it was measured

- **Sets** (chosen for the widest focal length and straight lines near the edges; there is no vertical set: all 22
  brackets in the folder are landscape, by EXIF and by Lightroom's exports). Groups were confirmed from EXIF: same
  second, same shutter, exposure compensation 0 / −2 / +2 EV (the A7 III brackets ISO at a fixed shutter) and
  −0.7 / −1.3 / 0 EV on the drone. Note: Lightroom named the last set `DSC01213-HDR.dng` after its *first* frame.

  | Set | Frames | Lightroom HDR | Why |
  |---|---|---|---|
  | `sony-01150-living` | DSC01150–01152 | DSC01152-HDR.dng | blown windows; door frame and ceiling line at the edges |
  | `sony-01159-comedor` | DSC01159–01161 | DSC01161-HDR.dng | corner windows: vertical frames at the edge, sea and sky highlights |
  | `sony-01171-cocina` | DSC01171–01173 | DSC01173-HDR.dng | long straight cabinets reaching the edges, 1/50 s |
  | `dji-0107-edificios` | DJI_0107–0109 | DJI_0109-HDR.dng | buildings and roads; the DNGs carry their own `WarpRectilinear` |

- **Copies:** each set (sources + Lightroom HDR) was copied to `D:\lightcraft-hdr-test\<set>\`, with a second copy of
  the sources in `<set>\cli\` for the CLI merges; every copy was SHA-256 checked against the original. Originals,
  Lightroom and its catalog were not touched.
- **Merges:** the user merged each set in the app (Auto Align on, Deghost None: the defaults). The CLI merged the
  `cli\` copies with `lightcraft-cli merge hdr` (same defaults). Both chose the 0 EV frame as the reference (the
  −0.7 EV one on the drone).
- **Renders:** `lightcraft-cli render` to full-size 16-bit TIFF, with `{"optics": {"lens_profile": true,
  "profile_distortion": 100}}` (ON) or `lens_profile: false` (OFF). **Every rendered file is a copy with an empty XMP
  sidecar next to it**: a sidecar wins over the file's embedded XMP (`crates/engine/src/import.rs`, the `packet = …
  find_sidecar … .or_else(embedded)` lines), so the edits Lightroom stored inside its DNG are not applied. Checked with
  `develop.get`: with the empty sidecar every file starts with all Light controls at 0, an uncropped frame and no
  masks; without it, Lightroom's DNG would bring non-zero Light values, a crop and two masks (the user's edits).
- **Geometry (4b):** SIFT on a 3000 px greyscale copy (CLAHE), matches with a 0.75 ratio test, RANSAC similarity
  (threshold 1 % of the width, to drop mismatches without clipping real distortion), then a least-squares refit;
  residuals in full-size pixels, also by distance from the centre ("radial" = signed component along the radius).
  "Core" = the points within 5× the median residual. Same approach as `tools/sony_distortion_check.py` on
  `claude/keen-heisenberg-szgpjb`.
- **Tone (4d):** regions taken from the reference frame's render: *windows* = pixels with any channel ≥ 99 % (opened
  7 px, 3 % border excluded), *shadows* = darkest 10 %, *mids* = 40th–60th percentile; median log2 of linear
  luminance per region, on renders at default settings and at −3 EV (where no window is clipped). At default settings
  most window pixels are clipped in both renders, so the "windows − mids" and "window local SD" values of those rows
  only reflect the clip: compare the −3 EV rows. Lightroom's HDR is
  first warped onto LightCraft's with a radial lens model + similarity fitted on SIFT matches (0.21–0.35 px median
  after the fit), which the drone set needs. *Disagreement* = per-pixel |Δ log2 luminance| after removing the median
  offset, 4 px Gaussian blur, −3 EV renders. *Magenta* = window pixels at −3 EV where G < 0.85 · min(R, B) (linear).
- **4c:** the DNG tags were read with a small TIFF reader written for this test: structure only (see the clean-room
  note).
- The scripts stay in the local work folder (`D:\lightcraft-hdr-test\tools`); renders and crops are not published.

## Results per set

### sony-01150-living

| Frame | Time | Shutter | f | ISO | EV comp | Focal | Lens |
|---|---|---|---|---|---|---|---|
| DSC01150.ARW (ref) | 2025:11:08 12:17:51 | 1/160 | 8.0 | 500 | +0.0 | 16 mm | FE 16-35mm F4 ZA OSS |
| DSC01151.ARW | 2025:11:08 12:17:51 | 1/160 | 8.0 | 125 | -2.0 | 16 mm | FE 16-35mm F4 ZA OSS |
| DSC01152.ARW | 2025:11:08 12:17:51 | 1/160 | 8.0 | 2000 | +2.0 | 16 mm | FE 16-35mm F4 ZA OSS |

CLI merge: 12.3 s, 6000×4000, EV measured [0.0, -1.994, 1.983] (EXIF [0.0, -2.0, 2.0]), alignment translation 1.84 px, translation 1.87 px.

**4a — correction on vs off** (8-bit levels)

| File | mean | p99 | max | pixels > 1 level | identical |
|---|---|---|---|---|---|
| reference frame | 6.89 | 72.02 | 255 | 97.0 % | no |
| LightCraft HDR (CLI) | 0.00 | 0.00 | 0 | 0.0 % | yes |
| LightCraft HDR (app) | 0.00 | 0.00 | 0 | 0.0 % | yes |
| Lightroom HDR | 0.00 | 0.00 | 0 | 0.0 % | yes |

**4b — geometry** (px of the full-size image; similarity fit)

| Comparison | median | p95 | core median / p95 | scale | rotation | shift (x, y) | points |
|---|---|---|---|---|---|---|---|
| ref ON ↔ CLI HDR ON | 10.95 | 33.93 | 10.93 / 30.91 | 1.00147 | -0.016° | +5.52, -2.78 | 4196 |
| ref ON ↔ app HDR ON | 10.90 | 31.59 | 10.90 / 29.07 | 1.00140 | -0.017° | +5.45, -2.73 | 4193 |
| ref OFF ↔ CLI HDR OFF (control) | 0.41 | 1.88 | 0.39 / 1.33 | 1.00013 | +0.003° | +0.25, +0.02 | 4499 |
| ref OFF ↔ app HDR OFF (control) | 0.41 | 1.89 | 0.39 / 1.32 | 1.00011 | +0.004° | +0.21, +0.04 | 4494 |
| ref ON ↔ ref OFF (the correction itself) | 11.75 | 32.02 | 11.75 / 31.39 | 1.00078 | +0.003° | +3.70, -2.38 | 5914 |

ref ON ↔ CLI HDR ON by radius (0 = centre, 1 = corner): 0.00-0.25: 9.0 px (+8.1 radial), 0.25-0.50: 12.7 px (+11.7 radial), 0.50-0.75: 6.7 px (-4.7 radial), 0.75-0.90: 38.4 px (-39.3 radial), 0.90-1.00: 53.4 px (-43.1 radial)

Lightroom HDR geometry: vs ref OFF median 0.40 / p95 2.18 px; vs ref ON median 10.97 / p95 29.03 px.

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
App vs CLI HDR (renders, correction off): mean 0.024, max 29 levels, 0.03 % of pixels > 1 level.

- App: `DSC01152-HDR-2.dng` (the app took DSC01152 as the first photo). The user's screenshot of it: dim and orange, As Shot 6502 K / +10 (the white-balance bug above).
- At −3 EV both merges recover the clouds and the buildings in the windows with similar detail; LightCraft's is warmer (white balance). Magenta: a few fringes.

### sony-01159-comedor

| Frame | Time | Shutter | f | ISO | EV comp | Focal | Lens |
|---|---|---|---|---|---|---|---|
| DSC01159.ARW (ref) | 2025:11:08 12:21:40 | 1/640 | 8.0 | 500 | +0.0 | 16 mm | FE 16-35mm F4 ZA OSS |
| DSC01160.ARW | 2025:11:08 12:21:40 | 1/640 | 8.0 | 125 | -2.0 | 16 mm | FE 16-35mm F4 ZA OSS |
| DSC01161.ARW | 2025:11:08 12:21:40 | 1/640 | 8.0 | 2000 | +2.0 | 16 mm | FE 16-35mm F4 ZA OSS |

CLI merge: 17.4 s, 6000×4000, EV measured [0.0, -1.994, 1.986] (EXIF [0.0, -2.0, 2.0]), alignment translation 1.56 px, translation 2.19 px.

**4a — correction on vs off** (8-bit levels)

| File | mean | p99 | max | pixels > 1 level | identical |
|---|---|---|---|---|---|
| reference frame | 8.98 | 103.14 | 255 | 98.1 % | no |
| LightCraft HDR (CLI) | 0.00 | 0.00 | 0 | 0.0 % | yes |
| LightCraft HDR (app) | 0.00 | 0.00 | 0 | 0.0 % | yes |
| Lightroom HDR | 0.00 | 0.00 | 0 | 0.0 % | yes |

**4b — geometry** (px of the full-size image; similarity fit)

| Comparison | median | p95 | core median / p95 | scale | rotation | shift (x, y) | points |
|---|---|---|---|---|---|---|---|
| ref ON ↔ CLI HDR ON | 12.88 | 24.36 | 12.88 / 24.36 | 1.00309 | -0.088° | +2.96, -3.72 | 3857 |
| ref ON ↔ app HDR ON | 12.88 | 24.36 | 12.88 / 24.36 | 1.00309 | -0.088° | +2.96, -3.72 | 3857 |
| ref OFF ↔ CLI HDR OFF (control) | 0.62 | 2.83 | 0.59 / 1.91 | 0.99995 | -0.043° | -0.27, -0.81 | 3923 |
| ref OFF ↔ app HDR OFF (control) | 0.62 | 2.83 | 0.59 / 1.91 | 0.99995 | -0.043° | -0.27, -0.81 | 3923 |
| ref ON ↔ ref OFF (the correction itself) | 14.26 | 24.31 | 14.26 / 24.31 | 1.00374 | -0.050° | +2.55, -3.31 | 6002 |

ref ON ↔ CLI HDR ON by radius (0 = centre, 1 = corner): 0.00-0.25: 13.0 px (+11.9 radial), 0.25-0.50: 15.3 px (+14.8 radial), 0.50-0.75: 7.4 px (-3.8 radial), 0.75-0.90: 25.7 px (-26.4 radial)

Lightroom HDR geometry: vs ref OFF median 0.50 / p95 2.47 px; vs ref ON median 13.00 / p95 24.62 px.

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

- App: `DSC01159-HDR.dng`, bit-identical to the CLI result (same first photo, same order).
- Magenta fringes run along the chandelier's thin wires against the sky; the −2 EV frame alone and Lightroom's HDR show the wires clean. The chandelier did not move (no doubled wires), although the Deghost overlay marks it.

### sony-01171-cocina

| Frame | Time | Shutter | f | ISO | EV comp | Focal | Lens |
|---|---|---|---|---|---|---|---|
| DSC01171.ARW (ref) | 2025:11:08 12:28:14 | 1/50 | 8.0 | 640 | +0.0 | 16 mm | FE 16-35mm F4 ZA OSS |
| DSC01172.ARW | 2025:11:08 12:28:14 | 1/50 | 8.0 | 160 | -2.0 | 16 mm | FE 16-35mm F4 ZA OSS |
| DSC01173.ARW | 2025:11:08 12:28:14 | 1/50 | 8.0 | 2500 | +2.0 | 16 mm | FE 16-35mm F4 ZA OSS |

CLI merge: 14.0 s, 6000×4000, EV measured [0.0, -1.979, 1.999] (EXIF [0.0, -2.0, 1.966]), alignment translation 2.28 px, translation 2.77 px.

**4a — correction on vs off** (8-bit levels)

| File | mean | p99 | max | pixels > 1 level | identical |
|---|---|---|---|---|---|
| reference frame | 8.00 | 72.41 | 255 | 97.6 % | no |
| LightCraft HDR (CLI) | 0.00 | 0.00 | 0 | 0.0 % | yes |
| LightCraft HDR (app) | 0.00 | 0.00 | 0 | 0.0 % | yes |
| Lightroom HDR | 0.00 | 0.00 | 0 | 0.0 % | yes |

**4b — geometry** (px of the full-size image; similarity fit)

| Comparison | median | p95 | core median / p95 | scale | rotation | shift (x, y) | points |
|---|---|---|---|---|---|---|---|
| ref ON ↔ CLI HDR ON | 11.28 | 23.67 | 11.27 / 23.60 | 1.00343 | +0.067° | +1.06, -6.14 | 7763 |
| ref ON ↔ app HDR ON | 11.30 | 23.70 | 11.30 / 23.69 | 1.00344 | +0.065° | +0.87, -6.18 | 7680 |
| ref OFF ↔ CLI HDR OFF (control) | 0.77 | 2.51 | 0.75 / 2.11 | 1.00102 | -0.020° | -0.00, -0.69 | 7656 |
| ref OFF ↔ app HDR OFF (control) | 0.76 | 2.48 | 0.74 / 2.08 | 1.00101 | -0.021° | -0.11, -0.70 | 7626 |
| ref ON ↔ ref OFF (the correction itself) | 12.13 | 23.86 | 12.13 / 23.86 | 1.00349 | +0.080° | +0.46, -6.14 | 10810 |

ref ON ↔ CLI HDR ON by radius (0 = centre, 1 = corner): 0.00-0.25: 13.0 px (+11.9 radial), 0.25-0.50: 14.5 px (+13.8 radial), 0.50-0.75: 6.7 px (-3.2 radial), 0.75-0.90: 30.8 px (-32.4 radial)

Lightroom HDR geometry: vs ref OFF median 0.73 / p95 2.39 px; vs ref ON median 11.94 / p95 24.24 px.

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
App vs CLI HDR (renders, correction off): mean 0.233, max 155 levels, 5.00 % of pixels > 1 level.

- App: `DSC01173-HDR-2.dng` (first photo DSC01173). It differs from the CLI result on 5 % of pixels: e.g. the granite texture is slightly softer in the app's merge; same reference frame, same options.
- Magenta on the sunlit blind slats and a patch in the window at −3 EV; Lightroom's window is neutral there. The +2 EV frame is scaled 1.0015 against the reference (about 5.7 px at the corners), which the translation model leaves; the Deghost Medium overlay marks the long cabinet edges and the blinds.

### dji-0107-edificios

| Frame | Time | Shutter | f | ISO | EV comp | Focal | Lens |
|---|---|---|---|---|---|---|---|
| DJI_0107.DNG (ref) | 2025:11:08 13:08:04 | 1/1250 | 2.8 | 100 | -0.7 | 4.5 mm | FC3170 |
| DJI_0108.DNG | 2025:11:08 13:08:05 | 1/2000 | 2.8 | 100 | -1.3 | 4.5 mm | FC3170 |
| DJI_0109.DNG | 2025:11:08 13:08:05 | 1/800 | 2.8 | 100 | +0.0 | 4.5 mm | FC3170 |

CLI merge: 8.0 s, 4000×3000, EV measured [0.0, -0.639, 0.684] (EXIF [0.0, -0.678, 0.644]), alignment translation 0.55 px, translation 0.56 px.

**4a — correction on vs off** (8-bit levels)

| File | mean | p99 | max | pixels > 1 level | identical |
|---|---|---|---|---|---|
| reference frame | 9.11 | 83.34 | 255 | 95.1 % | no |
| LightCraft HDR (CLI) | 0.00 | 0.00 | 0 | 0.0 % | yes |
| LightCraft HDR (app) | 0.00 | 0.00 | 0 | 0.0 % | yes |
| Lightroom HDR | 0.00 | 0.00 | 0 | 0.0 % | yes |

**4b — geometry** (px of the full-size image; similarity fit)

| Comparison | median | p95 | core median / p95 | scale | rotation | shift (x, y) | points |
|---|---|---|---|---|---|---|---|
| ref ON ↔ CLI HDR ON | 6.03 | 20.28 | 5.88 / 17.62 | 1.00671 | -0.004° | +0.22, -0.93 | 12953 |
| ref ON ↔ app HDR ON | 6.01 | 20.32 | 5.88 / 17.59 | 1.00671 | -0.004° | +0.22, -0.92 | 12984 |
| ref OFF ↔ CLI HDR OFF (control) | 0.15 | 0.63 | 0.15 / 0.60 | 1.00001 | -0.001° | -0.00, -0.01 | 14880 |
| ref OFF ↔ app HDR OFF (control) | 0.15 | 0.63 | 0.15 / 0.60 | 1.00001 | -0.001° | -0.00, -0.01 | 14897 |
| ref ON ↔ ref OFF (the correction itself) | 6.04 | 20.25 | 5.90 / 17.87 | 1.00681 | -0.005° | +0.18, -0.97 | 13729 |

ref ON ↔ CLI HDR ON by radius (0 = centre, 1 = corner): 0.00-0.25: 2.1 px (-1.9 radial), 0.25-0.50: 4.3 px (+4.3 radial), 0.50-0.75: 8.5 px (+4.5 radial), 0.75-0.90: 21.4 px (-22.2 radial), 0.90-1.00: 21.7 px (+8.5 radial)

Lightroom HDR geometry: vs ref OFF median 6.02 / p95 20.54 px; vs ref ON median 0.26 / p95 0.96 px.

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

- App: `DJI_0109-HDR-2.dng` (first photo DJI_0109); practically the same as the CLI result.
- No window highlights in this scene (nothing clipped in the reference), so the highlight columns are empty.
- Lightroom's HDR has the drone's distortion correction baked in (scale 0.993 against LightCraft's), so it was aligned with the radial model before comparing tone.
- **Ghosts:** cars moving between the three frames (1 s) come out smeared or doubled in LightCraft's HDR with the default Deghost None; Lightroom's HDR shows them sharp (which Deghost setting was used in Lightroom is unknown). A CLI merge with `--deghost medium` (10 s, `cli/DJI_0107-HDR-2.dng`) shows them sharp, at the reference frame's position; the Deghost Medium preview marks exactly those cars.

## Lightroom's HDR DNG (4c, structure only)

| | Sony sets (3 files) | Drone set |
|---|---|---|
| DNG version | 1.7 | 1.7 |
| Main image | 6000 × 4000, LinearRaw, 3 samples | 4000 × 3000, LinearRaw, 3 samples |
| Samples | 16-bit **float** | 16-bit float |
| Compression | JPEG XL | JPEG XL |
| DefaultCrop | origin 0, 0; size = whole image | same |
| BaselineExposure | 2.35 / 2.35 / 2.33 | 0.66 |
| OpcodeList1 / 2 / 3 | **none** | **none** (the source DNGs have OpcodeList3: GainMap, WarpRectilinear) |
| Colour / profile tags present | AsShotNeutral, CameraCalibration1/2, ColorMatrix1/2, ForwardMatrix1/2, ProfileName, ProfileEmbedPolicy, ProfileHueSatMapDims, ProfileLookTableDims | AsShotNeutral, ColorMatrix1/2, ProfileName, ProfileEmbedPolicy, ProfileHueSatMapDims, ProfileToneCurve |
| Other images | full-size transparency mask (NewSubFileType 4); reduced-resolution previews 1024, 2048, 512, 256 px (type 1) with matching reduced masks (type 5); 256 px thumbnail in IFD0 | same layout |
| Embedded XMP | yes, with develop settings (the user's edits) | yes, with develop settings |

LightCraft's HDR for comparison: DNG 1.4, LinearRaw, 3 × 16-bit float, Deflate tiles, DefaultCrop = whole image,
BaselineExposure 1.96 / 1.44 / 2.09 (Sony) and 1.01 (drone), no opcode lists; colour tags: **none** on the Sony
files, AsShotNeutral + ColorMatrix1/2 + ProfileHueSatMapDims + ProfileToneCurve on the drone file (carried from the
source); capture EXIF only in XMP (no Exif IFD).

The geometry Lightroom's files have (from the pixels): Sony = uncorrected reference geometry, drone = corrected (see
the per-set lines "Lightroom HDR geometry").

## Other observations

- **Import message in the app:** while importing the test folders (including Lightroom's HDRs) the user saw an import
  error ("something could not be imported") but could not read it, and the log (`%APPDATA%\LightCraft\logs\
  lightcraft.log`) only holds start-up lines, so the message was not logged. Importing each Lightroom HDR and each
  LightCraft HDR with the CLI works, and both Lightroom HDRs render (the JPEG XL DNG 1.7 files decode). Most likely
  a duplicate notice (the `cli\` copies have the same content as the main copies, and the merge result is already
  imported). Not reproduced. If such messages matter, logging the import report's failures would help.
- **Output names:** the app writes `<first selected>-HDR.dng`. The user's selection put DSC01152, DSC01173 and
  DJI_0109 first, whose `-HDR.dng` names Lightroom's files already had, so the app wrote `…-HDR-2.dng` and replaced
  nothing (`write_new_unique`), as designed.
- **Defaults:** the app applies Auto Settings to the merged photo in the catalog (Exposure +0.18, Contrast +68,
  Highlights −66, Shadows +52, Vibrance +30 on the living-room set, per the user's screenshot). The DNG files carry no
  develop settings, and all renders here use the defaults.

## What round 2 should show

With the build that carries the reference frame's corrections into the DNG, for the same four sets:

- **4a:** the HDR renders ON and OFF differ about as much as the reference frame's do (mean around 7–9 levels, more
  than 95 % of pixels).
- **4b:** reference ON ↔ HDR ON drops to about the control (median ≤ 0.8 px, p95 ≤ 2.8 px); on the drone, LightCraft's
  HDR ON should also match Lightroom's baked geometry.
- If the same build also carries the white balance: the HDR's As Shot should read close to the reference ARW's
  (4813 / 5268 / 3932 K).

## Clean-room note

Lightroom's HDR DNGs were only read as files: image sizes, sample format, compression, DefaultCrop,
BaselineExposure, which tags and which opcode *types* exist. No coefficient values, lens-profile data or
camera-profile data were recorded. The As Shot temperature LightCraft derives from Lightroom's files (it depends on
their colour matrices) is left out too. Lightroom, its catalog and the original photos were not touched; no photos,
DNGs or crops are committed.
