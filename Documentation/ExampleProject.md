# Example Project

The separate example project intentionally does not contain or redistribute HGSDM.

1. Install HGSDM for UE5.8 through Fab/Epic Games Launcher.
2. Download and extract the single `HGSDM_ExampleProject` folder.
3. Open `HGSDM_ExampleProject.uproject` with UE5.8.

The descriptor enables `WildMods_SSDM`. The example project opens its own `/Game/Maps/HGSDM_Example` level, which references the meshes, materials, profiles, and components in the separately installed plugin. If Unreal reports a missing plugin, install HGSDM for that exact engine version. Never place a `Plugins/WildMods_SSDM` copy inside the example-project archive.
