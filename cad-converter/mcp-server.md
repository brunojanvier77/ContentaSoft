# 3D CAD Converter: MCP server

`cadconvert serve` exposes 4 tools over the [Model Context Protocol](https://modelcontextprotocol.io/), so an AI agent (Claude Desktop, Claude Code, Cursor, Windsurf and others) can inspect and convert 3D files on your PC.

## Setup

```json
{
  "mcpServers": {
    "cad-converter": {
      "command": "cadconvert",
      "args": ["serve"]
    }
  }
}
```

If your AI client cannot find `cadconvert`, use the full path, for example `"C:\\Users\\<you>\\AppData\\Local\\Programs\\CadConverter\\cadconvert.exe"`. See the [MCP config guide](../mcp-config/) for each client.

Pass absolute paths to every tool.

## Protocol

| Property | Value |
|----------|-------|
| Transport | stdio |
| Protocol | JSON-RPC 2.0, MCP `2024-11-05` |
| Server name | `cad-converter` |
| Server version | `1.0.25` |

**License**: the server runs during the 30-day trial and for registered copies; after the trial it refuses to start. The trial includes 10 conversions; after that `convert_cad` returns a "Trial limit reached" error until you register.

## Tools

### convert_cad

Convert STEP, IGES or BREP to a mesh format, or convert between mesh formats.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `input_path` | string | Yes | Absolute path to the source file |
| `output_path` | string | Yes | Absolute path for the output file |
| `format` | string | No | `stl`, `obj`, `ply`, `gltf`, `glb`, `3mf`, `dae`, `step`, `iges`, `fbx`, `vrml`, `off`, `brep` (default: from the output extension) |
| `tessellation` | string | No | `draft`, `standard` (default), `fine`, `ultrafine` |
| `units` | string | No | Target unit: `mm` (default), `cm`, `in`, `m`, `ft`. The source is read in the unit it declares; see [Units](README.md#units) |
| `repair` | boolean | No | Repairs STEP, IGES and BREP surfaces before meshing (fixes broken edges and faces, closes small gaps). No effect on mesh sources such as STL or OBJ |
| `binary` | boolean | No | Binary STL/PLY (default true) |

The tessellation presets set these deflections (angular deflection in radians, the same unit as the CLI's `--angular`):

| Preset | Linear | Angular |
|--------|--------|---------|
| `draft` | 1.0 | 5.0 |
| `standard` | 0.1 | 0.5 |
| `fine` | 0.01 | 0.1 |
| `ultrafine` | 0.001 | 0.05 |

**Returns**: success, input, output, format, durationMs. Two more fields appear only when they apply: `meshFallback` (a fine mesh took too long and a coarser one was used) and `warning` (a note about the conversion).

`convert_cad` has no up-axis or polygon-reduction parameter. For those, run the CLI (`cadconvert convert ... --up-axis y` or `--decimate 0.25`); see the [CLI reference](README.md#conversion-options).

---

### get_file_info

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Absolute path to the 3D file |

**Returns**: path, fileName, fileSize, format, units, boundingBoxUnits, partCount, triangleCount, vertexCount, hasMaterials, hasTextures, boundingBox (minX/minY/minZ, maxX/maxY/maxZ; STEP, IGES, BREP and USD files).

`units` is the unit the file declares: STEP, IGES and 3MF carry one, glTF/GLB is always metres, FBX has `UnitScaleFactor`, Collada `<unit>` and USD `metersPerUnit` (centimetres when absent). A BREP file (which stores none) is read as millimetres. STL, OBJ, PLY and OFF carry no unit. `boundingBoxUnits` is the unit the bounding box is in.

---

### detect_format

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Absolute path to the file |

**Returns**: path, supported, format, extension, displayName, description, canImport, canExport, category (`Parametric` or `Mesh`).

---

### list_formats

No parameters. **Returns** `formats`: an array with format, extension, displayName, description, canImport, canExport and category for each of the 19 formats.

## Example requests

> "Convert C:\Parts\bracket.step to STL for printing, smooth curves please."
>
> The agent calls `convert_cad` with `input_path`, `output_path: "C:\\Parts\\bracket.stl"`, `tessellation: "fine"`.

> "How big is this part?"
>
> The agent calls `get_file_info` and reads `boundingBox` and `units`.
