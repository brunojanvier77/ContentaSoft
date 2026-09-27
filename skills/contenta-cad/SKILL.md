---
name: contenta-cad
description: Convert 3D CAD and mesh files with the 3D CAD Converter CLI (cadconvert). Use when the user asks to convert STEP, IGES or BREP to STL, OBJ, 3MF, glTF/GLB, FBX, PLY or other formats, convert between mesh formats, change units, control mesh quality for 3D printing, inspect a 3D file, or batch-convert or watch a folder of CAD files.
allowed-tools: Bash
---

# 3D CAD Converter

Use the `cadconvert` CLI (3D CAD Converter 1.0.23+, Windows). Default per-user install: `%LOCALAPPDATA%\Programs\CadConverter\cadconvert.exe`, on the user PATH. Check with `cadconvert --version`.

No `--json` flag; for structured results use the MCP server (`cadconvert serve`).

## Commands

```bash
cadconvert convert -i <input> -o <output> [-f <fmt>] [--tessellation N] [--angular RAD] [--units mm|cm|in|m|ft] [--repair] [--binary true|false]
cadconvert batch -i <dir> -o <dir> [-f stl] [-r] [-w N]
cadconvert info <file>
cadconvert formats
cadconvert watch -i <dir> -o <dir> [-f stl]        # runs until Ctrl+C
cadconvert register -k <key> -e <email>
```

- `-i` and `-o` are required flags (no positional paths).
- `-f` is optional on `convert` (taken from the output extension); `batch` and `watch` default to `stl`.
- `batch` is recursive by default; `-w` defaults to the number of logical processors. It uses the default mesh quality (no `--tessellation` on batch).

## Mesh quality (STEP/IGES/BREP sources only)

`--tessellation` is the linear deflection (number, default 0.1). `--angular` is the angular deflection in **radians** (default 0.5, about 29 degrees).

| Quality | `--tessellation` | `--angular` |
|---------|------------------|-------------|
| Draft | 1.0 | 5.0 |
| Standard (default) | 0.1 | 0.5 |
| Fine (smooth curves, 3D printing) | 0.01 | 0.1 |
| Ultra fine | 0.001 | 0.05 |

Finer settings make much larger files: on a small test part, fine produced about 12x the triangles of standard. Mesh-to-mesh conversions ignore both options.

## Formats

Write: stl, obj, ply, fbx, dae, 3mf, gltf, glb, wrl (vrml), x3d, off, step (.step/.stp), iges (.iges/.igs), brep.
Read only: amf, dwg, dxf, usd, usdz.

## Examples (verified on 1.0.23)

```powershell
cadconvert convert -i model.step -o model.stl
cadconvert convert -i model.step -o model_fine.stl --tessellation 0.01 --angular 0.1
cadconvert convert -i model.step -o model.glb
cadconvert convert -i model.step -o model_in.obj --units in --repair
cadconvert convert -i model.stl -o model_ascii.stl --binary false
cadconvert batch -i .\cad-files -o .\meshes -f obj
cadconvert info model.step
cadconvert watch -i .\incoming -o .\converted -f stl
```

## Exit codes

0 success · 1 error · 2 invalid arguments · 3 file not found · 4 conversion failed · 5 license required (trial limit reached or trial ended).

## Guidelines

- Run `cadconvert info` on STEP files first: it gives units and the bounding box. For mesh files it only gives counts, and in 1.0.23 it misreports IGES files (convert them anyway; conversion works).
- Use standard quality unless the user needs smooth curves (fine) or a quick preview (draft).
- STL has no colors or materials; the CLI says so. Use 3MF, OBJ or GLB when color matters.
- glTF (`.gltf`) is JSON plus side files; GLB (`.glb`) is a single binary file and is easier to share.
- Trial: 30 days and 10 conversions, then conversion stops (exit 5) until `cadconvert register -k <key> -e <email>`.
