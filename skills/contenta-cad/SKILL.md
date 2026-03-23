---
name: contenta-cad
description: Convert 3D CAD files using the cadconvert CLI. Use when the user asks to convert STEP, IGES, STL, OBJ, FBX, glTF, 3MF, or other 3D formats, inspect 3D file metadata, or batch convert CAD directories.
allowed-tools: Bash
---

# Contenta CAD Converter

You have access to the `cadconvert` CLI for 3D file conversion. Use `--json` where available for structured output.

## Commands

### Convert a single file
```bash
cadconvert convert -i <input> -o <output> --format <fmt> --tessellation <value> --angular <value> --units <unit> --repair --binary
```

Formats: stl, obj, ply, gltf, glb, 3mf, dae, step, iges, vrml, off

Tessellation controls mesh quality when converting from parametric (STEP/IGES) to mesh:
- `--tessellation 0.5` — draft quality (fast, coarse mesh)
- `--tessellation 0.1` — standard quality (default)
- `--tessellation 0.01` — fine quality (slow, smooth mesh)
- `--tessellation 0.001` — ultra fine quality (very slow, maximum detail)

### Batch convert
```bash
cadconvert batch -i <dir> -o <dir> --format <fmt> --recursive --workers <n>
```

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
cadconvert watch -i <dir> -o <dir> --format <fmt>
```

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
- Trial users can convert freely for 30 days.
