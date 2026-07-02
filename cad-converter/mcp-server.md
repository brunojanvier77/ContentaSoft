# 3D CAD Converter — MCP Server

The CAD Converter exposes 4 tools for 3D file conversion and analysis via the [Model Context Protocol](https://modelcontextprotocol.io/) (MCP).

## Getting Started

```json
{
  "mcpServers": {
    "cad-converter": {
      "command": "C:\\Program Files\\ContentaSoft\\3D CAD Converter\\cadconvert.exe",
      "args": ["serve"]
    }
  }
}
```

If the install directory is on your `PATH`, `"command": "cadconvert"` also works.

## Protocol Details

| Property | Value |
|----------|-------|
| Transport | stdio (stdin/stdout) |
| Protocol | JSON-RPC 2.0 |
| Protocol Version | `2024-11-05` |
| Server Name | `cad-converter` |
| Server Version | `2026.1.0` |

## Tools

### convert_cad

Convert a 3D CAD file (STEP, IGES, BREP) to mesh formats (STL, OBJ, PLY, FBX, glTF, 3MF, …) or between mesh formats.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `input_path` | string | Yes | Absolute path to the source 3D file |
| `output_path` | string | Yes | Output file path |
| `format` | string | No | Target format: `stl`, `obj`, `ply`, `gltf`, `glb`, `3mf`, `dae`, `step`, `iges`, `fbx`, `vrml`, `off`, `brep`. If omitted, inferred from the output file extension |
| `tessellation` | string | No | Tessellation preset: `draft`, `standard`, `fine`, `ultrafine` (default: standard) |
| `units` | string | No | Target units: `mm`, `cm`, `in`, `m`, `ft` |
| `repair` | boolean | No | Enable mesh repair |
| `binary` | boolean | No | Binary output for STL/PLY (default: true) |

Unlike the CLI (which takes numeric deflection values), the MCP tool accepts keyword presets that map to these deflection values:

| Preset | Linear deflection | Angular deflection (°) |
|--------|------------------|------------------------|
| `draft` | 1.0 | 5.0 |
| `standard` | 0.1 | 0.5 |
| `fine` | 0.01 | 0.1 |
| `ultrafine` | 0.001 | 0.05 |

**Returns**: success, input, output, format, durationMs.

---

### get_file_info

Get metadata about a 3D file including format, parts, triangles, vertices, and bounding box.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Absolute path to the 3D file |

**Returns**: format, units, partCount, triangleCount, vertexCount, hasMaterials, hasTextures, boundingBox (minX/Y/Z, maxX/Y/Z).

---

### detect_format

Detect the 3D file format and check if it is supported for import/export.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Absolute path to the file |

**Returns**: format, extension, displayName, description, canImport, canExport, category (Parametric or Mesh).

---

### list_formats

List all supported 3D file formats with import/export capabilities. No parameters required.

**Returns**: Array of format info with:

| Category | Formats |
|----------|---------|
| **Parametric** | STEP (.step, .stp), IGES (.iges, .igs), BREP (.brep) — all import + export |
| **Mesh — import + export** | STL, OBJ, PLY, FBX, Collada (.dae), 3MF, glTF (.gltf), GLB (.glb), VRML (.wrl), X3D, OFF |
| **Mesh — import only** | AMF, DWG, DXF, USD, USDZ |
