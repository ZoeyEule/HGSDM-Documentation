# Release Verification



## Target and authors



HGSDM 1.0.1 by **WildMods — Sophia Eule and Valentine Littlelight**. Plugin identity WildMods_SSDM. Build target UE5.8.2 Win64, DX12/SM6 deferred rendering; principal qualification uses Substrate blendable GBuffer.



## Qualification basis



Sophia Eule accepted the functioning plugin as V1.0 after showroom and DtGA Mesh Terrain testing, covering ordinary relief, movement/grazing views, multiple lights, side depth and mirrored rocks. Recent repairs rebuilt Editor Development and UnrealGame Development/Shipping and compiled trace/occlusion, capture, normalization and four legacy material resolve shader variants.



Profile normalization validation passed in DtGA. Production-expression checks passed 38 depth arbitration, 1,161 plane projection, 46 ordinary authority and 234 existing continuity/reprojection checks. These check equations/wiring rather than every GPU scene.



Explicit terrain-side-on comparison completed 66/66 charts for 11 sources before capture. Ordinary detail remained visible, normalization had a rendered effect and detached color bands were absent at the reproduced view. Settings/camera were restored, no assets saved and no GPU crash occurred. Startup shader availability messages were observed; the latest repair did not independently requalify every UV case.



The 1.0.1 optimization pass adds exact planar tracing, tighter face intervals, face-specific solver selection and a per-Profile side-view trace budget. In the supplied showroom, the side-depth GPU pass fell from roughly 145 ms at the original full settings to roughly 22–24 ms using the aggressive 8-sample preset at 40% screen percentage. This combines renderer improvements with an explicit quality tradeoff and is not a universal whole-frame speedup. The focused Editor run remained subject to separate frame pacing, so the release owner should confirm final focused-PIE frame rate on the target machine.



## Package delivery



A clean UE5.8 source package includes source, shaders, sample content and current guides. Generated Binaries, Intermediate, Build and Saved folders are excluded as required by the Fab technical review; Unreal regenerates them when the plugin is compiled. Release preparation checks required files, version, documentation links and credits, archive paths and SHA256 hashes. Build records are retained separately from customer materials.



A fresh downloaded-package runtime check and store submission remain release-owner actions. Successful packaging is not an exhaustive compatibility test. See Troubleshooting.md for supported scope and scene-specific limits.

