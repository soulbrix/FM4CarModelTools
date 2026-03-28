# FM4 Car Model Tools

A standalone Tkinter tool for exporting **Forza Motorsport 4** `.carbin` model sections to Wavefront OBJ, editing them in Blender, and importing them back into the game file.

This app is built around two Python files:

- `fm4_obj_app.py` — desktop GUI
- `fm4_obj.py` — OBJ export/import/conversion logic

It relies on the `fm4carbin` package for parsing and writing FM4 carbins.

## What it does

The app is designed for **car part editing workflows** such as:

- exporting FM4 sections to OBJ for Blender editing
- importing edited meshes back into a `.carbin`
- rebuilding section topology when vertex counts change
- patching positions only when topology stays identical
- importing foreign OBJ meshes into a chosen subsection
- validating OBJ files before import
- remapping OBJ materials to FM4 subsection slots
- batch-importing many OBJ files into matching sections
- adding brand-new sections to a carbin

## Main features

### Export to OBJ

- Export **selected sections** or **all sections** from a loaded `.carbin`
- Choose the target **LOD** (`0` to `5`)
- Writes OBJ files per section
- Preserves subsection/material grouping in export
- Intended for round-trip editing in Blender or other DCC tools

### Import from OBJ

Supports two import modes:

#### 1. Topology import
Recommended in most cases.

- Allows changed vertex count and changed topology
- Rebuilds the shared vertex pool
- Rebuilds subsection index buffers
- Recomputes tangent-space data needed by the game
- Best choice when you edited or replaced geometry in Blender

#### 2. Safe / positions-only import
Use when the edited OBJ has the **same topology** as the original export.

- Fastest import path
- Updates positions only
- Requires the original OBJ for matching
- Safer when only moving existing vertices

### External OBJ Import

Designed for bringing in an OBJ that did **not** originate from FM4.

Available workflows include:

- **Full topology import**
- **Foreign mesh → one subsection**

Options include:

- target subsection name, such as `body`
- fit mode:
  - `BBox fit`
  - `Uniform fit`
  - `No fit`
- axis handling for Blender-style exports
- optional **flip triangle winding** for foreign meshes

This is useful for experimental swaps, custom meshes, badges, text meshes, and other donor-based conversions.

### Validate OBJ

Checks an OBJ against a target FM4 section before import.

Validation includes:

- vertex and triangle counts
- OBJ material detection
- subsection/material matching
- unmatched OBJ materials
- unmatched section slots
- world-space bounds comparison
- overlap checks
- axis-swap hinting
- warnings and hard errors

### Material Mapper

Lets you inspect and override how OBJ materials map to FM4 subsection slots.

- load auto-detected mapping
- manually edit OBJ material assignments per subsection
- apply the mapping and import in one step
- useful when Blender or exporter naming does not match FM4 subsection names cleanly

### Batch Import

- Import multiple OBJ files from a folder
- Match them to FM4 sections automatically
- Write a single output `.carbin`
- Useful when updating many parts in one pass

### Add New Section

Create a new section inside a carbin using an OBJ as the source mesh.

Includes controls for:

- new section name
- source OBJ
- LOD
- transform/bounds settings
- import options similar to the external import workflow

### Log panel

- persistent in-app operation log
- useful for troubleshooting export/import steps

## Supported workflow

### Standard FM4 round-trip

1. Open an FM4 `.carbin`
2. Select a section
3. Export the section to OBJ at the desired LOD
4. Edit the OBJ in Blender
5. Import it back using:
   - **Topology import** if topology changed
   - **Safe / positions-only** if only vertex positions changed
6. Save as a new `.carbin`
7. Test in Forza Studio and in-game

### Foreign OBJ workflow

1. Open a donor FM4 `.carbin`
2. Go to **External OBJ Import**
3. Pick the foreign OBJ
4. Choose the donor section
5. Choose the target subsection, usually `body`
6. Pick a fit mode
7. Enable **Flip triangle winding** if faces disappear from some angles in-game
8. Save as a new `.carbin`
9. Test in-game

## Requirements

### Required

- Python 3.10+
- `fm4carbin` package placed next to the app files

Required layout:

```text
project_folder/
  fm4_obj_app.py
  fm4_obj.py
  fm4carbin/
    __init__.py
    model.py
    ops.py
    patch.py
    ...
```

### Optional but recommended

- `numpy` — speeds up vertex encoding
- `scipy` — speeds up KD-tree nearest-neighbour mapping during topology rebuilds

Install optional dependencies with:

```bash
pip install numpy scipy
```

## Running the app

From the project folder:

```bash
python fm4_obj_app.py
```

## Building an EXE

If you want a standalone Windows build, use PyInstaller.

### Important

The app will **not** build or run unless the `fm4carbin` package is present.

### Basic command

```bash
pyinstaller --onefile --windowed fm4_obj_app.py
```

### Safer command

```bash
pyinstaller --onefile --windowed --clean --hidden-import=fm4carbin --collect-submodules=fm4carbin fm4_obj_app.py
```

## Blender notes

Recommended general workflow:

- export the original FM4 section first
- use the exported OBJ as the reference for scale, axis, and placement
- keep LOD consistent between export and import
- triangulate meshes before export when possible
- recalculate normals if faces disappear or shade incorrectly
- for foreign objects, align them to the donor part before import

## Known limitations

- The app is focused on **FM4** carbins, not general Forza formats
- Topology import rebuilds shared pools and can affect sibling non-LOD0 subsections if the source section structure depends on them
- Foreign OBJ import is best-effort and depends heavily on:
  - alignment
  - material naming
  - UVs
  - normals
  - donor subsection structure
- Some meshes can look correct in Forza Studio but still fail in-game if runtime expectations are not matched closely enough
- A missing or incomplete `fm4carbin` package will prevent the app from starting or building

## Troubleshooting

### `ModuleNotFoundError: No module named 'fm4carbin'`

The `fm4carbin` package is missing or not in the expected folder.

Fix:

- place `fm4carbin` next to `fm4_obj_app.py`
- or add its path to `sys.path`
- or include it explicitly in your PyInstaller build

### Import succeeds but the part is huge or distorted in-game

Possible causes:

- bad bounds or transform data
- incorrect vertex encoding expectations
- subsection routing mismatch
- invalid LOD rebuild side effects
- importing a foreign mesh without proper donor alignment

### Parts disappear from some viewing angles

Most common causes:

- flipped triangle winding
- reversed normals
- thin or non-manifold geometry

Try:

- enabling **Flip triangle winding for foreign mesh**
- recalculating normals in Blender
- triangulating before export

### OBJ materials do not match the FM4 section

Use:

- **Validate OBJ** to see unmatched materials and subsection slots
- **Material Mapper** to override mappings manually

## Repository contents

Typical minimal repository structure:

```text
.
├─ fm4_obj_app.py
├─ fm4_obj.py
├─ fm4carbin/
├─ README.md
└─ requirements.txt   # optional
```

Example `requirements.txt`:

```text
numpy
scipy
```

## Credits

This toolchain combines a GUI round-trip workflow with FM4 OBJ export/import logic and depends on the `fm4carbin` parsing/writing layer.

## Disclaimer

This project is for research, modding, and format experimentation. Always keep backups of original `.carbin` files before writing modified output.
