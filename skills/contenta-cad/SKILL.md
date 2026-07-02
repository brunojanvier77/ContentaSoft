---
name: contenta-cad
description: Convert 3D CAD files using the cadconvert CLI. Use when the user asks to convert STEP, IGES, STL, OBJ, FBX, glTF, 3MF, or other 3D formats, inspect 3D file metadata, or batch convert CAD directories.
allowed-tools: Bash
---

# Contenta CAD Converter

You have access to the `cadconvert` CLI for 3D file conversion (default install: `C:\Program Files\ContentaSoft\3D CAD Converter\cadconvert.exe`). The CLI has no `--json` flag — for structured output, use the MCP server (`cadconvert serve`).

## Commands

### Convert a single file
```bash
cadconvert convert -i <input> -o <output> [-f <fmt>] [--tessellation <value>] [--angular <degrees>] [--units <unit>] [--repair] [--binary <true|false>]
```

`-i` and `-o` are required. `-f/--format` is optional — inferred from the output extension.

Example: `cadconvert convert -i model.step -o model.stl --tessellation 0.01`

Export formats: stl, obj, ply, gltf, glb, 3mf, dae, fbx, vrml, off, x3d, step, iges, brep
Import-only: amf, dwg, dxf, usd, usdz

Tessellation controls mesh quality when converting from parametric (STEP/IGES/BREP) to mesh. It is **numeric** (linear deflection — no keyword presets):
- `--tessellation 0.5` — draft quality (fast, coarse mesh)
- `--tessellation 0.1` — standard quality (default)
- `--tessellation 0.01` — fine quality (slow, smooth mesh)
- `--tessellation 0.001` — ultra fine quality (very slow, maximum detail)

`--angular` sets angular deflection in degrees (default: 0.5).

### Batch convert
```bash
cadconvert batch -i <dir> -o <dir> -f <fmt> [--recursive] [--workers <n>]
```
`-f` defaults to stl, `--recursive` defaults to true, `--workers` defaults to 24.

### Inspect a 3D file
```bash
cadconvert info <file>
```
Returns: format, units, part count, triangle count, vertex count, bounding box, materials, textures.

### List supported formats
```bash
cadconvert formats
```

### Watch folder for auto-conversion
```bash
cadconvert watch -i <dir> -o <dir> -f <fmt>
```

## Exit Codes

`0` Success · `1` Error · `2` Invalid arguments · `3` File not found · `4` Conversion failed · `5` License required

## Format Routing

The converter automatically picks the right engine:

| Source | Target | Engine |
|--------|--------|--------|
| STEP/IGES | Any mesh (STL, OBJ, etc.) | OpenCascade (tessellation required) |
| Any mesh | Any mesh | Assimp (fast, in-process) |
| Any mesh | STEP/IGES | OpenCascade |

## Unit Conversion

Use `--units` to convert between unit systems:
- `mm` — millimeters (default for most CAD)
- `cm` — centimeters
- `in` — inches
- `m` — meters
- `ft` — feet

Example: `cadconvert convert -i part.step -o part.stl --units in` converts to inches.

## Mesh Repair

Add `--repair` to fix common mesh issues:
- Non-manifold edges
- Holes in the mesh
- Degenerate faces
- Self-intersections

## Guidelines

- For STEP/IGES to mesh, tessellation quality matters. Use `0.1` (standard) for most cases. Use `0.01` (fine) for curved surfaces that need to look smooth. Use `0.5` (draft) for quick previews.
- For mesh-to-mesh (e.g., STL to OBJ), tessellation is ignored — the conversion is direct and fast.
- Use `--binary` (default: true) for STL and PLY to get smaller files. Use `--binary false` for ASCII output when human-readability matters.
- Run `cadconvert info <file>` first to understand what you're working with before converting.
- glTF (`.gltf`) is JSON-based, GLB (`.glb`) is the binary equivalent. Prefer GLB for distribution.
- Trial: 30 days from install, watermark after 10 files per session; conversion is blocked after the trial expires (exit code 5).
