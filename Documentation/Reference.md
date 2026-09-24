# Profiles and Settings Reference

## Profiles

| Control | Default / units | Behavior |
|---|---|---|
| Height Source | Profile Texture | Texture, strict authored output, or preferred authored output with valid texture fallback. |
| Texture / Material Height Direction | Black Is Raised | Choose the outward endpoint along the surface normal. |
| Displacement | 20 cm | Full black-to-white height span, not viewport pixels. |
| Base Anchored Displacement | On | 0..D outward; off gives -D/2..D/2. |
| Normalize Sharp Edges | On | Smooth source height before displacement. |
| Sharp Edge Normalization Size | 2; 1..128 texels | Canonical bank texels for textures; evaluation pixels/chart texels for authored height. Tent support extends twice the size per axis. Larger values cost more and soften detail. |
| Enable Smart Bevel | Off | Rounded authoritative texture-height preparation; Profile Texture YZ/triplanar only. No extra triangles. |
| Bevel Depth | 0 cm | Maximum vertical change per sample; 0 leaves height unchanged. Historically serialized as MirrorDepthCm; it does not make an underside. |
| Bevel Radius Override | 0 cm | 0 uses Bevel Depth as source-space radius; explicit values change filter width, capped to a repeat. |
| Enable Side-View Depth | On | Upper relief from sides, without mirroring. Texture height supports YZ/triplanar; strict authored output uses captured height. |
| Side-View Trace Samples | 128; 8..512 | Maximum samples for the general side-view carrier solver. 16 is the balanced performance setting; 8 is aggressive. Eligible exact planar faces do not use this limit. |
| Enable Mirrored Surface | Off; advanced | Reflected underside with the same source/mapping eligibility. |
| Mirror Point | 5 cm | Reflection level outward from carrier. Maximum D anchored or D/2 centered. Underside = 2*point - upper height; the maximum can collapse the shell to its highest contour. |
| Re-evaluate Mirror Side Material | Off; experimental | Retain original opaque Default Lit material at virtual positions on blendable GBuffer. Cache-safe static world-coordinate materials/MICs only; not dynamic instances, time/view/scene dependence or untracked collection edits. UV-only graphs retain their UVs; unsupported cases use captured material. |
| Height Tiling | 0.002 | World height sampling scale; UV0 has separate controls. |
| Mapping Mode / Offset | Triplanar / zero | Triplanar, World Planar YZ or Mesh UV0, plus world offset. |
| Triplanar Sharpness | 4; 0.1..32 | Axis blending sharpness. |
| Mesh UV Tiling / Offset | (1,1) / (0,0) | Authored UV0 repeat and offset. |
| Full Quality Distance | 25 m | Full profile strength through this distance. |
| Max Distance | 100 m | Smooth fade from Full Quality to physical carrier, then termination. Keep Full Quality lower. |
| Minimum Screen Displacement | 0.35 px | Subpixel parallax threshold; not centimeters or physical span. |
| Bilinear Filtering | On | Neighbor height blend; off uses nearest sampling. |
| Anisotropic Filtering | Off | Additional bounded directional taps at grazing views. |
| Lerp Speed | 1; 0.05..1 | Inverse correction scale; not a seconds-based animation, 2x speed control or temporal-AA switch. |
| Adaptive Subsampling | Off | Additional CPDAS sampling refinement. |
| Max Projected Sample Distance | 4 px | Refinement spacing while adaptive sampling is enabled. |
| Max Adaptive Subdivision Level | 3; 0..3 | Refinement depth, up to eight subdivisions per axis. |
| Minimum Shadow Displacement | 0 cm | Suppress smaller relief-shadow separations without altering visible height. Values above 1 cm resolved the reported micro-shadow artifact in the tested rock material; tune per asset. |

Legacy MinimumIslandSizeCm, bEnableMirroredUnderside and bEnableSilhouetteExpansion remain serialized but do not enable retired topology/rendering.

## Component

Assign Profile and optionally Target Primitive. **Enable SSDM Shadows** controls that component's HGSDM relief-shadow casting/receiving independently of Unreal's native shadows. **Enable Mesh Terrain Shadow Captures** is off by default, separate from automatic side-depth residency. **Allow Mesh Terrain Hardware Ray Tracing** is off by default to limit large BLAS residency; raster HGSDM and native virtual shadow maps remain available. `SetProfile` and `SetSSDMShadowsEnabled` are Blueprint-callable.

## Project settings

| Setting | Default | Purpose |
|---|---|---|
| Enable SSDM Renderer | On | Global GPU pass gate; restart. |
| Height Bank Resolution | 512 | Canonical G8 slices; 256/512/1024/2048. Higher values preserve more texture height detail at extra cost; restart. |
| Profile ID Update Mode | Every Frame | Optional camera/SSDM-transform cache for rigid targets. Dynamic/WPO cases fall back; unrelated moving occluders aren't tracked. Restart. |
| Mirror Material Pixel Budget | 1,048,576 | Shared retained side/mirror capture samples; 1,536..16,777,216. Multiple buffers and re-evaluation weights per sample. Not viewport resolution, GPU milliseconds or total VRAM cap; live. |
| Maximum Mirror Capture Resolution | 512 | Power-of-two maximum chart dimension, 16..512; rectangular charts share budget; live. |
| Field Memory Budget | 256 MB | Separate optional field retained-output budget; transient/source/staging memory is separate. |
| Field Max Captures / Readbacks Per Frame | 2 / 1 | Optional field scheduling; CPU readbacks are explicit. |
| Field Max Local Tile Resolution | 512 | Optional tile limit; powers of two 64..1024. |
| Field Idle Release Delay | 5 seconds | Optional field lifetime; persistence has separate rules. |

## Diagnostic overrides

Console names retain SSDM for compatibility. Restore temporary overrides after comparison.

| Variable | Default | Purpose |
|---|---|---|
| r.WildMods.SSDM.Enabled | 1 | Global renderer comparison toggle. |
| r.WildMods.SSDM.Shadows | 1 | HGSDM relief-shadow toggle. |
| r.WildMods.SSDM.MirrorSurface.PixelBudget | 0 | Positive value overrides project budget; 0 follows settings. |
| r.WildMods.SSDM.MirrorSurface.CaptureResolution | 0 | Positive value overrides resolution; 0 follows settings. |
| r.WildMods.SSDM.SideViewDepth.CarrierSteps | 0 | Diagnostic override for general side-view ray samples. 0 uses the Profile's **Side-View Trace Samples**; 8..512 forces a value. |
| r.WildMods.SSDM.SideViewDepth.CameraGraceCm | 25 | Near-camera margin, 0..200 cm; larger values can expand trace work. |
| r.WildMods.SSDM.SideViewDepth.TerrainMaxSources | 4 | Fine terrain source cap. |
| r.WildMods.SSDM.SideViewDepth.ViewTileCount | 8 | Fine view-window count with coarse fallback. |
| r.WildMods.SSDM.SideViewDepth.ViewTileSizeCm | 512 | Fine core width. |
| r.WildMods.SSDM.MirrorSurface.HardwareRayTracing | 0 | Experimental tracing/compare. Ordinary authority metadata forces compute fallback. |

See [Technical Architecture](TechnicalArchitecture.md) for allocation and cost details.
