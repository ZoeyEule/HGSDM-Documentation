# Performance

HGSDM's ordinary screen-space relief and side-view depth have separate costs. Side-view depth is the expensive path because it reconstructs virtual relief outside the original mesh silhouette.

## Side-View Trace Samples

Each Profile exposes **Side-View Trace Samples** under Quality:

- **128** is the full-quality default.
- **16** is the recommended balanced starting point.
- **8** is an aggressive performance setting for scenes with many overlapping side-view carriers.

Lower values affect the general carrier solver used by nonplanar terrain and faces that require blended triplanar reconstruction. Eligible planar faces use the exact solver and do not lose samples from this setting. Very low values can miss thin relief at difficult grazing views, so verify representative assets before shipping.

`r.WildMods.SSDM.SideViewDepth.CarrierSteps` is a diagnostic override. Its default value of `0` uses each Profile's setting. Values from 8 through 512 force the same budget across all active Profiles.

## Balanced rendering

When the scene is still GPU-bound, begin with 16 samples and use Unreal's TSR screen percentage between 40% and 50%. In the supplied showroom, going below 40% increased TSR upscaling cost and did not improve the result. Screen percentage and Unreal scalability settings are project-wide choices; HGSDM does not force them at runtime.

Measure in PIE or a Development build with the game window focused. Editor background pacing can mask GPU improvements with a separate frame-rate cap.
