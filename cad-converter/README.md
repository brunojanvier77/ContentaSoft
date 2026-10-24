# 3D CAD Converter

Turn STEP and IGES files into STL, OBJ, 3MF, glTF/GLB, FBX and other mesh formats on Windows, one file or a whole folder at a time, with control over mesh quality and units. The same install gives you the desktop app, the `cadconvert` CLI and an MCP server for AI agents.

[Download the free trial](https://www.contenta-software.com/3dcadconverter/) | [CLI reference](cli.md) | [MCP server reference](mcp-server.md)

Documented version: **1.0.32**.

## What it does

- **CAD to mesh**: STEP, IGES and BREP tessellated with OpenCascade; you pick a quality preset or set the linear and angular deflection.
- **Mesh to mesh**: STL, OBJ, PLY, FBX, Collada, 3MF, glTF/GLB, VRML, X3D, OFF; USD and USDZ (text or binary) are read. VRML files (`.wrl`, `.vrml`, gzipped `.wrz`) are read as well as written.
- **19 formats read, 14 written** (table below).
- **Units**: files that declare a unit come in at their true size; the output is in mm, or in cm, in, m or ft.
- **Up axis**: rotate the output to Y-up (game engines, glTF, three.js) or Z-up (CAD, 3D printing), starting from the up axis the source declares.
- **Polygon reduction** for mesh sources, from the CLI.
- **Surface repair** for STEP, IGES and BREP before meshing, on request.
- **Batch** a folder with parallel workers; **watch** a folder and convert new files.
- **glTF/GLB under a size limit**: `--max-size 5` writes the file again with fewer triangles until it is under 5 MB (down to 5% of its triangles).

## Changes in 1.0.27

- **VRML import.** `.wrl`, `.vrml` and gzipped `.wrz` files convert to every mesh format, work with `info`, the desktop preview and thumbnails, and can be reduced with `--decimate`. Until 1.0.27 every VRML conversion failed, even of a file the app had written itself. See [VRML input](#vrml-input).
- **VRML output states its unit.** A VRML file written by `cadconvert` now has a root `scale 0.001`: its numbers are millimetres and the scale turns them into metres, as the VRML specification requires. A viewer that follows the specification shows the model at its true size, and STEP to VRML to STL keeps its size.
- **The trial no longer blocks.** After the clean conversions, conversion continues as a trial export; see [Trial and license](#trial-and-license). `batch` no longer exits with a license error.
- **Unreadable files are errors.** `info` (and the MCP tool `get_file_info`) on a file that is missing, empty, corrupt or not a 3D format now fails with an exit code and a message, instead of printing a table of zeros.
- **A contradicting `-f` is refused.** `cadconvert convert model.step out.stl -f obj` exits 2 instead of writing OBJ data into a `.stl` file; leave `-f` out to take the format from the extension.
- **`register` reports failure.** It exits non-zero when the key is rejected (it exited 0 before), and 2 when the key is not shaped like a key.
- **`convert` accepts a folder as the output** (`-o .\out`) and names the file after the input.
- **`watch` mirrors subfolders.** Files in subfolders of the watched folder land in the same subfolders of the output, so two files with the same name no longer overwrite each other.
- **`convert_cad` (MCP) needs absolute paths** and names the file it wrote.

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
cadconvert --version    # 1.0.32 or later (prints the build id after a +)
cadconvert status       # license or trial state, free conversions and trial days left
cadconvert formats
```

If `cadconvert` is not found, or `--version` prints an older number, call the exe by its full path or remove the older copy that comes first on `PATH`.

## Commands

| Command | What it does |
|---------|--------------|
| `convert <input> -o <output>` | Convert one file |
| `batch <dir> -o <dir>` | Convert every 3D file in a folder |
| `watch <dir> -o <dir>` | Convert new files that appear in a folder |
| `info <file>` | Format, declared unit, parts, triangles, vertices, bounding box |
| `formats` | List supported formats (`--input` / `--output` to filter) |
| `status` | License or trial state: free conversions left, trial days left |
| `register <email> <key>` | Register a license key (exit 0 on success, 2 for a badly shaped key, 6 when the key is rejected) |
| `serve` | Start the MCP server (stdio) |

The input is the first argument; the output is given with `-o`. When `-o` is left out:

- `convert` writes next to the input, with the extension of the `-f` format (`-f` is then required);
- `batch` and `watch` write to a new folder next to the input folder, named `<input>_converted`.

Every command (except `catalog` and `serve`) takes `--json`, `--plain`, `--quiet`, `-v`/`--verbose` and `--progress`; see [JSON output](#json-output). The [CLI reference](cli.md) lists every option of every command.

## Examples

The examples use the 1.0.32 spellings.

```powershell
# STEP to STL at the default (standard) quality
cadconvert convert model.step -o model.stl

# No output path: writes model.glb next to the input
cadconvert convert model.step -f glb

# Finer mesh for smooth curved surfaces: the fine preset, or the numbers behind it
cadconvert convert model.step -o model_fine.stl --quality fine
cadconvert convert model.step -o model_fine.stl --quality 0.01 --angular 0.1

# Y-up GLB for game engines and three.js
cadconvert convert model.step -o model_y.glb --up-axis y

# STEP to OBJ in inches, repairing the STEP surfaces before meshing
cadconvert convert model.step -o model_in.obj --units in --repair

# glTF (metres) to STL at its true size in mm, and in inches
cadconvert convert tower.glb -o tower.stl
cadconvert convert tower.glb -o tower_in.stl --units in

# A Blender FBX in centimetres to STL in mm
cadconvert convert tower_cm.fbx -o tower_fbx.stl

# USDZ (binary USD) to GLB; a Y-up USD stage to Z-up STL for 3D printing
cadconvert convert scene.usdz -o scene.glb
cadconvert convert tower_yup.usda -o tower_z.stl --up-axis z

# Keep about a quarter of a mesh's triangles (mesh sources only)
cadconvert convert scan.stl -o scan_light.glb --decimate 0.25

# ASCII STL instead of binary
cadconvert convert model.stl -o model_ascii.stl --binary false

# STEP to IGES, and BREP to STEP
cadconvert convert model.stp -o model.igs
cadconvert convert part.brep -o part.step

# A folder to OBJ at fine quality, and a folder to STL (the default format) in .\cad-files_converted
cadconvert batch .\cad-files -o .\meshes -f obj --quality fine
cadconvert batch .\cad-files

# Run the batch again later: convert only what is missing, and write a CSV report
cadconvert batch .\cad-files -o .\meshes -f obj --quality fine --skip-existing --report .\meshes\report.csv

# GLB files of at most 5 MB for a web viewer
cadconvert batch .\cad-files -o .\web -f glb --up-axis y --max-size 5

# VRML in and out: the file is read in metres (a 1 m cube is 1000 mm)
cadconvert convert room.wrl -o room.stl
cadconvert convert model.step -o model.wrl

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
| `-o, --output` | Output file or folder. Optional (see above); `convert` also accepts an existing folder and names the file after the input |
| `-f, --format` | Target format. `convert`: from the output extension, and a `-f` that contradicts the output extension is refused (exit 2); `batch` and `watch`: `stl` |
| `--quality` | Mesh quality for STEP, IGES and BREP sources: a preset (`draft`, `standard`, `fine`, `ultrafine`) or a linear deflection in mm, e.g. `--quality 0.01`. Default `standard` |
| `--angular` | Angular deflection in **radians** (0.5 is about 29 degrees). Default: the preset's value; `0.5` with a numeric `--quality` |
| `--up-axis` | `y`, `z` or `unchanged` (default). The rotation starts from the up axis the source declares (USD, Collada, FBX, glTF and VRML declare one). A STEP, IGES or BREP re-export cannot be rotated; the CLI prints a note and writes it unrotated |
| `--decimate` | Share of triangles to keep, as `0.25` or `25%`. Mesh sources only (see below) |
| `--repair` | Repairs STEP, IGES and BREP surfaces before meshing: fixes broken edges and faces and closes small gaps. No effect on mesh sources (STL, OBJ, PLY ...) |
| `--units` | Target unit: `mm` (default), `cm`, `in`, `m`, `ft`. See [Units](#units) |
| `--binary` | Binary STL/PLY (default `true`); `--binary false` writes ASCII |
| `--max-size` | glTF/GLB only: the largest file to write, in MB (`5`, `0.5` or `15MB`; 1 MB = 1,000,000 bytes). A file over it is written again with fewer triangles until it fits, down to 5% of its triangles; one that still does not fit is written at the smallest size reached and reported as over budget (exit 4) |
| `--profile` | Start from a saved conversion profile (JSON, as the desktop app saves it in `%APPDATA%\CadConverter\profiles`); options on the command line override it |
| `--overwrite` | Replace an output file that already exists |
| `--skip-existing` | Leave an input alone when its output already exists, so a re-run only converts what is missing; a skipped file costs no free conversion |

The presets set these deflections:

| Preset | Linear (`--quality N`) | Angular (`--angular`) |
|--------|---------------------------|-----------------------|
| `draft` | `1.0` | `5.0` |
| `standard` (default) | `0.1` | `0.5` |
| `fine` | `0.01` | `0.1` |
| `ultrafine` | `0.001` | `0.05` |

`--quality` and `--angular` apply when the source is STEP, IGES or BREP; mesh-to-mesh conversions ignore them. On our test part, `fine` gave about 12x the triangles of `standard`.

`batch` and `watch` also take `--include` / `--exclude` (patterns such as `*.step`, several separated by `;`), `-r/--recursive` (default on; `-r false` converts the folder itself only), `-w/--workers` (default: the number of processors) and `--report <file.csv>` (one row per input: files, formats, triangles before and after, output size, size budget, duration, result or error).

### Polygon reduction (`--decimate`)

- Works on mesh sources: STL, OBJ, PLY, glTF/GLB, FBX, 3MF, Collada, OFF, VRML, X3D, AMF and DXF.
- STEP, IGES and BREP are not reduced: they are meshed at the density you choose with `--quality`, so use `--quality draft` for the lightest mesh. DWG and USD sources are not reduced either. In these cases the file is written at full detail and the CLI prints a note.
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
| VRML (`.wrl`, `.vrml`, `.wrz`) | always metres (VRML specification) |
| FBX | `UnitScaleFactor` |
| Collada | `<unit>` (metres when absent) |
| USD, USDZ | `metersPerUnit` (centimetres when absent, the USD default) |
| BREP | stores no unit; read as millimetres |
| STL, OBJ, PLY, OFF, X3D, AMF, DXF, DWG | carry no unit; the numbers are taken as millimetres |

The CLI has no option to set the unit of a source. glTF, FBX and Collada outputs declare the unit and up axis they are written in, so a glTF output opens at its true size in metres. VRML outputs declare their unit with a root `scale 0.001`.

In the desktop app, the wizard's "From" unit applies only to files that carry no unit (STL, OBJ, PLY ...); a file that declares its unit ignores it.

## BREP input

OpenCascade native `.brep` and `.brp` files convert to every mesh format, re-export to STEP, IGES or BREP, and work with `info`, the desktop preview and thumbnails. Limits:

- OpenCascade BREP only. Parasolid (`.x_t`) and ACIS (`.sat`) files do not open.
- A `.brep` file stores no unit, so it is read as millimetres.
- Each file becomes one merged solid, with no part names, colours or assembly tree.

## VRML input

`.wrl`, `.vrml` and gzipped `.wrz` files are read by a reader built into the app, and then go through the same route as any other mesh: every writer, `--decimate`, `--units`, `--up-axis`, `info`, the desktop preview and thumbnails work.

- **Units and axis follow the specification:** VRML is in metres and Y-up. A 1 m cube comes in as 1000 mm. `info` reports the unit as `Meter` and the bounding box in mm.
- **Versions:** VRML 2.0 (Transform, Group, Anchor, Billboard, Collision, Switch, LOD, Inline, Shape, IndexedFaceSet with polygons of any size, Box, Sphere, Cylinder, Cone, ElevationGrid, material colour and transparency, DEF/USE) and VRML 1.0 (Separator, Transform nodes, Coordinate3, IndexedFaceSet, Cube, Sphere, Cylinder, Cone, Material, WWWInline).
- **Not drawn:** Extrusion, IndexedLineSet, PointSet, Text, PROTO instances, textures, scripts and animation. The conversion message names what was left out.
- **Inline** follows only files that sit next to the model, by a relative path. Web, `file:`, drive-letter, network and absolute addresses are refused; an Inline that cannot be read is a warning, not a failure.
- **Limits:** a very large or deeply nested file (about 1.5 GB of text, 24 million triangles) is refused with a message instead of hanging.
- **Output:** `.wrl` and `.vrml` are written; `.wrz` is read-only. The writer does not state an up axis, and a VRML file made from a STEP file holds the STEP file's Z-up data; VRML is read as Y-up, so use `--up-axis` deliberately when you read such a file back.

## info

`info` prints the format, the unit the file declares, part count, triangle and vertex counts, materials/textures and, for STEP, IGES, BREP, USD and VRML files, the bounding box in mm. A file that is missing (exit 3), has no 3D extension (exit 2) or contains no readable geometry is an error with a message. The unit is read as described in [Units](#units); STL, OBJ, PLY and OFF carry no unit and show `Unknown`. A USD or USDZ file shows its own format (`Usd`, `Usdz`), its parts and its unit.

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
| VRML | `.wrl`, `.vrml`, `.wrz` | Yes | Yes (`.wrl`, `.vrml`) | Mesh |
| X3D | `.x3d` | Yes | Yes | Mesh |
| OFF | `.off` | Yes | Yes | Mesh |
| AMF | `.amf` | Yes | No | Mesh |
| DWG | `.dwg` | Yes | No | Mesh |
| DXF | `.dxf` | Yes | No | Mesh |
| USD | `.usd`, `.usda`, `.usdc` | Yes | No | Mesh |
| USDZ | `.usdz` | Yes | No | Mesh |

`cadconvert formats` prints the list for your installed version.

## JSON output

With `--json` every command prints one JSON object on stdout and nothing else. Every object has `success`, `exitCode`, `error` (one sentence, or `null`), `warnings[]`, `errors[]` and `outputs[]`, beside the command's own fields. During the trial it also carries `trialExport`, `trialNotice`, `freeFilesRemaining` and `buyUrl` when they apply. With `--json --progress`, `batch` and `watch` first print one line per finished file, `{"type":"progress","file":...,"completed":N,"total":T}`, and the result object last.

## Exit codes

The four ContentaSoft CLIs share one table; `--help` ends with it and the [CLI reference](cli.md#exit-codes) lists it.

| Code | Name | Meaning |
|------|------|---------|
| 0 | `ok` | Everything asked for was done |
| 1 | `unexpected_error` | An error with no specific code (a bug or a broken install); see the log |
| 2 | `invalid_arguments` | Unknown option, missing or malformed value; nothing was done |
| 3 | `input_not_found` | Input not found or unreadable; nothing was done |
| 4 | `partial_failure` | Some items succeeded, some failed |
| 5 | `all_failed` | No item succeeded (a single-file command whose file failed) |
| 6 | `license_refused` | The license or the trial refused the run (also a key the server rejected in `register`) |
| 130 | `cancelled` | Cancelled (Ctrl+C) |

Exit 2 also covers a badly shaped license key, a `-f` that contradicts the output extension, an unknown output extension and a file that is not a 3D format. A failed `convert` exits 5; a `batch` or `watch` run with some failed files exits 4, and so does a run where a glTF/GLB stayed over its `--max-size` budget (the file is still written).

## Trial and license

- **10 free conversions at full quality**, with no end date (plus 10 more after you confirm the newsletter in the app). The count is per computer, never per batch; nothing expires.
- **After that nothing is blocked.** The CLI, `batch`, `watch` and the MCP server keep converting, as trial exports: STEP, IGES and BREP to a mesh format is meshed at the `draft` quality whatever you chose, and every trial export carries a note in the file where the format has a place for one (all of them except FBX). Conversions that do not mesh a CAD file (mesh to mesh, STEP to IGES) keep their full geometry and carry the note only. There is no watermark.
- `convert` and `batch` print `Buy:` and a link when an output was a trial export; `--json` and the MCP tool return `trialExport: true` and a `buyUrl`.
- `cadconvert status` shows the free conversions left. In `cadconvert status --json` the `license` object has `state` (`Trial` or `Registered`), `freeFilesRemaining`, `nextOutput` (`clean` or `draft_quality`) and `afterFreeFiles`, the same shape as the other Contenta CLIs; the MCP tool `get_license_status` returns the same object.
- License: **$179 one-time** for a lifetime license; the [buy page](https://www.contenta-software.com/3dcadconverter/buy.php) also lists a quarterly plan. Registering removes the trial exports.
- Register from the CLI with `cadconvert register <email> <key>`, or in the desktop app.

## MCP server

`cadconvert serve` starts an MCP server with 5 tools: `convert_cad`, `get_file_info`, `detect_format`, `list_formats`, `get_license_status`, each with a title and read-only, destructive and network hints. See the [MCP reference](mcp-server.md).
