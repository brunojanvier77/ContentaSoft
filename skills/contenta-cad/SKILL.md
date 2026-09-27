---
name: contenta-cad
description: Convert 3D CAD and mesh files with the 3D CAD Converter CLI (cadconvert). Use when the user asks to convert STEP, IGES or BREP to STL, OBJ, 3MF, glTF/GLB, FBX, PLY or other formats, convert between mesh formats, change units, control mesh quality for 3D printing, reduce the polygon count of a mesh, set the up axis for game engines, inspect a 3D file, or batch-convert or watch a folder of CAD files.
allowed-tools: Bash
---

# 3D CAD Converter

Use the `cadconvert` CLI (3D CAD Converter 1.0.24+, Windows). Default per-user install: `%LOCALAPPDATA%\Programs\CadConverter\cadconvert.exe`, on the user PATH. Check with `cadconvert --version`.

No `--json` flag; for structured results use the MCP server (`cadconvert serve`).

## Commands

```bash
cadconvert convert <input> [<output>] [-f <fmt>] [--quality P|N] [--angular RAD] [--up-axis y|z|unchanged] [--decimate R] [--units mm|cm|in|m|ft] [--repair] [--binary true|false]
cadconvert batch <dir> [<outdir>] [-f stl] [-r] [-w N] [same conversion options]
cadconvert watch <dir> [<outdir>] [-f stl] [same conversion options]     # runs until Ctrl+C
cadconvert info <file>
cadconvert formats
cadconvert register -k <key> -e <email>
```

- Paths can be positional or given with `-i`/`-o`.
- No output: `convert` writes next to the input with the `-f` extension (then `-f` is required); `batch` and `watch` write to `<input>_converted` next to the input folder.
- `-f` on `convert` defaults to the output extension; `batch` and `watch` default to `stl`.
- `batch` is recursive by default; `-w` defaults to the number of logical processors.

## Mesh quality (STEP/IGES/BREP sources only)

`--quality` (same option as `--tessellation`) takes a preset or a linear deflection in mm. `--angular` is in **radians** and defaults to the preset's value (0.5 with a numeric deflection).

| Preset | Linear | Angular |
|--------|--------|---------|
| draft | 1.0 | 5.0 |
| standard (default) | 0.1 | 0.5 |
| fine (smooth curves, 3D printing) | 0.01 | 0.1 |
| ultrafine (`ultra` also works) | 0.001 | 0.05 |

Finer settings make much larger files: on a small test part, fine produced about 12x the triangles of standard. Mesh-to-mesh conversions ignore these options.

## Up axis and polygon reduction

- `--up-axis y` for game engines, glTF and three.js; `z` for CAD and 3D printing. A STEP/IGES/BREP re-export is not rotated (the CLI prints a note).
- `--decimate 0.25` (or `25%`) keeps about a quarter of the triangles. Mesh sources only: STL, OBJ, PLY, glTF/GLB, FBX, 3MF, Collada, OFF, VRML, X3D, AMF, DXF. STEP/IGES/BREP are not reduced; lower their `--quality` instead (draft is lightest). DWG and USD are not reduced. Meshes over 2,000,000 triangles are written unreduced. The desktop app has no reduction control.

## Formats

Write: stl, obj, ply, fbx, dae, 3mf, gltf, glb, wrl (vrml), x3d, off, step (.step/.stp), iges (.iges/.igs), brep.
Read only: amf, dwg, dxf, usd, usdz.

BREP input means OpenCascade `.brep`/`.brp` only (not Parasolid `.x_t` or ACIS `.sat`). It has no unit, so it is read as mm, and it comes in as one merged solid without part names, colours or assembly tree.

## Examples (verified on 1.0.24)

```powershell
cadconvert convert model.step model.stl
cadconvert convert model.step -f glb
cadconvert convert model.step -o model_fine.stl --quality fine
cadconvert convert model.step -o model_fine.stl --tessellation 0.01 --angular 0.1
cadconvert convert model.step -o model_y.glb --up-axis y
cadconvert convert -i model.step -o model_in.obj --units in --repair
cadconvert convert scan.stl -o scan_light.glb --decimate 0.25
cadconvert convert -i model.stl -o model_ascii.stl --binary false
cadconvert convert part.brep part.step
cadconvert batch -i .\cad-files -o .\meshes -f obj --quality fine
cadconvert batch .\cad-files
cadconvert info model.step
cadconvert watch .\incoming -f glb --up-axis y
```

## Exit codes

0 success · 1 error · 2 invalid arguments · 3 file not found · 4 conversion failed · 5 license required (trial limit reached or trial ended).

## Guidelines

- Run `cadconvert info` first: it gives the declared unit (3MF, STEP, IGES; glTF is metres) and, for STEP/IGES/BREP, the bounding box. STL, OBJ, PLY and OFF carry no unit and show Unknown.
- Use standard quality unless the user needs smooth curves (fine) or a quick, light preview (draft).
- To shrink a mesh file, use `--decimate`; to shrink a mesh made from STEP/IGES, use a coarser `--quality`.
- STL has no colors or materials; the CLI says so. Use 3MF, OBJ or GLB when color matters.
- glTF (`.gltf`) is JSON plus side files; GLB (`.glb`) is a single binary file and is easier to share.
- Trial: 30 days and 10 conversions, then conversion stops (exit 5) until `cadconvert register -k <key> -e <email>`.
