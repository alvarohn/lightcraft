# Photo Merge ▸ HDR on real brackets — round 2b (colour of ARW merges)

LightCraft **0.7.1-a7iii-hdr**, release `a7iii-hdr-20261013-eece99b` of alvarohn/lightcraft: round 2's build
plus storytold/lightcraft#1123 (raws without colour matrices keep their colour in merges, #965). 2026-10-13. The three
Sony sets of rounds 1–2, merged again with `lightcraft-cli merge hdr` (defaults) from fresh copies in `<set>\cli3\`.
The user's app merge of the kitchen with this build (`DSC01171-HDR-2.dng`) is byte-identical to the CLI's. Numbers in
[`results-2b.json`](results-2b.json); method as in [`report.md`](report.md), colour method below.

## Summary

1. **The rendered colour is fixed.** At default settings the merged HDR renders like its reference ARW: CIEDE2000
   median **0.41 / 0.82 / 0.50** (p90 1.07 / 1.74 / 1.53), mean Δa\* and Δb\* within ±0.35, ΔL\* within −0.15.
   Rendered by the same build, the round-2 HDRs (no colour tags) give 7.41 / 11.36 / 7.81 (p90 17.6–19.6): yellow
   (Δb\* +4.4 to +10.3) and darker (ΔL\* −7 to −11).
2. **The DNG now carries colour tags:** `ColorMatrix1` with `CalibrationIlluminant1` = 21 (D65), `ForwardMatrix1`,
   `AsShotNeutral`, a 72 × 5 × 5 `ProfileHueSatMap` and a `ProfileToneCurve` (LightCraft's own values, in the JSON).
   The lens correction from round 2 is still there (`OpcodeList3: WarpRectilinear`).
3. **But the As Shot white the DNG declares is off.** `ColorMatrix1⁻¹ · AsShotNeutral` lands at xy (0.294, 0.387),
   (0.286, 0.379) and (0.313, 0.408): about 6950 / 7350 / 6125 K with **Duv ≈ +0.038**, far above the Planckian locus
   (real light sources are within about ±0.01). LightCraft therefore shows the HDRs at **As Shot 6913 K / +115,
   7307 K / +117, 6113 K / +115**, where the reference ARWs read 4813 / +8, 5268 / +8 and 3932 / +7. That is the
   "Tint +115" the user noticed in the app.
4. **So any white-balance edit is offset.** The picture is right only while the white balance stays at As Shot. With
   the same Temp/Tint set on the HDR and on its reference ARW, the HDR renders clearly green:

   | Set | 5000 K, tint 0: ΔE00 median / p90, Δa\* | 3200 K, tint 0: ΔE00 median / p90, Δa\* |
   |---|---|---|
   | sony-01150-living | 10.90 / 13.68, −10.5 | 11.80 / 14.00, −10.0 |
   | sony-01159-comedor | 12.45 / 14.60, −13.4 | 13.13 / 14.66, −14.4 |
   | sony-01171-cocina | 13.10 / 15.18, −13.6 | 10.93 / 14.55, −10.9 |

   Copying the white balance from the ARW, picking a preset, or typing a temperature all give a green HDR.
5. **A likely cause, from reading the code (not verified):** `dng_colour` in `crates/engine/src/camera_preview.rs`
   writes `ColorMatrix1 = (bradford(D50 → t.white_xy) · forward · diag(1/neutral))⁻¹`, with `forward` scaled so that
   the *mean* of `m · [1, 1, 1]` is 1 (`m` = the fitted look's matrix × the camera transform). Then
   `ColorMatrix1⁻¹ · AsShotNeutral = bradford(D50 → t.white_xy) · forward · [1, 1, 1]`, which equals `t.white_xy` only
   if `m` maps camera white to a neutral. If the look's matrix tints white, the declared white moves off `t.white_xy`,
   and the render still looks right because it uses `ForwardMatrix1` and `AsShotNeutral` only. A regression test could
   assert that a merged DNG's As Shot temp/tint equals its reference frame's (here they differ by about 2000–2200 K and
   ~108 tint units).

## Per set

| Set | ref ARW As Shot | HDR As Shot (LightCraft) | declared white xy → CCT, Duv | ΔE00 default (median / p90) | round-2 HDR ΔE00 | CLI merge |
|---|---|---|---|---|---|---|
| sony-01150-living | 4813 K / +8 | 6913 K / +115 | (0.2939, 0.3866) → 6947 K, +0.0383 | 0.41 / 1.07 | 7.41 / 17.63 | 19.2 s |
| sony-01159-comedor | 5268 K / +8 | 7307 K / +117 | (0.2861, 0.3789) → 7349 K, +0.0389 | 0.82 / 1.74 | 11.36 / 17.64 | 63.0 s* |
| sony-01171-cocina | 3932 K / +7 | 6113 K / +115 | (0.3131, 0.4081) → 6125 K, +0.0385 | 0.50 / 1.53 | 7.81 / 19.56 | 53.8 s* |

\* The PC was at 98 % CPU during those two merges (an unrelated `find` over the whole disk from another session). The
dining-room merge run again took 19.5 s and produced a byte-identical file. These timings are not comparable with
rounds 1–2.

In the reference's neutral areas (C\*ab < 6) the new HDR's mean a\*, b\* match the reference's within about 0.2; the round-2
HDR was at b\* +6 to +16 there.

## Colour method

- Renders: `lightcraft-cli render --size 1500`, 16-bit TIFF, default settings (or `--set wb.temp=… --set wb.tint=…`),
  each file staged with an empty XMP sidecar (as in round 1).
- ΔE: CIEDE2000 on 4 × 4-pixel block averages, sRGB → Lab (D65), over the reference's mid-tone blocks (L\* 20–85)
  with no clipped pixel in either render and a 4 % border left out.
- Declared white: xy of `ColorMatrix1⁻¹ · AsShotNeutral` (single illuminant, D65); CCT by McCamy's formula, Duv as the
  distance in u′v′ (×2/3) from the Planckian locus (Krystek's approximation), positive above the locus (green).

## Still open

1. **The As Shot white of ARW-sourced HDRs** (this report): the render is right, but the white balance controls are
   offset by about +108 tint and +2000 to +2200 K, so editing the white balance turns the HDR green.
2. From round 1, unchanged: magenta fringes in bright windows, ghosts with the default Deghost None, translation-only
   alignment, order dependence, DNG size.

Clean-room: only LightCraft-made files were read for tag values; nothing from Lightroom was used in this round.
