# Known Limitations

- Rendered relief only: no added triangles and no physical collision, navigation, physics or native shadow-caster deformation.
- Non-Nanite targets only in 1.0.1; qualified on UE5.8 Windows/Win64, DX12, SM6 and deferred rendering.
- Mesh UV0 supports authored UV channel 0 on eligible ordinary static meshes only.
- Material Surface Output must publish the final normalized combined height; partial layer height can create splits or flat areas.
- Side-view depth costs more than ordinary relief and retained captures can settle after large changes.
- Very low trace samples can miss thin relief at difficult grazing views.
- Re-evaluate Mirror Side Material is experimental and limited to cache-safe static materials.
- Native light shadows use the carrier; HGSDM shadows are separate and optional.
- Large normalization radii soften detail and cost preparation work.
- Collision, snow, sand, water and gameplay deformation are not included.
- Networking is not implemented or required. Projects must replicate any runtime configuration changes they add.
