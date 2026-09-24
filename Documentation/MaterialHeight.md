# Material Surface Output

Use **Material Surface Output** when the height seen by the material is more
complex than a single Profile texture. It lets SSDM evaluate the final height
from the real material instance, including layered materials, triplanar
projection, cell bombing, distance blending, parameters, and static switches.

## The one connection that matters

1. Open the parent material, not a material instance.
2. Add **WildMods SSDM Surface Output** to the graph.
3. Find the final normalized height value after every layer, randomization,
   projection, and distance blend.
4. Connect that value to the node's **Height** input.
5. Leave the material's Base Color, Roughness, Normal, wetness, Material
   Attributes, and Substrate routing exactly as they were.
6. In the SSDM Profile, set **Height Source** to **Material Surface Output**.
7. Select the height direction that matches the material. **Black Is Raised**
   is the SSDM default; use **White Is Raised** for the opposite convention.

The custom output is a side channel. It does not replace a texture sample,
Material Attributes, or the material output node, and it does not change the
visible material by itself.

## Sensei master terrain material

In Sensei's master terrain graph, use the single **Resulting Height Map** value
after Material A-E have been combined and after the final cell-bombing and
distance-blend selection. Connect that final value to **WildMods SSDM Surface
Output → Height**.

Do not place the SSDM surface output inside Material C or repeat it inside
Materials A-E. Doing so gives SSDM one layer's height while Base Color displays
the fully combined terrain. That mismatch is what creates the sharp split at a
material boundary.

Wetness may continue to modify Base Color and Roughness after the height has
been assembled. If a wetness function intentionally changes physical relief,
route its resulting height into the same final height chain before the SSDM
output.

## Optional channels

- **Coverage** defaults to 1. Use 0 to disable SSDM for a pixel or a value
  between 0 and 1 to fade the relief.
- **Surface ID** defaults to 0. It is reserved for consumers that need to
  distinguish material-defined surface types.
- **Payload 0** and **Payload 1** default to zero. They are versioned extension
  channels for separately installed SSDM modules. Core rendering does not
  allocate payload targets unless a consumer requests them.

## Choosing a source mode

- **Profile Texture** preserves the original SSDM workflow and adds no
  material-surface capture pass.
- **Material Surface Output** is strict. If the visible material does not
  contain the custom output, SSDM fails closed for that pixel instead of using
  unrelated height.
- **Prefer Material Surface Output** uses the custom output when available and
  otherwise falls back to the Profile's Height Map. Use it while migrating a
  mixed set of materials; it requires a valid fallback Height Map.

## Current physical scope

Core SSDM changes the rendered surface. It does not change collision,
navigation, or the physical shadow caster. The public, versioned surface-field
contract is present for separately installed collision, snow, sand, and water
modules, but those systems are not part of the core plugin.

## Mesh Terrain procedure in V1.0

Attach WildModsSSDM to the Mesh Partition terrain owner so eligible live generated near-field primitives can be resolved. Publish the final blended normalized material height, not an unrelated fallback texture. For Sensei, use strict Material Surface Output and the final Resulting Height Map described above. Base anchoring, height direction and Full Quality/Max Distance apply to ordinary and side depth.

Enable Side-View Depth independently of mirroring. Live/stable sources enter capture allocation only within camera-to-bounds MaxDistance plus hysteresis. Stale Preview/FarField objects are excluded. Fine view-focused windows have padded margins, staged replacement and coarse fallback under the Mirror Material Pixel Budget. Allow preparation to finish after edits; this is active-view detail, not globally resident high-resolution world geometry.

Normalize Sharp Edges now works on authored height, including preferred authored output. Radius measures evaluation pixels/capture texels, with resolution-dependent world width; start small. Smart Bevel is separately available for Profile Texture YZ/triplanar height, not authored output.

Leave optional Mesh Terrain Shadow Captures and Allow Mesh Terrain Hardware Ray Tracing off unless deliberately qualifying them. They are independent of ordinary HGSDM and automatic side-depth residency. Mesh Partition and Sensei are optional integrations, not required or included dependencies for ordinary meshes.
