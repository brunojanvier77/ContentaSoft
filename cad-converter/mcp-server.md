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

In Claude Code on Windows one command adds it, and nothing else needs installing because the client calls the installed `cadconvert.exe` directly:

```powershell
claude mcp add cad-converter -- cadconvert serve
```

Pass absolute paths to every tool. `convert_cad` refuses a relative `input_path` or `output_path` with an error, because a relative path would resolve against a folder the caller cannot see.

## Protocol

| Property | Value |
|----------|-------|
| Transport | stdio, newline-delimited JSON-RPC 2.0 |
| Protocol versions | `2025-11-25`, `2025-06-18`, `2025-03-26`, `2024-11-05`. The server answers with the version the client asks for; for any other version it answers `2025-11-25` |
| Capabilities | `tools` only (no resources or prompts) |
| Server name | `cad-converter` |
| Server version | `1.0.27` |

All four ContentaSoft servers run on the same host, which behaves like this:

- **stdout carries JSON-RPC only.** Logs and anything else go to stderr, so a client can parse every line it reads.
- **`ping` is answered while a tool runs**, so a long conversion does not look like a hung server.
- **`notifications/cancelled` cancels a running call.** The call stops and, as the specification says, gets no reply.
- **Tool calls run one at a time**, in the order they arrive. `ping`, `tools/list` and cancellations are handled in between.
- **Errors**: an unknown tool or method is a JSON-RPC error (`-32602`, `-32601`); a tool that cannot do its job returns a normal result with `isError: true` and a sentence that names the argument and what it accepts. A call missing a required argument says which one.
- **Every tool has a `title` and `annotations`** (`readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint`); see the next section.

**Trial**: the server never refuses to start. The first 10 conversions (within 30 days of the first launch, plus 10 after the newsletter confirmation in the app) are at full quality. After that `convert_cad` still converts, as a trial export: STEP, IGES and BREP to a mesh format is meshed at `draft` whatever `tessellation` says, and a trial note goes into the file where the format has a place for one. The result then has `trialExport: true` and a `buyUrl`. See [Trial and license](README.md#trial-and-license). Register with `cadconvert register -k <key> -e <email>`.

## Tool annotations

| Tool | Title | Read-only | Destructive | Network |
|------|-------|:---------:|:-----------:|:-------:|
| `convert_cad` | Convert 3D/CAD file | no | yes | no |
| `get_file_info` | Get 3D file info | yes | no | no |
| `detect_format` | Detect 3D format | yes | no | no |
| `list_formats` | List supported formats | yes | no | no |

`convert_cad` is marked destructive because it writes to the exact `output_path` you give it and can replace a file that is already there (an output that would replace its own source is saved under another name instead). No tool uses the network. The three read-only tools are also marked idempotent.

## Tools

### convert_cad

Convert STEP, IGES or BREP to a mesh format, or convert between mesh formats, including VRML (`.wrl`, `.vrml`, gzipped `.wrz`). VRML is read in metres and Y-up, as the specification says, so a 1 m cube is 1000 mm.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `input_path` | string | Yes | Absolute path to the source file |
| `output_path` | string | Yes | Absolute path for the output file |
| `format` | string | No | `stl`, `obj`, `ply`, `gltf`, `glb`, `3mf`, `dae`, `step`, `iges`, `fbx`, `vrml`, `off`, `brep` (default: from the output extension). A `format` that contradicts the output extension is an error |
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

**Returns**: success, input, output (the file that was written, which can differ from `output_path` when the requested name would have replaced the source), format, durationMs. More fields appear only when they apply: `meshFallback` (a fine mesh took too long and a coarser one was used), `warning` (a note about the conversion), `trialExport` and `buyUrl` (the output is a trial export, see above).

Arguments are checked before anything is converted: a missing or non-string path, a relative path, an unknown `format`, `tessellation` or `units` value and a `format` that does not match the output extension are errors that name the argument.

`convert_cad` has no up-axis or polygon-reduction parameter. For those, run the CLI (`cadconvert convert ... --up-axis y` or `--decimate 0.25`); see the [CLI reference](README.md#conversion-options).

---

### get_file_info

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Absolute path to the 3D file |

**Returns**: path, fileName, fileSize, format, units, boundingBoxUnits, partCount, triangleCount, vertexCount, hasMaterials, hasTextures, boundingBox (minX/minY/minZ, maxX/maxY/maxZ; STEP, IGES, BREP, USD and VRML files).

A file that is missing, is not a 3D format or contains no readable geometry returns an error (`isError: true`) with the reason, not a result of zeros.

`units` is the unit the file declares: STEP, IGES and 3MF carry one, glTF/GLB and VRML are always metres, FBX has `UnitScaleFactor`, Collada `<unit>` and USD `metersPerUnit` (centimetres when absent). A BREP file (which stores none) is read as millimetres. STL, OBJ, PLY and OFF carry no unit. `boundingBoxUnits` is the unit the bounding box is in.

---

### detect_format

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Absolute path to the file |

**Returns**: path, supported, format, extension, displayName, description, canImport, canExport, category (`Parametric` or `Mesh`). A `.wrz` file reports format `Vrml` with `canExport: false`: gzipped VRML is read-only.

---

### list_formats

No parameters. **Returns** `formats`: an array with format, extension, extensions (every extension of the format), displayName, description, canImport, canExport and category for each of the 19 formats.

## Example requests

> "Convert C:\Parts\bracket.step to STL for printing, smooth curves please."
>
> The agent calls `convert_cad` with `input_path`, `output_path: "C:\\Parts\\bracket.stl"`, `tessellation: "fine"`.

> "How big is this part?"
>
> The agent calls `get_file_info` and reads `boundingBox` and `units`.
