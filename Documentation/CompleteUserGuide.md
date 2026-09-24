# Complete User Guide

## What HGSDM is

HGSDM is WildMods' next step in parallax. It reconstructs height-driven relief, virtual surface normals and optional side depth in screen space without adding triangles. HGSDM extends SSDM by keeping raised relief readable at grazing and side views through a carrier-and-capture path. Mirroring can add a reflected underside, but side-view depth does not require mirroring.

## Install

Fab installs code plugins as engine plugins. Install HGSDM for UE5.8 in Epic Games Launcher, enable **WildMods HGSDM**, restart, and allow shaders to compile. For local development, close the editor, place the single `WildMods_SSDM` folder under the project's `Plugins` folder, build, enable and restart. Never keep both an engine-installed and project-local copy active.

## First surface

1. Use a non-Nanite static mesh with an opaque or masked material.
2. Create a **Wild Mods SSDMProfile** data asset.
3. Choose Height Source, Height Direction and Mapping Mode.
4. Add **WildModsSSDM** to the actor; assign Profile and, when needed, Target Primitive.
5. Start with small displacement, Full Quality Distance below Max Distance, then test direct, moving, close, grazing and side views in PIE.

## Profile Texture workflow

Import grayscale height, disable sRGB and use Grayscale compression. **Black Is Raised** projects dark values outward; **White Is Raised** reverses the convention. Direction is relative to the physical normal, not the camera. Base Anchored Displacement produces 0..D outward; disabling it centers the range around the carrier.

World Planar YZ samples world YZ. Triplanar blends three axes; Triplanar Sharpness controls dominance. Mesh UV0 follows authored UV channel 0 and has the restrictions in [MeshUV0](MeshUV0.md).

## Material Surface Output and Mesh Terrain

For layered/randomized terrain, add **WildMods SSDM Surface Output** to the parent material and connect the final normalized combined height. Keep Base Color, Roughness, Normal, Material Attributes and Substrate routing unchanged. Use strict Material Surface Output when every target publishes height; use the preferred mode only with a valid fallback texture. Follow [MaterialHeight](MaterialHeight.md) for Sensei and Mesh Partition.

## Distance and stability

Full relief remains through Full Quality Distance, fades smoothly to the carrier until Max Distance, then terminates. Side depth follows the same range. Minimum Screen Displacement is a pixel threshold for sub-pixel motion; zero can restore visible darting. Lerp Speed scales inverse-solver correction: 1 is the established immediate response, lower values converge more slowly.

## Sharp edges and Smart Bevel

Normalize Sharp Edges smooths source height. Radius is canonical bank texels for Profile Texture and evaluation/chart texels for material output. Start small. Smart Bevel is separate Profile Texture YZ/triplanar preparation. It changes authoritative height without triangles; Bevel Depth limits vertical change and Radius Override sets filter width.

## Side-view depth and mirrors

**Enable Side-View Depth** reconstructs upper relief at grazing and side views and is the normal control for full side readability. **Enable Mirrored Surface** adds an optional reflected underside. Mirror Point is the reflection level inside the active displacement range.

Retained material charts share the project pixel budget. Live sources near the active view receive captures; padded view-focused regions and coarse fallback protect detail while staged updates prepare. A large camera/source/profile change can settle briefly. Re-evaluate Mirror Side Material is experimental and limited to cache-safe static materials; leave it off unless [Reference](Reference.md) confirms eligibility.

## HGSDM relief shadows

**Enable SSDM Shadows** controls separate relief-shadow work; native light shadows still use the carrier. Minimum Shadow Displacement suppresses microscopic separations. Values above 1 cm solved the reported qualified rock artifact, but tune per asset. Mesh Terrain shadow captures and hardware ray tracing are optional and off by default.

## Performance

Start side-view trace samples at 16, use 8 for aggressive performance, and 128 for full quality. Keep distance ranges and the shared capture budget no larger than needed. Use Profile-ID caching only for eligible rigid targets. Benchmark focused PIE or a Development build. See [Performance](Performance.md) and [ProjectSettings](ProjectSettings.md).

## Package and remove

Package with UE5.8, Win64, DX12 and SM6. Verify Development before Shipping. To remove HGSDM, first remove components and material custom-output nodes/references, disable the plugin, restart, then uninstall it. Back up the project first.
