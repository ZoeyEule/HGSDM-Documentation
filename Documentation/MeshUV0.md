# Mesh UV0 Workflow

Mesh UV0 lets SSDM follow the same authored UV layout used by a normal static
mesh material. The visible material stays unchanged: base color, normal,
roughness, packed maps, seams, islands, and existing material-side UV logic
continue to render as authored.

## Setup

1. Keep the mesh's existing material assigned. Do not add
   `MF_WildMods_SSDM_Surface` merely to obtain UV support.
2. Make sure the Static Mesh has the intended texture layout in UV channel 0.
3. Author or choose a grayscale heightmap that corresponds to that UV0 layout.
4. In the WildMods SSDM Profile, set **Mapping Mode** to **Mesh UV0**.
5. Leave **Mesh UV Tiling** at `(1, 1)` for one repeat over the authored UV
   layout. Adjust X and Y independently when desired.
6. Use **Mesh UV Offset** to shift the heightmap in UV space.
7. Add the WildMods SSDM component to the actor and register the Static Mesh
   Component as its render target in the usual way.

Use Texture Height Direction for Black Is Raised or White Is Raised. Raised means outward along the surface normal; base anchoring keeps the opposite endpoint on the carrier.
The Profile's displacement, quality distance, maximum distance, and minimum
pixel displacement controls continue to work normally. Minimum pixel
displacement fades only sub-pixel parallax movement; it does not punch a hole
in virtual-normal detail when the camera faces the surface directly.

## Supported texture UV0 workflow in V1.0

- An ordinary `Static Mesh Component` using UV channel 0
- Non-Nanite static meshes
- Opaque or masked materials
- Multiple material sections on the same supported static mesh
- Positive, negative, and greater-than-one authored UV coordinates
- Independent two-axis tiling and offset
- Both **Every Frame** and optimized Profile-ID update modes

## Not supported in Mesh UV0

- Nanite meshes
- Instanced, hierarchical-instanced, spline, skeletal, geometry-cache, or
  other deforming mesh types
- Materials using World Position Offset, material displacement, or Pixel Depth
  Offset
- Translucent materials
- UV channels other than channel 0
- Material-generated UV expressions that do not exist in the mesh's authored
  UV0 vertex data

Unsupported targets fail closed: SSDM does not guess a UV or corrupt the
surface. World Planar YZ and Triplanar remain available for those targets.

## Material-function distinction

`MF_WildMods_SSDM_Surface` is a helper for visible textures that should use the
plugin's world-space projection. It intentionally replaces authored UV
sampling with World Planar YZ or Triplanar sampling.

Do not route a uniquely unwrapped material through that function when the goal
is to preserve its authored layout. Keep the material as-is and choose
**Mesh UV0** on the Profile instead.

## Troubleshooting

### The material still wraps correctly, but there is no SSDM relief

- Confirm the Profile uses **Mesh UV0**.
- Confirm the component targets the intended `Static Mesh Component`.
- Confirm Nanite is disabled for that mesh.
- Confirm UV channel 0 exists on LOD 0.
- Confirm every material section is opaque or masked and does not move/offset
  mesh position.
- Check the Output Log for a one-time `[WildMods_SSDM] Mesh UV0 target`
  warning; it names the target and the unsupported condition.

### The log says the Mesh UV shader is unavailable

Close Unreal Editor, perform a clean plugin build, and restart the editor so
the fixed default-surface UV shader can compile. Then inspect the shader
compiler messages if the warning returns.

### Relief crosses a UV seam badly

Confirm the heightmap itself tiles cleanly or matches both sides of the authored
island seam. The renderer rejects unsafe cross-seam normal derivatives and
falls back to the source material normal there, but it cannot make unrelated
heightmap edges continuous.

### Light shadows look flatter than relief

Native light shadow maps see the carrier mesh. HGSDM relief shadows are separate shader work; check Enable SSDM Shadows and Minimum Shadow Displacement. No physical micro-geometry is added to native shadow maps.

## What Mesh UV0 does not change

Mesh UV0 does not rewrite the material, duplicate the mesh, write Custom Depth
or Stencil, consume another developer's stencil channel, or alter collision.
It uses private WildMods renderer textures and the existing dedicated
Profile-ID path.

## Side-depth eligibility

Profile Texture with Mesh UV0 uses ordinary relief without side/mirror capture tracing. Strict Material Surface Output can use final captured authored height under its separate non-Nanite eligibility. These paths are not interchangeable. Collision and native caster geometry remain unchanged.
