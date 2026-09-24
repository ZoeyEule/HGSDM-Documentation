# Project Settings Reference

Open **Project Settings > Plugins > WildMods SSDM**.

## Renderer and captures

| Setting | Default | Range / behavior |
|---|---:|---|
| Enable SSDM Renderer | On | Gates all HGSDM GPU passes. Restart required. |
| Height Bank Resolution | 512 | 256, 512, 1024 or 2048 square G8 texels per Profile Texture slice. Higher values preserve detail and use more GPU memory. Restart required. |
| Profile ID Update Mode | Every Frame | Optional cache updates when camera or registered rigid SSDM transform changes. Dynamic/deforming/instanced/spline/skeletal/WPO targets use every-frame work. Restart required. |
| Mirror Material Pixel Budget | 1,048,576 | 1,536-16,777,216 retained chart pixels shared by side view and mirrors per world. Not viewport resolution or milliseconds; each sample stores several buffers. Applies live when allocations change. |
| Maximum Mirror Capture Resolution | 512 | 16-512, rounded down to a power of two, per chart dimension. Applies live. More resolution cannot repair unsuitable coordinates. |

## Optional Surface Field API

Renderer-only use performs no field work. These settings apply only when an external consumer requests the versioned Surface Field API.

| Setting | Default | Range / behavior |
|---|---:|---|
| Field Memory Budget (MB) | 256 | 32-4096 MB of retained tile outputs, including unsaved persistent snapshots. Transient capture/source/staging allocations are separate. |
| Field Max Captures Per Frame | 2 | 1-16 capture submissions per frame. Restart required. |
| Field Max Readbacks Per Frame | 1 | 1-8 explicit CPU readback submissions per frame; CPU access is never automatic. |
| Field Max Owner Metadata Bytes | 65,536 | 0-1,048,576 bytes; hard per-owner archive metadata ceiling. |
| Field Max Restore Uploads | 8 | 1-64 outstanding restore uploads, including superseded uploads until retired. |
| Field Max Restore Upload MiB | 64 | 1-512 MiB of encoded archive/owner bytes admitted across outstanding uploads. |
| Field Max Local Tile Resolution | 512 | 64-1024; accepted power-of-two tile resolutions. Restart required. |
| Field Idle Release Delay (Seconds) | 5 | 0-60 seconds before idle tiles release. Dirty persistent snapshots remain until saved/cleared or world teardown. |

`r.WildMods.SSDM.*` console variables are diagnostics. Profiles and Project Settings are the supported shipping configuration; do not ship a project that relies on undocumented overrides.
