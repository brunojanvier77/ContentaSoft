# 3D CAD Converter

Turn STEP and IGES files into STL, OBJ, 3MF, glTF/GLB, FBX and other mesh formats on Windows, one file or a whole folder at a time, with control over mesh quality and units. The same install gives you the desktop app, the `cadconvert` CLI and an MCP server for AI agents.

[Download the free trial](https://www.contenta-software.com/3dcadconverter/) | [MCP server reference](mcp-server.md)

Documented version: **1.0.23**.

## What it does

- **CAD to mesh**: STEP, IGES and BREP tessellated with OpenCascade; you set the linear and angular deflection.
- **Mesh to mesh**: STL, OBJ, PLY, FBX, Collada, 3MF, glTF/GLB, VRML, X3D, OFF.
- **19 formats read, 14 written** (table below).
- **Units**: convert to mm, cm, in, m or ft.
- **Mesh repair** on request.
- **Batch** a folder with parallel workers; **watch** a folder and convert new files.

## Install and PATH

The CLI is installed with the desktop app. A default install is per-user, in `%LOCALAPPDATA%\Programs\CadConverter\`, and the installer adds that folder to your user `PATH`. Open a new terminal after installing, then check:

```powershell
cadconvert --version    # 1.0.23 or later
cadconvert formats
```

If `cadconvert` is not found, or `--version` prints an older number, call the exe by its full path or remove the older copy that comes first on `PATH`.

## Commands

| Command | What it does |
|---------|--------------|
| `convert -i <file> -o <file>` | Convert one file |
| `batch -i <dir> -o <dir>` | Convert every 3D file in a folder |
| `info <file>` | Format, units, parts, triangles, vertices, bounding box |
| `formats` | List supported formats |
| `watch -i <dir> -o <dir>` | Convert new files that appear in a folder |
| `register -k <key> -e <email>` | Register a license key |
| `serve` | Start the MCP server (stdio) |

The CLI has no `--json` flag; for structured output use the [MCP server](mcp-server.md).

## Examples

Each of these was run against 1.0.23.

```powershell
# STEP to STL with the default mesh quality
cadconvert convert -i model.step -o model.stl

# Finer mesh for smooth curved surfaces (about 12x the triangles on our test part)
cadconvert convert -i model.step -o model_fine.stl --tessellation 0.01 --angular 0.1

# STEP to GLB for web and AR viewers
cadconvert convert -i model.step -o model.glb

# STEP to OBJ in inches, with mesh repair
cadconvert convert -i model.step -o model_in.obj --units in --repair

# ASCII STL instead of binary
cadconvert convert -i model.stl -o model_ascii.stl --binary false

# STEP to IGES
cadconvert convert -i model.stp -o model.igs

# A folder to OBJ, and a folder to STL (the default format)
cadconvert batch -i .\cad-files -o .\meshes -f obj
cadconvert batch -i .\cad-files -o .\stl-out

# Inspect a file
cadconvert info model.step

# Convert new files dropped into a folder to STL (Ctrl+C to stop)
cadconvert watch -i .\incoming -o .\converted -f stl
```

The output format comes from `-f`, or from the output file extension when `-f` is left out.

## convert options

| Option | Description |
|--------|-------------|
| `-i, --input` | Input file (required) |
| `-o, --output` | Output file (required) |
| `-f, --format` | Target format (default: from the output extension) |
| `--tessellation` | Linear deflection, a number (default `0.1`). Smaller = finer mesh, bigger file |
| `--angular` | Angular deflection in **radians** (default `0.5`, about 29 degrees). Smaller = smoother curves |
| `--repair` | Mesh repair |
| `--units` | `mm`, `cm`, `in`, `m`, `ft` |
| `--binary` | Binary STL/PLY (default `true`); `--binary false` writes ASCII |

`--tessellation` and `--angular` apply when the source is STEP, IGES or BREP. Mesh-to-mesh conversions ignore them.

Starting points for `--tessellation` / `--angular` (these are the values behind the MCP server's named presets):

| Quality | `--tessellation` | `--angular` |
|---------|------------------|-------------|
| Draft | `1.0` | `5.0` |
| Standard (default) | `0.1` | `0.5` |
| Fine | `0.01` | `0.1` |
| Ultra fine | `0.001` | `0.05` |

## batch and watch options

`batch`: `-i/--input` and `-o/--output` folders (required), `-f/--format` (default `stl`), `-r/--recursive` (default on), `-w/--workers` (default: the number of logical processors). Batch uses the default mesh quality.

`watch`: `-i/--input` and `-o/--output` folders (required), `-f/--format` (default `stl`).

## info

`info` prints format, units, part count, triangle and vertex counts, materials/textures and, for STEP files, the bounding box. For mesh formats it reports counts only (units show as Unknown). In 1.0.23, `info` on an IGES file reports it as an empty STL; converting IGES files works normally.

## Supported formats

| Format | Extensions | Read | Write | Type |
|--------|-----------|------|-------|------|
| STEP | `.step`, `.stp` | Yes | Yes | CAD |
| IGES | `.iges`, `.igs` | Yes | Yes | CAD |
| BREP | `.brep` | Yes | Yes | CAD |
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
