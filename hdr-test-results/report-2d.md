# Photo Merge ▸ HDR on real brackets — round 2d (white balance, 45e02b6)

LightCraft **0.7.1-a7iii-hdr**, release `a7iii-hdr-20261013-45e02b6` of alvarohn/lightcraft: round 2c's build plus
`45e02b63 LR-MERGE-HDR: merged DNGs declare the white-balance model the source shows (covered cameras)`. 2026-10-13.
The three Sony sets merged again with `lightcraft-cli merge hdr` (defaults) from the `cli3` copies, measured as in
rounds 2b–2c ([`report-2b.md`](report-2b.md), [`report-2c.md`](report-2c.md)). The user's app merge of the dining room
with this build (`DSC01159-HDR-3.dng`) is byte-identical to the CLI's. Numbers in [`results-2d.json`](results-2d.json).

## Summary

1. **The As Shot white is fixed.** The merged HDRs open at the reference ARW's As Shot exactly: **4813 K / +8,
   5268 K / +8, 3932 K / +7** (round 2c: 7062 / +116, 7342 / +115, 6445 / +117). The DNGs now carry the camera's two
   spectral matrices (`ColorMatrix1/2` with `CalibrationIlluminant1/2` = 17 / 21, A and D65; `ForwardMatrix1/2`) and
   declare a white at 4818 / 5272 / 3936 K with Duv +0.003 (round 2c: Duv +0.039). That white is interpolated between
   the two matrices as the DNG spec does; with a single matrix the figure would be off.
2. **White-balance edits now behave like on the ARW for moderate moves.** Same Temp/Tint on HDR and reference ARW,
   CIEDE2000 median (p90):

   | Set (As Shot) | 5000 K / 0 | 3200 K / 0 | 6500 K / +10 | round 2c, same three |
   |---|---|---|---|---|
   | sony-01150-living (4813 K) | **1.24** (2.13) | **1.33** (2.43) | **0.91** (2.52) | 14.1 · 19.6 · 15.0 |
   | sony-01159-comedor (5268 K) | **0.77** (1.88) | 5.19 (5.93), Δa\* −5.8 | **1.74** (3.10) | 18.9 · 20.4 · 19.2 |
   | sony-01171-cocina (3932 K) | 2.71 (4.64), Δb\* +5.1 | **1.55** (2.79) | 5.12 (6.25), Δb\* +11.1 | 15.1 · 23.5 · 13.8 |

   The two cases near 5 are the largest moves away from As Shot (5268 → 3200 K and 3932 → 6500 K): the HDR comes out
   a little greener, or yellower, than the ARW. As noted in round 2c, the ARW applies a white-balance edit as a
   Bradford adaptation on top of its fitted look, while the DNG re-develops camera RGB through the spectral matrices,
   so the two models drift apart as the edit grows. Whether that is acceptable, or worth matching (for example by
   giving the merged DNG the source's adaptation), is the cloud session's call.
3. **Nothing else moved.** The render at As Shot is unchanged (CIEDE2000 median 0.41 / 0.82 / 0.50, p90 1.07 / 1.74 /
   1.53); the lens correction is still there (`OpcodeList3: WarpRectilinear`; corrected HDR vs corrected reference
   0.34 / 1.43, 0.50 / 1.84, 0.79 / 2.27 px, median / p95). CLI merges took 14.4 / 15.8 / 13.6 s, with the PC around
   40 % CPU.

## Notes

- "Round 2c" figures are the round-2c HDRs of the same sets, rendered by this build.
- The kitchen's first merge attempt failed with "could not write the merged DNG … Espacio en disco insuficiente (os
  error 112)": drive D: was full. The CLI reported it cleanly and wrote nothing (good: no crash, no partial file).
  After freeing space the merge succeeded.
- Clean-room: only LightCraft-made files were read for tag values; nothing from Lightroom was used in this round.
