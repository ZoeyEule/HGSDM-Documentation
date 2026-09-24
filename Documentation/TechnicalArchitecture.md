# HGSDM V1.0 Technical Architecture

**WildMods — Sophia Eule and Valentine Littlelight.**

## Renderer contract

The runtime module loads PostConfigInit to register plugin shaders; the editor module handles editor-only integration. A scene view extension integrates private renderer passes through Unreal's Render Dependency Graph. No engine source patch is required. Private renderer headers tie compatibility to the target engine version; rebuilding on another version requires qualification.

The release target is UE5.8.2 Win64 DX12 SM6 deferred rendering, principally Substrate blendable GBuffer. Native adaptive Substrate code exists but has different retained packet/material limitations; parity is not claimed. Hardware ray tracing is optional, off by default.

HGSDM changes rendered material response, virtual normals and resolved virtual surface depth. Some ordinary passes preserve physical SceneDepth and separately record accepted virtual depth; side-depth arbitration consumes that record. Collision, navigation, physics, native shadow-map caster geometry and ray-tracing acceleration geometry remain the carrier mesh. Do not assume every engine consumer sees a physically displaced mesh. HGSDM relief shadows are additional shader work, not tessellated shadow geometry. Side depth can extend rendered relief silhouettes; ordinary screen-space relief has source-visibility limits.

## Source height and ownership

Components register profiles and eligible live primitives with the world subsystem. Private profile/ownership buffers identify visible physical sources without Custom Depth or Stencil. Texture height is direction-normalized and resampled to canonical square G8 slices. Normalization and optional Smart Bevel prepare authoritative samples when profiles change; displacement/normal/height consumers share those samples. No continuity membranes or bevel triangles are created. Retired serialized controls remain inert.

A private material pass reads final authored height from the real material instance's WildMods SSDM Surface Output. Strict mode rejects missing output; preferred mode may use valid prepared texture fallback. It is a side channel, not replacement visible material routing. Optional Surface ID/payload targets are consumer-requested. Authored-height normalization runs two bounded separable tent filters before signed-offset conversion. Owner, profile, coverage, physical depth and geometric continuity constrain neighborhoods; other published channels are preserved. Disabling it bypasses auxiliary targets/filter work. Radius measures evaluation pixels/chart texels, so world footprint changes with resolution.

Base anchoring maps raised height to [0,D] along the surface normal; centered mode maps to [-D/2,D/2]. Direction chooses raised endpoints. Camera, captures, mirrors and shadows share this convention. Strength is full through FullQualityDistance, fades smoothly to zero at MaxDistance and returns to the carrier. Camera-to-bounds admission is separately distance-bounded with hysteresis.

## Ordinary relief

Private source mapping/inverse reconstruction evaluates relief and publishes material/normal response. Lerp Speed scales iterative corrections; temporal reconstruction/AA is a separate source of perceived settling. Filtering and CPDAS refinement exchange sampling cost for useful detail/stability. Accepted ordinary records carry source screen coordinates, physical device depth and virtual device depth. Identity belongs to physical SOURCE coordinates rather than necessarily the displaced destination.

## Side depth and mirrors

Static eligible sources prepare six projected face charts, retained until source/profile changes. Known cube faces use analytic carrier-foot projection while retaining rotated triplanar/general mapping. Nonplanar terrain retains bounded iterative projection and root refinement. Captures store coverage, physical anchors, offset/height and material data. Shared tangent reconstruction and continuity checks reject disconnected interpolation instead of inventing carrier sheets or blending unrelated colors.

Side depth traces the upper virtual surface without mirroring. The optional mirror adds an underside defined by `2 * MirrorPoint - upperHeight`, meeting at the reflection contour. Coverage, valid footprints, ray bounds and distance fade constrain the shell.

After ordinary resolve, side hits compare against physical/live and accepted virtual depth. Nearest depth alone cannot protect a fine valley from a coarse nearer-looking capture. An early authority guard preserves accepted ordinary pixels matching source owner, chart region and compatible physical face. It derives geometric normals from immutable physical-depth neighbors, not normal maps. Other faces, uncovered regions and uncertain silhouette neighborhoods remain traceable. Source-qualified fine cores suppress coarse filling of their geometry holes.

Blendable material resolve can reuse validated original full-resolution visible material; other hits use retained chart material. Optional re-evaluation retains original opaque Default Lit material at virtual positions. Dynamic instances, untracked collection edits and time/view/scene dependence are unsafe caches. UV-only graphs do not gain a unique side unwrap. Native adaptive Substrate retains packet-specific handling.

Winning pixels are grouped/compacted by face for material resolve, avoiding repeated full-rectangle material scans. Projected bounds and distance admission limit work; near-camera grace deliberately expands some rectangles. Ordinary authority metadata currently forces compute tracing so experimental hardware cannot bypass it, including hardware comparison mode.

## Mesh Terrain at world scale

Mesh Partition sources are classified before allocation. Live/stable near-field primitives must fall within Profile.MaxDistance from camera to bounds, plus hysteresis. Stale Preview/FarField representations are excluded. Unreal streaming governs source residency; HGSDM does not stream an entire terrain itself.

Bounded fine windows focus on the active view with padded margins, staged replacements and coarse fallback. Last completed usable data remains while replacements prepare. Source-qualified priority prevents one terrain primitive suppressing another's tile. The shared pixel budget counts retained multi-buffer samples and re-evaluation weights; it is not viewport size or total GPU memory. Coarse captures cannot recover detail never sampled. Sensei height must come from the final visible layered blend.

Native terrain hardware RT is off by default to limit large BLAS residency/page faults. Optional terrain relief-shadow captures are independently off. Neither is required for base HGSDM or automatic side-depth residency.

## Performance and boundaries

Costs vary with displaced screen coverage, ray steps, overlapping charts, rebuilds, material complexity, normalization radius, anisotropic taps and refinement. Retention avoids repeated preparation; tracing still costs GPU work each frame. Larger budget/resolution can improve quality and increase memory/capture cost. They are not speed controls or total-VRAM guarantees.

The known-plane shortcut and ordinary-interior exits reduced an isolated yaw50 cube side-depth pass median from 38.282 to 8.637 ms at 960x540. This is not a release-wide FPS promise; broader performance optimization remains a later pass.

Physical derivatives can be ambiguous near adjacent faces. Coarse same-owner classification cannot distinguish every folded/overlapping layer. Thin silhouettes, overhangs and capture transitions need scene-specific checks. Normalization uses numerical tolerances; close layers and coplanar redraws are edge cases. Mirror caches retain the restrictions above. See [Troubleshooting](Troubleshooting.md).

## Optional field API

The versioned public surface-field API supports separate integrations with explicit requests, persistence and readback. Ordinary relief needs no field actor. Collision/snow/sand/water gameplay systems are not included core features; see [Surface Field Integration](SurfaceFieldIntegration.md).
