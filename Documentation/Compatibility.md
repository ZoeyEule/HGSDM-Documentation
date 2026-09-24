# Compatibility and Requirements

HGSDM 1.0.1 supports Unreal Engine 5.8 on Windows with a Win64 target. The qualified renderer is DirectX 12, Shader Model 6 and deferred rendering. The plugin contains `WildMods_SSDM` (Runtime) and `WildMods_SSDMEditor` (Editor). It is not network replicated and requires no network service.

## Required

- UE 5.8 installed binary or source build on Windows.
- Win64 development and packaged target.
- DX12, SM6 and deferred rendering.
- Non-Nanite targets and opaque or masked materials.
- Restart after enabling the plugin or changing restart-required Project Settings.

## Supported workflows

- Ordinary `UStaticMeshComponent` targets with Profile Texture height.
- World Planar YZ, Triplanar, and restricted authored Mesh UV0 mapping.
- Final blended height published through WildMods SSDM Surface Output.
- Mesh Partition / Sensei Mesh Terrain owners using live near-field primitives and final published height.
- Side-view depth and optional mirrored undersides on supported carriers.

## Unsupported or unqualified in 1.0.1

Nanite targets; forward shading; mobile; Vulkan; DX11; macOS; Linux; consoles; non-Win64 packages; translucent materials; skeletal, spline, geometry-cache, deforming, instanced and hierarchical-instanced targets unless a workflow expressly says otherwise; Mesh UV0 materials using WPO, material displacement or Pixel Depth Offset.

HGSDM does not alter physical collision, navigation, physics or Unreal's native shadow caster. Mesh Partition, Sensei terrain and the Surface Field API are optional integrations, not included or required dependencies.
