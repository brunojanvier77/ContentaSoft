# 3D CAD Converter

Turn STEP and IGES files into STL, OBJ, 3MF, glTF/GLB, FBX and other mesh formats on Windows, one file or a whole folder at a time, with control over mesh quality and units. The same install gives you the desktop app, the `cadconvert` CLI and an MCP server for AI agents.

[Download the free trial](https://www.contenta-software.com/3dcadconverter/) | [MCP server reference](mcp-server.md)

Documented version: **1.0.25**.

## What it does

- **CAD to mesh**: STEP, IGES and BREP tessellated with OpenCascade; you pick a quality preset or set the linear and angular deflection.
- **Mesh to mesh**: STL, OBJ, PLY, FBX, Collada, 3MF, glTF/GLB, VRML, X3D, OFF; USD and USDZ (text or binary) are read.
- **19 formats read, 14 written** (table below).
- **Units**: files that declare a unit come in at their true size; the output is in mm, or in cm, in, m or ft.
- **Up axis**: rotate the output to Y-up (game engines, glTF, three.js) or Z-up (CAD, 3D printing), starting from the up axis the source declares.
- **Polygon reduction** for mesh sources, from the CLI.
- **Surface repair** for STEP, IGES and BREP before meshing, on request.
- **Batch** a folder with parallel workers; **watch** a folder and convert new files.

## Changes in 1.0.25

Source units are now read, so some outputs have a different size than in 1.0.24:

- glTF/GLB to STL (or OBJ, 3MF, PLY) is 1000x larger: glTF is in metres, and the output is now its true size in mm.
- Any file converted to glTF/GLB opens at its true size in metres in a glTF viewer.
- A Blender FBX in centimetres (Blender's default) comes in 10x larger, at its true size.

Also in 1.0.25:

- Binary USD (`.usdc`) and USDZ convert; in 1.0.24 every binary USD failed. USD prim transforms and `metersPerUnit` are applied, and `info` on a USD file reports its real format, parts, unit and bounding box.
- `--up-axis` reads the up axis the source declares (USD, Collada, FBX, glTF).
- glTF, FBX and Collada outputs declare their unit and up axis.
- `--repair` is described for what it does: STEP/IGES/BREP surface repair. It has no effect on mesh sources.
- A batch of files from different drives writes every output inside the output folder.
- Desktop app: the "From" unit is ignored for files that declare their own unit.

## Install and PATH

The CLI is installed with the desktop app. A default install is per-user, in `%LOCALAPPDATA%\Programs\CadConverter\`, and the installer adds that folder to your user `PATH`. Open a new terminal after installing, then check:

```powershell
cadconvert --version    # 1.0.25 or later
cadconvert formats
```

If `cadconvert` is not found, or `--version` prints an older number, call the exe by its full path or remove the older copy that comes first on `PATH`.

## Commands

| Command | What it does |
|---------|--------------|
| `convert <file> [<output>]` | Convert one file |
| `batch <dir> [<dir>]` | Convert every 3D file in a folder |
| `info <file>` | Format, declared unit, parts, triangles, vertices, bounding box |
| `formats` | List supported formats |
| `watch <dir> [<dir>]` | Convert new files that appear in a folder |
| `register -k <key> -e <email>` | Register a license key |
| `serve` | Start the MCP server (stdio) |

Input and output paths can be given as arguments or with `-i`/`-o`; both forms work on `convert`, `batch` and `watch`. When the output is left out:

- `convert` writes next to the input, with the extension of the `-f` format (`-f` is then required);
- `batch` and `watch` write to a new folder next to the input folder, named `<input>_converted`.

The CLI has no `--json` flag; for structured output use the [MCP server](mcp-server.md).

## Examples

Each of these was run against 1.0.25.

```powershell
# STEP to STL at the default (standard) quality
cadconvert convert -i model.step -o model.stl

# The same with positional paths
cadconvert convert model.step model.stl

# No output path: writes model.glb next to the input
cadconvert convert model.step -f glb

# Finer mesh for smooth curved surfaces: the fine preset, or the numbers behind it
cadconvert convert model.step -o model_fine.stl --quality fine
cadconvert convert model.step -o model_fine.stl --tessellation 0.01 --angular 0.1

# Y-up GLB for game engines and three.js
cadconvert convert model.step -o model_y.glb --up-axis y

# STEP to OBJ in inches, repairing the STEP surfaces before meshing
cadconvert convert -i model.step -o model_in.obj --units in --repair

# glTF (metres) to STL at its true size in mm, and in inches
cadconvert convert tower.glb tower.stl
cadconvert convert tower.glb tower_in.stl --units in

# A Blender FBX in centimetres to STL in mm
cadconvert convert tower_cm.fbx tower_fbx.stl

# USDZ (binary USD) to GLB; a Y-up USD stage to Z-up STL for 3D printing
cadconvert convert scene.usdz scene.glb
cadconvert convert tower_yup.usda tower_z.stl --up-axis z

# Keep about a quarter of a mesh's triangles (mesh sources only)
cadconvert convert scan.stl -o scan_light.glb --decimate 0.25

# ASCII STL instead of binary
cadconvert convert -i model.stl -o model_ascii.stl --binary false

# STEP to IGES, and BREP to STEP
cadconvert convert -i model.stp -o model.igs
cadconvert convert part.brep part.step

# A folder to OBJ at fine quality, and a folder to STL (the default format) in .\cad-files_converted
cadconvert batch -i .\cad-files -o .\meshes -f obj --quality fine
cadconvert batch .\cad-files

# Inspect a file: format, declared unit, size
cadconvert info model.step
cadconvert info scene.usdz

# Convert new files dropped into a folder to Y-up GLB in .\incoming_converted (Ctrl+C to stop)
cadconvert watch .\incoming -f glb --up-axis y
```

The output format comes from `-f`, or from the output file extension when `-f` is left out.

## Conversion options

`convert`, `batch` and `watch` take the same conversion options.

| Option | Description |
|--------|-------------|
| `-i, --input` | Input file (`convert`) or folder (`batch`, `watch`); or give it as the first argument |
| `-o, --output` | Output file or folder; or give it as the second argument. Optional (see above) |
| `-f, --format` | Target format. `convert`: from the output extension; `batch` and `watch`: `stl` |
| `--tessellation`, `--quality` | Mesh quality for STEP, IGES and BREP sources: a preset (`draft`, `standard`, `fine`, `ultrafine`; `ultra` also works) or a linear deflection in mm. Default `standard` |
| `--angular` | Angular deflection in **radians** (0.5 is about 29 degrees). Default: the preset's value; `0.5` with a numeric `--tessellation` |
| `--up-axis` | `y`, `z` or `unchanged` (default). The rotation starts from the up axis the source declares (USD, Collada, FBX and glTF declare one). A STEP, IGES or BREP re-export cannot be rotated; the CLI prints a note and writes it unrotated |
| `--decimate` | Share of triangles to keep, as `0.25` or `25%`. Mesh sources only (see below) |
| `--repair` | Repairs STEP, IGES and BREP surfaces before meshing: fixes broken edges and faces and closes small gaps. No effect on mesh sources (STL, OBJ, PLY ...) |
| `--units` | Target unit: `mm` (default), `cm`, `in`, `m`, `ft`. See [Units](#units) |
| `--binary` | Binary STL/PLY (default `true`); `--binary false` writes ASCII |

The presets set these deflections:

| Preset | Linear (`--tessellation`) | Angular (`--angular`) |
|--------|---------------------------|-----------------------|
| `draft` | `1.0` | `5.0` |
| `standard` (default) | `0.1` | `0.5` |
| `fine` | `0.01` | `0.1` |
| `ultrafine` | `0.001` | `0.05` |

`--tessellation`, `--quality` and `--angular` apply when the source is STEP, IGES or BREP; mesh-to-mesh conversions ignore them. On our test part, `fine` gave about 12x the triangles of `standard`.

`batch` also takes `-r/--recursive` (default on) and `-w/--workers` (default: the number of logical processors).

### Polygon reduction (`--decimate`)

- Works on mesh sources: STL, OBJ, PLY, glTF/GLB, FBX, 3MF, Collada, OFF, VRML, X3D, AMF and DXF.
- STEP, IGES and BREP are not reduced: they are meshed at the density you choose with `--quality` or `--tessellation`, so use `--quality draft` for the lightest mesh. DWG and USD sources are not reduced either. In these cases the file is written at full detail and the CLI prints a note.
- It uses quadric edge collapse: the cheapest edges collapse first, so flat areas lose triangles before curved ones, and open borders stay open.
- A mesh over 2,000,000 triangles is written unreduced, and so is one where the reduction would leave less than half the triangles you asked for; the CLI prints a note.
- The desktop app has no reduction control; this is a CLI option.

## Units

A file that declares its unit is read at its true size, converted to millimetres, then written in `--units` (millimetres when left out):

| Source | Unit read from |
|--------|----------------|
| STEP, IGES | the file's own unit |
| 3MF | the model's `unit` attribute |
| glTF, GLB | always metres (glTF specification) |
| FBX | `UnitScaleFactor` |
| Collada | `<unit>` (metres when absent) |
| USD, USDZ | `metersPerUnit` (centimetres when absent, the USD default) |
| BREP | stores no unit; read as millimetres |
| STL, OBJ, PLY, OFF, VRML, X3D, AMF, DXF, DWG | carry no unit; the numbers are taken as millimetres |

The CLI has no option to set the unit of a source. glTF, FBX and Collada outputs declare the unit and up axis they are written in, so a glTF output opens at its true size in metres.

In the desktop app, the wizard's "From" unit applies only to files that carry no unit (STL, OBJ, PLY ...); a file that declares its unit ignores it.

## BREP input

OpenCascade native `.brep` and `.brp` files convert to every mesh format, re-export to STEP, IGES or BREP, and work with `info`, the desktop preview and thumbnails. Limits:

- OpenCascade BREP only. Parasolid (`.x_t`) and ACIS (`.sat`) files do not open.
- A `.brep` file stores no unit, so it is read as millimetres.
- Each file becomes one merged solid, with no part names, colours or assembly tree.

## info

`info` prints the format, the unit the file declares, part count, triangle and vertex counts, materials/textures and, for STEP, IGES, BREP and USD files, the bounding box in mm. The unit is read as described in [Units](#units); STL, OBJ, PLY and OFF carry no unit and show `Unknown`. A USD or USDZ file shows its own format (`Usd`, `Usdz`), its parts and its unit.

## Supported formats

| Format | Extensions | Read | Write | Type |
|--------|-----------|------|-------|------|
| STEP | `.step`, `.stp` | Yes | Yes | CAD |
| IGES | `.iges`, `.igs` | Yes | Yes | CAD |
| BREP | `.brep`, `.brp` | Yes | Yes | CAD |
| STL | `.stl` | Yes | Yes | Mesh |
| OBJ | `.obj` | Yes | Yes | Mesh |
| PLY | `.ply` | Yes | Yes | Mesh |
| FBX | `.fbx` | Yes | Yes | Mesh |
| Collada | `.dae` | Yes | Yes | Mesh |
| 3MF | `.3mf` | Yes | Yes | Mesh |
| glTF | `.gltf` | Yes | Yes | Mesh |
| GLB | `.glb` | Yes | Yes | Mesh |
| VRML | `.wrl` | Yes | Yes | Mesh |
| X3D | `.x3d` | Yes | Yes | Mesh |
| OFF | `.off` | Yes | Yes | Mesh |
| AMF | `.amf` | Yes | No | Mesh |
| DWG | `.dwg` | Yes | No | Mesh |
| DXF | `.dxf` | Yes | No | Mesh |
| USD | `.usd`, `.usda`, `.usdc` | Yes | No | Mesh |
| USDZ | `.usdz` | Yes | No | Mesh |

`cadconvert formats` prints the list for your installed version.

## Exit codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error |
| 2 | Invalid arguments |
| 3 | File not found |
| 4 | Conversion failed |
| 5 | License required (trial limit reached or trial ended) |

## Trial and license

- The trial lasts 30 days and includes **10 conversions**. After the 10th conversion, or after 30 days, conversion stops until you register.
- After 30 days the MCP server refuses to start.
- License: **$179 one-time** for a lifetime license; the [buy page](https://www.contenta-software.com/3dcadconverter/buy.php) also lists a quarterly plan.
- Register from the CLI with `cadconvert register -k <key> -e <email>`, or in the desktop app.

## MCP server

`cadconvert serve` starts an MCP server with 4 tools: `convert_cad`, `get_file_info`, `detect_format`, `list_formats`. See the [MCP reference](mcp-server.md).
