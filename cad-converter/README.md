# 3D CAD Converter

Turn STEP and IGES files into STL, OBJ, 3MF, glTF/GLB, FBX and other mesh formats on Windows, one file or a whole folder at a time, with control over mesh quality and units. The same install gives you the desktop app, the `cadconvert` CLI and an MCP server for AI agents.

[Download the free trial](https://www.contenta-software.com/3dcadconverter/) | [MCP server reference](mcp-server.md)

Documented version: **1.0.24**.

## What it does

- **CAD to mesh**: STEP, IGES and BREP tessellated with OpenCascade; you pick a quality preset or set the linear and angular deflection.
- **Mesh to mesh**: STL, OBJ, PLY, FBX, Collada, 3MF, glTF/GLB, VRML, X3D, OFF.
- **19 formats read, 14 written** (table below).
- **Units**: convert to mm, cm, in, m or ft.
- **Up axis**: rotate the output to Y-up (game engines, glTF, three.js) or Z-up (CAD, 3D printing).
- **Polygon reduction** for mesh sources, from the CLI.
- **Mesh repair** on request.
- **Batch** a folder with parallel workers; **watch** a folder and convert new files.

## Install and PATH

The CLI is installed with the desktop app. A default install is per-user, in `%LOCALAPPDATA%\Programs\CadConverter\`, and the installer adds that folder to your user `PATH`. Open a new terminal after installing, then check:

```powershell
cadconvert --version    # 1.0.24 or later
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

Each of these was run against 1.0.24.

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

# STEP to OBJ in inches, with mesh repair
cadconvert convert -i model.step -o model_in.obj --units in --repair

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

# Inspect a file
cadconvert info model.step

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
| `--up-axis` | `y`, `z` or `unchanged` (default). A STEP, IGES or BREP re-export cannot be rotated; the CLI prints a note and writes it unrotated |
| `--decimate` | Share of triangles to keep, as `0.25` or `25%`. Mesh sources only (see below) |
| `--repair` | Mesh repair |
| `--units` | `mm`, `cm`, `in`, `m`, `ft` |
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

## BREP input

OpenCascade native `.brep` and `.brp` files convert to every mesh format, re-export to STEP, IGES or BREP, and work with `info`, the desktop preview and thumbnails. Limits:

- OpenCascade BREP only. Parasolid (`.x_t`) and ACIS (`.sat`) files do not open.
- A `.brep` file stores no unit, so it is read as millimetres.
- Each file becomes one merged solid, with no part names, colours or assembly tree.

## info

`info` prints the format, the unit the file declares, part count, triangle and vertex counts, materials/textures and, for STEP, IGES and BREP files, the bounding box. 3MF, STEP and IGES files declare their unit and glTF/GLB is always metres; STL, OBJ, PLY and OFF carry no unit and show `Unknown`.

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
| USD | `.usd` | Yes | No | Mesh |
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
