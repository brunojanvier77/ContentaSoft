# 3D CAD Converter: MCP server

`cadconvert serve` exposes 5 tools over the [Model Context Protocol](https://modelcontextprotocol.io/), so an AI agent (Claude Code, Claude Desktop, Cursor, Windsurf, Codex and others) can work on files on your PC.

Generated from `cadconvert catalog`, version 1.0.35. The same reference is on the website: [www.contenta-software.com/3dcadconverter/docs/cli.html](https://www.contenta-software.com/3dcadconverter/docs/cli.html). The command line is documented in [cli.md](cli.md).

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

If your AI client cannot find `cadconvert`, give the full path, for example `"C:\\Users\\<you>\\AppData\\Local\\Programs\\CadConverter\\cadconvert.exe"`. The [MCP config guide](../mcp-config/) covers each client. In Claude Code one command adds it:

```powershell
claude mcp add cad-converter -- cadconvert serve
```

Pass absolute paths to every tool.

## Protocol

| Property | Value |
|---|---|
| Transport | stdio, newline-delimited JSON-RPC 2.0 |
| Protocol versions | `2025-11-25`, `2025-06-18`, `2025-03-26`, `2024-11-05`. The server answers with the version the client asks for; for any other version it answers `2025-11-25` |
| Capabilities | `tools` only (no resources or prompts) |
| Server name | `cad-converter` |
| Server version | `1.0.35` |

All four ContentaSoft servers run on the same host, which behaves like this:

- **stdout carries JSON-RPC only.** Logs go to stderr, so a client can parse every line it reads.
- **`ping` is answered while a tool runs**, so a long job does not look like a hung server.
- **`notifications/cancelled` cancels a running call.** The call stops and, as the specification says, gets no reply.
- **Tool calls run one at a time**, in the order they arrive.
- **Errors**: an unknown tool or method is a JSON-RPC error; a tool that cannot do its job returns a normal result with `isError: true` and a sentence that names the problem.
- **Licence first**: `get_license_status` returns the licence and trial state, so an agent can tell the user what a run will produce before it starts one.

## Tool annotations

| Tool | Title | Read-only | Destructive | Idempotent | Network |
|---|---|---|---|---|---|
| [`convert_cad`](#convert_cad) | Convert 3D/CAD file | no | yes | no | no |
| [`detect_format`](#detect_format) | Detect 3D format | yes | no | yes | no |
| [`get_file_info`](#get_file_info) | Get 3D file info | yes | no | yes | no |
| [`get_license_status`](#get_license_status) | Get license and trial status | yes | no | yes | no |
| [`list_formats`](#list_formats) | List supported formats | yes | no | yes | no |

Destructive: the tool can replace a file that already exists. Network: the tool uses the internet.

## Tools

### convert_cad

**Convert 3D/CAD file.** Convert a 3D CAD file (STEP, IGES) to mesh formats (STL, OBJ, PLY, FBX, glTF, 3MF) or between mesh formats. Mesh sources include VRML (.wrl, .vrml, gzip .wrz): VRML is read in metres (a 1 m cube is 1000 mm) and Y-up, as the specification says. The result's `notes` list what an option could not do on this route (a STEP source is tessellated, not reduced; a STEP re-export cannot be rotated).

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_path` | string | yes | Absolute path to the source 3D file. |
| `output_path` | string | no | Absolute path for the converted output file. Give this or output_dir. |
| `output_dir` | string | no | Absolute path of a folder: the output is written there, named after the input with the format's extension (format is then required). Give this or output_path. |
| `format` | string | no | Output format: stl, obj, ply, gltf, glb, 3mf, dae, step, iges, fbx, vrml, off, brep. One of: `stl`, `obj`, `ply`, `gltf`, `glb`, `3mf`, `dae`, `step`, `iges`, `fbx`, `vrml`, `off`, `brep`. |
| `quality` | string | no | Mesh quality for STEP/IGES/BREP sources: a preset (draft, standard, fine, ultrafine) or a linear deflection in mm ("0.1" = standard; lower = finer, e.g. "0.001"). Default: standard. No effect on mesh sources (STL, OBJ ...): they keep their triangles. |
| `angular` | number or string | no | Angular deflection in radians for STEP/IGES/BREP sources (0.5 = about 29 degrees; lower = smoother curves). Default: the preset's value (0.5 for standard and for a numeric quality). |
| `units` | string | no | Target units for conversion. One of: `mm`, `cm`, `in`, `m`, `ft`. |
| `up_axis` | string | no | Rotate the model so this axis points up: y (game engines, glTF, three.js) or z (CAD, 3D printing). Default: unchanged. A STEP/IGES/BREP re-export cannot be rotated and the result's notes say so. One of: `y`, `z`, `unchanged`. |
| `decimate` | number or string | no | Reduce mesh sources (STL, OBJ, PLY, glTF, FBX, 3MF ...) to this share of their triangles: 0.25 or "25%" keeps about a quarter. STEP/IGES/BREP sources are tessellated, not reduced - use quality instead; the result's notes say so. |
| `max_size_mb` | number or string | no | glTF/GLB only: the largest file to write, in MB (5, 0.5, 15; 1 MB = 1,000,000 bytes). A file over it is written again with fewer triangles until it fits, down to 5% of its triangles. The result's withinBudget is true when the written file fits and false when it was written at the smallest size reached and is still over; outputBytes, trianglesBefore and trianglesAfter are what was measured. |
| `repair` | boolean | no | Repair STEP/IGES/BREP surfaces before meshing (fixes broken edges and faces, closes small gaps). No effect on mesh sources such as STL or OBJ. |
| `binary` | boolean | no | Use binary output for STL/PLY (default: true). |
| `overwrite` | boolean | no | Replace an output file that already exists (default: the existing file is kept and the new one is written beside it as name (1)). |
| `skip_existing` | boolean | no | Leave the input alone when its output file already exists: nothing is converted and no free conversion is used; the result says skipped: true. |

### detect_format

**Detect 3D format.** Detect the 3D file format and check if it is supported for import/export.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_path` | string | yes | Absolute path to the 3D file. |

### get_file_info

**Get 3D file info.** Get metadata about a 3D file including format, parts, triangles, vertices, and bounding box.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_path` | string | yes | Absolute path to the 3D file. |

### get_license_status

**Get license and trial status.** Get the license and trial state before converting: registered or trial, free conversions left, trial days left, and what exports become after the trial (afterFreeFiles: draft_quality = Draft tessellation and a trial note in the file). nextOutput says what the next export will be: clean or draft_quality.

No parameters.

### list_formats

**List supported formats.** List all supported 3D file formats with import/export capabilities.

No parameters.
