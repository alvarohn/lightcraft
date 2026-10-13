# Photo Merge ▸ HDR on real brackets — round 2c (white balance, eefb14c)

LightCraft **0.7.1-a7iii-hdr**, release `a7iii-hdr-20261013-eefb14c` of alvarohn/lightcraft: round 2b's build plus
`eefb14c4 LR-MERGE-HDR: merged DNGs declare the source's white balance (follow-up to #1123)`. 2026-10-13. The three Sony
sets merged again with `lightcraft-cli merge hdr` (defaults) from the `cli3` copies, measured as in round 2b
([`report-2b.md`](report-2b.md)), plus a third white-balance setting and a geometry check. Numbers in
[`results-2c.json`](results-2c.json).

## Summary

1. **Not fixed: the As Shot white is still off, and white-balance edits are now greener.**

   | Set | ref ARW As Shot | HDR As Shot, 2b → 2c | declared white (2c): xy → CCT, Duv | same Temp/Tint on HDR and ARW, ΔE00 median (Δa\*), 2c | same, 2b |
   |---|---|---|---|---|---|
   | sony-01150-living | 4813 K / +8 | 6913 / +115 → **7062 / +116** | (0.2907, 0.3840) → 7100 K, +0.0388 | 5000/0: 14.1 (−14.5) · 3200/0: 19.6 (−13.5) · 6500/+10: 15.0 (−14.9) | 10.9 · 11.8 · 14.0 |
   | sony-01159-comedor | 5268 K / +8 | 7307 / +117 → **7342 / +115** | (0.2857, 0.3773) → 7383 K, +0.0384 | 18.9 (−26.0) · 20.4 (−18.3) · 19.2 (−24.0) | 12.5 · 13.1 · 15.2 |
   | sony-01171-cocina | 3932 K / +7 | 6113 / +115 → **6445 / +117** | (0.3041, 0.4003) → 6468 K, +0.0392 | 15.1 (−16.3) · 23.5 (−17.7) · 13.8 (−16.2) | 13.1 · 10.9 · 14.2 |

   The render at As Shot is unchanged and still right (ΔE00 median 0.41 / 0.82 / 0.50, p90 1.07 / 1.74 / 1.53). The
   lens correction is still there (`OpcodeList3: WarpRectilinear`; corrected HDR vs corrected reference 0.34 / 1.43,
   0.50 / 1.84, 0.79 / 2.27 px, median / p95, as in round 2).
2. **Why, very probably:** the ILCE-7M3 is one of the cameras in `lightcraft_raw::spectral`. Opened on its own, its ARW
   reads As Shot **through the spectral matrices** (`spectral::as_shot_white_through`, issue #730 in
   `docs/camera-preview-colour.md`), which gives the 3932–5268 K / +7–8 above. A white-balance edit on the ARW is then a
   Bradford adaptation on top of the fitted look. `eefb14c` builds `ColorMatrix1` from
   `xyz_to_camera(&raw.color, t.white_xy)`. But for these files the JPEG fit is accepted, so `starting_colour` returns
   before the spectral matrices are filled in: `raw.color` is the generic model, and `t.white_xy` is the generic
   model's reading of the multipliers. The DNG therefore declares the *generic* white. The same doc warns that the
   generic model reads one Sony sample as about 6829 K with a spurious tint of about 127. The new unit test passes
   because it builds `t` from the same generic `tags` it checks against. On a real ARW the white the app shows comes
   from somewhere else.
3. **What would match the source** (for the cloud session to decide): for a camera covered by `spectral`, the DNG's
   `ColorMatrix1` (and the white it implies) should come from the spectral matrices at the spectral as-shot white,
   i.e. the same reading the single ARW shows. Even then, a white-balance edit on the DNG re-develops in camera space,
   while the ARW applies a Bradford adaptation to its look, so the two will differ somewhat. A regression test on a
   real covered ARW (As Shot temp/tint of the merged DNG = the reference frame's, and a same-Temp/Tint render within a
   few ΔE00) would catch both. For raws without spectral matrices the source uses *relative* white balance, which a
   DNG cannot express; that case may need its own decision.

## Notes

- CLI merge times 20.0 / 22.4 / 22.3 s, with the PC at 81 % CPU (the same unrelated `find` from another session as in
  round 2b). Not comparable.
- Same method as round 2b. "2b" figures are the round-2b HDRs of the same sets, rendered by this build.
- Clean-room: only LightCraft-made files were read for tag values; nothing from Lightroom was used in this round.
