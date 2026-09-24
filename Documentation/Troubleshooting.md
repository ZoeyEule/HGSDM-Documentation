# Troubleshooting and Limits

## No relief / flat normals

Confirm plugin/shaders are ready, the component has a valid Profile, the correct primitive is registered, renderer settings are enabled, displacement is positive and the camera is within MaxDistance. Strict Material Surface Output requires the custom output on the visible material; missing output fails closed. Preferred mode needs a valid fallback texture. Check height direction.

Use World Normal buffer visualization to separate missing HGSDM normals from lighting. Compare Side-View Depth off/on to isolate ordinary and captured relief. Keep layered terrain on its final published height rather than substituting an unrelated texture.

## Mapping mismatch

Match visible world projection to the height projection. Use Mesh UV0 for authored static mesh layouts and leave visible material routing intact. Do not add the world projection helper merely for UV support.

Texture UV0 supports ordinary non-Nanite static meshes, opaque/masked material, authored LOD0 UV channel0 and multiple sections. Nanite, instanced/HISM, spline, skeletal/deforming, translucent, WPO/material displacement/PDO, other UV channels and arbitrary material-generated UVs are outside this workflow. Unsupported targets fail closed. Texture UV0 side/mirror controls are unavailable; strict Material Surface Output has separate captured-height eligibility.

## Spikes / excessive softness

Start with a small Normalize Sharp Edges radius and reasonable centimeter displacement. Height must be grayscale rather than color data; disable sRGB for texture height. Large normalization/bevel radii soften intended fine relief. Material-height radius uses screen/capture samples, not constant world width, so coarse captures and main view can smooth differently.

## Side overlays / blurry sides

V1.0 retains accepted ordinary relief on matching faces instead of replacing it with coarse side captures. Keep the whole installation consistent. Check capture resolution/budget and eligibility when side detail is absent; missing sampled frequency cannot be recovered. Static side-material re-evaluation has cache restrictions. Report on/off comparisons, profile values, view location and relevant logs if overlays return.

## Shadow speckles

HGSDM relief shadows are independent of native mesh shadows. Increase Minimum Shadow Displacement modestly; values above 1 cm resolved the reported micro-shadow artifact in the tested rock material. Disable component Enable SSDM Shadows to isolate it. Native shadow/VSM pool overflow is a separate engine/project memory issue.

## Terrain capture errors / GPU crash

Use final published blended height, a component on the Mesh Partition owner, bounded profile distance and stable near-field sources. Preview/FarField objects must not become near-field side captures. Leave native terrain hardware RT and optional terrain shadow captures off unless deliberately qualifying them. Base HGSDM uses GPU compute/raster work without requiring hardware RT.

Retain the project log/GPU dump, GPU/driver details, engine memory warnings and reproducible camera for a device-removed crash. Capture budget is not a total-memory cap; one RT/VSM warning alone does not establish a crash cause.

## Warmup / dynamic materials

Profile/source changes prepare replacements asynchronously, retaining last completed usable data while possible. Wait for shader/capture readiness before comparison. Dynamic material instances, untracked collection edits and time/view/scene-dependent graphs are unsafe mirror re-evaluation caches. Hardware diagnostics fall back to compute when ordinary-authority metadata is active.

## Upgrade / build mismatch

Close Unreal, replace the complete older plugin, and remove duplicate installations visible to the project. Match the supplied UE5.8.2 Win64 build or rebuild source for the exact engine build. Inspect compiler/shader errors. Do not delete project content/assets to resolve a binary mismatch.

## V1.0 qualification

UE5.8.2 Win64, DX12/SM6 deferred blendable-GBuffer integration. Other engines/platforms/RHIs, forward/mobile/VR and native adaptive Substrate parity are not claimed. Physics/collision/navigation and native caster/RT geometry are unchanged. Mirror/material re-evaluation are advanced opt-ins. Folded same-primitive layers, thin silhouettes, adjacent-face derivatives, coplanar overlap and capture transitions retain scene-specific limits.
