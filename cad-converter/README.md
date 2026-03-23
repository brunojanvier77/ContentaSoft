# 3D CAD Batch Converter

Convert between STEP, IGES, and 15+ 3D mesh formats on Windows.

[Download Free Trial](https://contenta-software.com)

## Features

- **STEP/IGES input** — read parametric CAD via OpenCascade
- **15+ output formats** — STL, OBJ, PLY, FBX, DAE (Collada), 3MF, glTF/GLB
- **Tessellation presets** — Draft, Standard, Fine, Ultra Fine
- **Mesh-to-mesh** — convert between STL, OBJ, PLY, FBX, DAE, 3MF, glTF via Assimp
- **Unit conversion** — mm to inches and vice versa
- **Batch processing** — convert entire directories

## CLI Reference

**Executable**: `cadconvert`

| Command | Description |
|---------|-------------|
| `convert` | Convert a single CAD/mesh file |
| `batch` | Batch convert a directory |
| `info` | Show file info and metadata |
| `formats` | List supported formats |
| `watch` | Watch folder for auto-conversion |
| `register` | Register a license key |

## CLI Examples

```bash
# Convert STEP to STL
cadconvert convert model.step --format stl --tessellation fine

# Batch convert STEP files to OBJ
cadconvert batch --input ./cad-files --output ./meshes --format obj

# Convert mesh format
cadconvert convert model.stl --format glb
```

## Supported Formats

**Input (BREP)**: STEP (.step, .stp), IGES (.iges, .igs)
**Input/Output (Mesh)**: STL, OBJ, PLY, FBX, DAE (Collada), 3MF, glTF (.gltf), GLB (.glb)
