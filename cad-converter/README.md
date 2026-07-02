# 3D CAD Converter

Convert between STEP, IGES, and 16+ 3D mesh formats on Windows.

[Download Free Trial](https://www.contenta-software.com/3dcadconverter/)

## Features

- **STEP/IGES/BREP input** — read parametric CAD via OpenCascade
- **19 supported formats** — 14 import+export, 5 import-only (see table below)
- **Tessellation control** — numeric linear/angular deflection for full quality control
- **Mesh-to-mesh** — convert between STL, OBJ, PLY, FBX, DAE, 3MF, glTF via Assimp
- **Unit conversion** — mm, cm, in, m, ft
- **Batch processing** — convert entire directories, parallel workers
- **Watch folders** — auto-convert files as they appear

## Install & PATH

The installer places the app in `C:\Program Files\ContentaSoft\3D CAD Converter\` (default). The CLI executable is `cadconvert.exe` in that directory.

To use `cadconvert` from any terminal, either add the install directory to your `PATH`, or call it with the full path:

```bash
"C:\Program Files\ContentaSoft\3D CAD Converter\cadconvert.exe" formats
```

Verify your installation:

```bash
cadconvert formats
```

This prints the supported-format table — if you see it, you're ready to convert.

## Quick Start

```bash
# STEP → STL for 3D printing (fine tessellation for curved surfaces)
cadconvert convert -i model.step -o model.stl --tessellation 0.01

# Batch convert a folder of CAD files to OBJ
cadconvert batch -i ./cad-files -o ./meshes -f obj

# STEP → GLB for web/AR viewers
cadconvert convert -i model.step -o model.glb

# Watch a folder and auto-convert new files to STL
cadconvert watch -i ./incoming -o ./converted -f stl
```

## CLI Reference

**Executable**: `cadconvert` (v1.0.9, .NET 9)

| Command | Description |
|---------|-------------|
| `convert` | Convert a single CAD/mesh file |
| `batch` | Batch convert a directory |
| `info <file>` | Show file info and metadata |
| `formats` | List supported formats |
| `watch` | Watch folder for auto-conversion |
| `register` | Register a license key (`-k/--key` and `-e/--email`, both required) |
| `serve` | Start the MCP server (stdio) |

### `convert` options

| Option | Description |
|--------|-------------|
| `-i, --input` | Input file (**required**) |
| `-o, --output` | Output file (**required**) |
| `-f, --format` | Target format (inferred from output extension if omitted) |
| `--tessellation` | Linear deflection, numeric (default: `0.1`) |
| `--angular` | Angular deflection in degrees (default: `0.5`) |
| `--repair` | Enable mesh repair |
| `--units` | Target units: `mm`, `cm`, `in`, `m`, `ft` |
| `--binary` | Binary output for STL/PLY (default: `true`) |

### `batch` options

| Option | Description |
|--------|-------------|
| `-i, --input` | Input directory (**required**) |
| `-o, --output` | Output directory (**required**) |
| `-f, --format` | Target format (default: `stl`) |
| `-r, --recursive` | Include subdirectories (default: `true`) |
| `-w, --workers` | Parallel workers (default: `24`) |

### `watch` options

| Option | Description |
|--------|-------------|
| `-i, --input` | Watched directory (**required**) |
| `-o, --output` | Output directory (**required**) |
| `-f, --format` | Target format (default: `stl`) |

### Tessellation values

`--tessellation` takes a **numeric** linear deflection value (there are no keyword presets in the CLI). Smaller = finer mesh, larger files, slower conversion:

| Quality | `--tessellation` | Use for |
|---------|-----------------|---------|
| Draft | `0.5` | Quick previews |
| Standard | `0.1` | Default — most conversions |
| Fine | `0.01` | Curved surfaces that must look smooth |
| Ultra Fine | `0.001` | Maximum detail |

Tessellation only applies when converting from parametric formats (STEP, IGES, BREP) to mesh. Mesh-to-mesh conversions ignore it.

### Exit codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error |
| 2 | Invalid arguments |
| 3 | File not found |
| 4 | Conversion failed |
| 5 | License required |

The CLI has no `--json` flag — for structured output, use the [MCP server](mcp-server.md).

## Supported Formats

| Format | Extensions | Import | Export | Category |
|--------|-----------|--------|--------|----------|
| 3MF | `.3mf` | Yes | Yes | Mesh |
| AMF | `.amf` | Yes | No | Mesh |
| BREP | `.brep` | Yes | Yes | Parametric |
| Collada | `.dae` | Yes | Yes | Mesh |
| DWG | `.dwg` | Yes | No | Mesh |
| DXF | `.dxf` | Yes | No | Mesh |
| FBX | `.fbx` | Yes | Yes | Mesh |
| GLB | `.glb` | Yes | Yes | Mesh |
| glTF | `.gltf` | Yes | Yes | Mesh |
| IGES | `.iges`, `.igs` | Yes | Yes | Parametric |
| OBJ | `.obj` | Yes | Yes | Mesh |
| OFF | `.off` | Yes | Yes | Mesh |
| PLY | `.ply` | Yes | Yes | Mesh |
| STEP | `.step`, `.stp` | Yes | Yes | Parametric |
| STL | `.stl` | Yes | Yes | Mesh |
| USD | `.usd` | Yes | No | Mesh |
| USDZ | `.usdz` | Yes | No | Mesh |
| VRML | `.wrl` | Yes | Yes | Mesh |
| X3D | `.x3d` | Yes | Yes | Mesh |

Run `cadconvert formats` for the authoritative list from your installed version.

## Trial

The free trial runs for **30 days from install** and adds a watermark after **10 files per session**. After the trial expires, conversion is blocked until you [register a license](https://www.contenta-software.com/3dcadconverter/) with `cadconvert register -k <key> -e <email>`.

## MCP Server

The CAD Converter includes a built-in MCP server with 4 tools for AI agent integration. See the [full MCP documentation](mcp-server.md).

```bash
cadconvert serve
```

Tools: `convert_cad`, `get_file_info`, `detect_format`, `list_formats`.
