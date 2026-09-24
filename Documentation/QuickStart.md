# Quick Start

## Install

Use the Win64 distribution with UE5.8.2, DX12, SM6 and deferred rendering. Close the editor, copy `WildMods_SSDM` into the project's `Plugins` folder, enable **WildMods HGSDM**, restart and allow shaders to compile. Runtime and editor modules are included.

## Texture-driven static mesh

1. Create or duplicate a **Wild Mods SSDMProfile** data asset. Import a corresponding grayscale heightmap, disable **sRGB**, and use **Grayscale** compression.
2. Select **Profile Texture** as Height Source and assign Height Map.
3. Choose **Texture Height Direction**. Default **Black Is Raised** means black grows outward along the physical surface normal; **White Is Raised** reverses it. This convention is not camera-dependent.
4. Keep **Base Anchored Displacement** enabled for an outward 0..D range. A 20 cm span gives 0..20 cm; disabling anchoring gives -10..+10 cm.
5. Add **WildModsSSDM** to the actor and assign Profile. Assign Target Primitive explicitly when several candidates exist; ordinary owners can otherwise resolve eligible primitives.
6. Choose Triplanar or World Planar YZ and match the visible material projection. `MF_WildMods_SSDM_Surface` is optional for visible world-projected textures. For authored layouts use [Mesh UV0](MeshUV0.md).
7. Start with modest displacement, Full Quality Distance below Max Distance, and test direct, moving, close and grazing views.

## Side depth and mirrors

Enable Side-View Depth is on by default for supported carriers, independently of mirroring. Enable Mirrored Surface adds the reflected shell; enable only when an underside is wanted. Set Mirror Point within the active displacement range. Both paths share capture limits and distance fade.

Leave Re-evaluate Mirror Side Material off while verifying basic setup. It has static-material/cache restrictions in the [Reference](Reference.md).

## Blended terrain

Publish the final blended normalized height through **WildMods SSDM Surface Output**, then use strict Material Surface Output. An unrelated Profile Texture cannot represent a layered terrain correctly. Follow [Material Height](MaterialHeight.md) for Sensei and Mesh Partition setup.

## Tune and explore

Normalize Sharp Edges defaults to radius 2. Smart Bevel is a separate texture-height preparation feature. Material-output radius measures evaluation pixels/chart texels, so world width varies with resolution.

Project settings: **Project Settings → Plugins → WildMods SSDM**. Renderer/height-bank changes require restart; mirror budget/resolution apply live. Profile edits reach the renderer live; capture replacements prepare asynchronously.

Showroom: `/WildMods_SSDM/Demo/SSDM_ShowRoom`. See [Troubleshooting](Troubleshooting.md) if relief is missing.
