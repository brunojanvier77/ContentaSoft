# 3D CAD Converter: MCP server

`cadconvert serve` exposes 9 tools over the [Model Context Protocol](https://modelcontextprotocol.io/), so an AI agent (Claude Code, Claude Desktop, Cursor, Windsurf, Codex and others) can work on files on your PC.

Generated from `cadconvert catalog`, version 1.1.1. The same reference is on the website: [www.contenta-software.com/3dcadconverter/docs/cli.html](https://www.contenta-software.com/3dcadconverter/docs/cli.html). The command line is documented in [cli.md](cli.md).

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
| Server version | `1.1.1` |

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
| [`batch_convert_cad`](#batch_convert_cad) | Convert a folder of 3D/CAD files | no | yes | no | no |
| [`convert_cad`](#convert_cad) | Convert 3D/CAD file | no | yes | no | no |
| [`detect_format`](#detect_format) | Detect 3D format | yes | no | yes | no |
| [`get_bom`](#get_bom) | Get the part list of a STEP assembly | yes | no | yes | no |
| [`get_file_info`](#get_file_info) | Get 3D file info | yes | no | yes | no |
| [`get_license_status`](#get_license_status) | Get license and trial status | yes | no | yes | no |
| [`list_formats`](#list_formats) | List supported formats | yes | no | yes | no |
| [`register_license`](#register_license) | Register a license key | no | no | yes | yes |
| [`render_model`](#render_model) | Render 3D model thumbnails | yes | no | yes | no |

Destructive: the tool can replace a file that already exists. Network: the tool uses the internet.

## Tools

### batch_convert_cad

**Convert a folder of 3D/CAD files.** Convert every 3D file in a folder (STEP, IGES, BREP and the mesh formats convert_cad reads) to one format, with the same options as convert_cad; format, or a preset that names one, is required. Outputs are named after their sources; with recursive, subfolders are mirrored under output_dir. Files already in the target format are left alone when output_dir is the input folder. The result lists every file with its own convert_cad result or error. Send a progressToken for progress over the whole folder.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_dir` | string | yes | Absolute path of the folder with the 3D files. |
| `output_dir` | string | no | Absolute path of the folder for the outputs (created if missing). Default: input_dir. |
| `recursive` | boolean | no | Include subfolders (their structure is kept under output_dir). Default: false. |
| `format` | string | no | Output format: stl, obj, ply, gltf, glb, 3mf, dae, step, iges, fbx, vrml, off, brep. Default: the output_path extension, else the preset's format. One of: `stl`, `obj`, `ply`, `gltf`, `glb`, `3mf`, `dae`, `step`, `iges`, `fbx`, `vrml`, `off`, `brep`. |
| `quality` | string | no | Mesh quality for STEP/IGES/BREP sources: a preset (draft, standard, fine, ultrafine) or a linear deflection in mm ("0.1" = standard; lower = finer, e.g. "0.001"). Default: standard. No effect on mesh sources (STL, OBJ ...): they keep their triangles. |
| `angular` | number or string | no | Angular deflection in radians for STEP/IGES/BREP sources (0.5 = about 29 degrees; lower = smoother curves). Default: the preset's value (0.5 for standard and for a numeric quality). |
| `units` | string | no | Target units for conversion. One of: `mm`, `cm`, `in`, `m`, `ft`. |
| `up_axis` | string | no | Rotate the model so this axis points up: y (game engines, glTF, three.js) or z (CAD, 3D printing). Default: unchanged. A STEP/IGES/BREP re-export cannot be rotated and the result's notes say so. One of: `y`, `z`, `unchanged`. |
| `decimate` | number or string | no | Reduce mesh sources (STL, OBJ, PLY, glTF, FBX, 3MF ...) to this share of their triangles: 0.25 or "25%" keeps about a quarter. STEP/IGES/BREP sources are tessellated, not reduced - use quality instead; the result's notes say so. |
| `max_size_mb` | number or string | no | glTF/GLB only: the largest file to write, in MB (5, 0.5, 15; 1 MB = 1,000,000 bytes). A file over it is written again with fewer triangles until it fits, down to 5% of its triangles. The result's withinBudget is true when the written file fits and false when it was written at the smallest size reached and is still over; outputBytes, trianglesBefore and trianglesAfter are what was measured. On a glTF/GLB source with textures ktx2 or webp, the texture size limit is lowered first; with textures kept as they are, they are converted to WebP (then scaled down) before any triangle goes. |
| `compress` | boolean | no | glTF/GLB only: compress the meshes with meshopt (gltfpack); the file is often 5 to 10 times smaller. The viewer must support EXT_meshopt_compression (three.js, Babylon.js, Blender 3.0+, Unity glTFast). Default: false. |
| `assembly` | string | no | STEP assemblies: preserve (default) writes one file per top-level body of a STEP that has several (the result's additionalOutputs lists them); merge writes every body into one file. A one-root assembly is one file either way. For glTF/GLB output, merge also joins the meshes into one (node names are not kept), but only together with a web option (textures, lods, quantize, compress_fallback, keep_extras); a plain conversion or compress alone keeps the nodes. One of: `preserve`, `merge`. |
| `textures` | string | no | glTF/GLB only: keep (default), ktx2 or webp. ktx2 converts every texture to KTX2 (Basis Universal), the one format that stays compressed in GPU memory; the viewer needs a KTX2 transcoder (three.js KTX2Loader, Babylon.js). webp makes the download smaller only. The result's web.texturesConverted and the two estimatedGpuTextureBytes say what changed. One of: `keep`, `ktx2`, `webp`. |
| `texture_mode` | string | no | With textures ktx2: auto (default: ETC1S for colour textures, UASTC for normal and data maps), etc1s (smallest) or uastc (best quality, larger). One of: `auto`, `etc1s`, `uastc`. |
| `base_color` | string | no | glTF/GLB output from FBX, OBJ, DAE and other mesh sources: path of a PNG or JPEG put on every material that has texture coordinates, with a white base colour (an asset pack's shared colour atlas, which its FBX files do not link). |
| `texture_dir` | string | no | glTF/GLB output from FBX, OBJ, DAE and other mesh sources: a folder searched by file name (and for a PNG or JPEG of the same name) for a texture the source links but that is not where the link says. A link found nowhere is removed with a note. |
| `texture_quality` | integer or string | no | Texture encoding quality, a whole number from 1 (smallest) to 10 (best). Default: 8. |
| `texture_max_size` | integer or string | no | Largest texture side in pixels when textures are converted, such as 2048. Default: no limit; 0 or none removes a limit a preset sets. |
| `texture_pow2` | boolean | no | Resize converted textures to the nearest power of two (WebGL 1). Default: false. |
| `lods` | string | no | glTF/GLB only: also write LOD files, one per ratio: "0.5,0.25" or "50%,25%" writes name.lod1 and name.lod2. Each ratio is a target share of the main file's triangles; a mesh that cannot be simplified further without visible damage keeps more. With max_size_mb the LOD files are made from the already reduced main file. They are written without textures (skins and animations stay) and are listed in the result's web.lodFiles. Ratios must fall; at most four; "none" removes the LOD files a preset sets. |
| `instancing` | boolean | no | glTF/GLB only: write a mesh that is placed several times once (EXT_mesh_gpu_instancing). Node names are not kept. Not with assembly merge. Default: false. |
| `quantize` | string | no | With compress or compress_fallback: standard (default), fine (more precise positions and normals, slightly larger) or off. One of: `standard`, `fine`, `off`. |
| `compress_fallback` | boolean | no | glTF/GLB only: compress with a fallback for viewers without meshopt support (a larger file). Implies compress. Default: false. |
| `keep_extras` | boolean | no | glTF/GLB only: keep the source's extras data. Extras are dropped only when the file is compressed, textures are converted, or meshes are instanced or merged; a plain conversion keeps them. Default: false. |
| `strict` | boolean | no | Fail the call when compression, texture conversion or a LOD file could not be applied, instead of writing the file with a warning (then web.applied is false and web.problem says why). Default: false. |
| `preset` | string | no | Start from a built-in preset; the other arguments override it. web-game: GLB, meshopt compression, KTX2 textures up to 2048 px. Without format the preset's format is used (an output_path extension still wins). One of: `web-game`, `3d-printing`, `visualization`, `engineering`, `mesh-review`. |
| `repair` | boolean | no | Repair STEP/IGES/BREP surfaces before meshing (fixes broken edges and faces, closes small gaps). No effect on mesh sources such as STL or OBJ. |
| `binary` | boolean | no | Use binary output for STL/PLY (default: true). |
| `overwrite` | boolean | no | Replace an output file that already exists (default: the existing file is kept and the new one is written beside it as name (1)). |
| `skip_existing` | boolean | no | Leave the input alone when its output file already exists: nothing is converted and no free conversion is used; the result says skipped: true. |

### convert_cad

**Convert 3D/CAD file.** Convert a 3D CAD file (STEP, IGES) to mesh formats (STL, OBJ, PLY, FBX, glTF, 3MF) or between mesh formats. Mesh sources include VRML (.wrl, .vrml, gzip .wrz): VRML is read in metres (a 1 m cube is 1000 mm) and Y-up, as the specification says. The result's `notes` list what an option could not do on this route (a STEP source is tessellated, not reduced; a STEP re-export cannot be rotated). For the web and games: textures ktx2, lods and preset web-game optimise a glTF/GLB (a glTF or GLB source keeps its animations and skins); the result's `web` object says what was done. A large assembly can take minutes: send a progressToken to get progress notifications. Calling again with exactly the same arguments within 30 minutes, while the source and the output are unchanged, returns the earlier result without converting again (for a client that gave up waiting).

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_path` | string | yes | Absolute path to the source 3D file. |
| `output_path` | string | no | Absolute path for the converted output file. Give this or output_dir. |
| `output_dir` | string | no | Absolute path of a folder: the output is written there, named after the input with the format's extension (format, or a preset that names one, is then required). Give this or output_path. |
| `format` | string | no | Output format: stl, obj, ply, gltf, glb, 3mf, dae, step, iges, fbx, vrml, off, brep. Default: the output_path extension, else the preset's format. One of: `stl`, `obj`, `ply`, `gltf`, `glb`, `3mf`, `dae`, `step`, `iges`, `fbx`, `vrml`, `off`, `brep`. |
| `quality` | string | no | Mesh quality for STEP/IGES/BREP sources: a preset (draft, standard, fine, ultrafine) or a linear deflection in mm ("0.1" = standard; lower = finer, e.g. "0.001"). Default: standard. No effect on mesh sources (STL, OBJ ...): they keep their triangles. |
| `angular` | number or string | no | Angular deflection in radians for STEP/IGES/BREP sources (0.5 = about 29 degrees; lower = smoother curves). Default: the preset's value (0.5 for standard and for a numeric quality). |
| `units` | string | no | Target units for conversion. One of: `mm`, `cm`, `in`, `m`, `ft`. |
| `up_axis` | string | no | Rotate the model so this axis points up: y (game engines, glTF, three.js) or z (CAD, 3D printing). Default: unchanged. A STEP/IGES/BREP re-export cannot be rotated and the result's notes say so. One of: `y`, `z`, `unchanged`. |
| `decimate` | number or string | no | Reduce mesh sources (STL, OBJ, PLY, glTF, FBX, 3MF ...) to this share of their triangles: 0.25 or "25%" keeps about a quarter. STEP/IGES/BREP sources are tessellated, not reduced - use quality instead; the result's notes say so. |
| `max_size_mb` | number or string | no | glTF/GLB only: the largest file to write, in MB (5, 0.5, 15; 1 MB = 1,000,000 bytes). A file over it is written again with fewer triangles until it fits, down to 5% of its triangles. The result's withinBudget is true when the written file fits and false when it was written at the smallest size reached and is still over; outputBytes, trianglesBefore and trianglesAfter are what was measured. On a glTF/GLB source with textures ktx2 or webp, the texture size limit is lowered first; with textures kept as they are, they are converted to WebP (then scaled down) before any triangle goes. |
| `compress` | boolean | no | glTF/GLB only: compress the meshes with meshopt (gltfpack); the file is often 5 to 10 times smaller. The viewer must support EXT_meshopt_compression (three.js, Babylon.js, Blender 3.0+, Unity glTFast). Default: false. |
| `assembly` | string | no | STEP assemblies: preserve (default) writes one file per top-level body of a STEP that has several (the result's additionalOutputs lists them); merge writes every body into one file. A one-root assembly is one file either way. For glTF/GLB output, merge also joins the meshes into one (node names are not kept), but only together with a web option (textures, lods, quantize, compress_fallback, keep_extras); a plain conversion or compress alone keeps the nodes. One of: `preserve`, `merge`. |
| `textures` | string | no | glTF/GLB only: keep (default), ktx2 or webp. ktx2 converts every texture to KTX2 (Basis Universal), the one format that stays compressed in GPU memory; the viewer needs a KTX2 transcoder (three.js KTX2Loader, Babylon.js). webp makes the download smaller only. The result's web.texturesConverted and the two estimatedGpuTextureBytes say what changed. One of: `keep`, `ktx2`, `webp`. |
| `texture_mode` | string | no | With textures ktx2: auto (default: ETC1S for colour textures, UASTC for normal and data maps), etc1s (smallest) or uastc (best quality, larger). One of: `auto`, `etc1s`, `uastc`. |
| `base_color` | string | no | glTF/GLB output from FBX, OBJ, DAE and other mesh sources: path of a PNG or JPEG put on every material that has texture coordinates, with a white base colour (an asset pack's shared colour atlas, which its FBX files do not link). |
| `texture_dir` | string | no | glTF/GLB output from FBX, OBJ, DAE and other mesh sources: a folder searched by file name (and for a PNG or JPEG of the same name) for a texture the source links but that is not where the link says. A link found nowhere is removed with a note. |
| `texture_quality` | integer or string | no | Texture encoding quality, a whole number from 1 (smallest) to 10 (best). Default: 8. |
| `texture_max_size` | integer or string | no | Largest texture side in pixels when textures are converted, such as 2048. Default: no limit; 0 or none removes a limit a preset sets. |
| `texture_pow2` | boolean | no | Resize converted textures to the nearest power of two (WebGL 1). Default: false. |
| `lods` | string | no | glTF/GLB only: also write LOD files, one per ratio: "0.5,0.25" or "50%,25%" writes name.lod1 and name.lod2. Each ratio is a target share of the main file's triangles; a mesh that cannot be simplified further without visible damage keeps more. With max_size_mb the LOD files are made from the already reduced main file. They are written without textures (skins and animations stay) and are listed in the result's web.lodFiles. Ratios must fall; at most four; "none" removes the LOD files a preset sets. |
| `instancing` | boolean | no | glTF/GLB only: write a mesh that is placed several times once (EXT_mesh_gpu_instancing). Node names are not kept. Not with assembly merge. Default: false. |
| `quantize` | string | no | With compress or compress_fallback: standard (default), fine (more precise positions and normals, slightly larger) or off. One of: `standard`, `fine`, `off`. |
| `compress_fallback` | boolean | no | glTF/GLB only: compress with a fallback for viewers without meshopt support (a larger file). Implies compress. Default: false. |
| `keep_extras` | boolean | no | glTF/GLB only: keep the source's extras data. Extras are dropped only when the file is compressed, textures are converted, or meshes are instanced or merged; a plain conversion keeps them. Default: false. |
| `strict` | boolean | no | Fail the call when compression, texture conversion or a LOD file could not be applied, instead of writing the file with a warning (then web.applied is false and web.problem says why). Default: false. |
| `preset` | string | no | Start from a built-in preset; the other arguments override it. web-game: GLB, meshopt compression, KTX2 textures up to 2048 px. Without format the preset's format is used (an output_path extension still wins). One of: `web-game`, `3d-printing`, `visualization`, `engineering`, `mesh-review`. |
| `repair` | boolean | no | Repair STEP/IGES/BREP surfaces before meshing (fixes broken edges and faces, closes small gaps). No effect on mesh sources such as STL or OBJ. |
| `binary` | boolean | no | Use binary output for STL/PLY (default: true). |
| `overwrite` | boolean | no | Replace an output file that already exists (default: the existing file is kept and the new one is written beside it as name (1)). |
| `skip_existing` | boolean | no | Leave the input alone when its output file already exists: nothing is converted and no free conversion is used; the result says skipped: true. |

### detect_format

**Detect 3D format.** Detect the 3D file format and check if it is supported for import/export.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_path` | string | yes | Absolute path to the 3D file. |

### get_bom

**Get the part list of a STEP assembly.** Get the bill of materials of a STEP assembly: one row per unique part with its name, how many times the assembly places it (quantity), its solids and faces, its size in millimetres and its colour. STEP only: IGES and BREP files have no product structure. Reads the file; converts nothing.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_path` | string | yes | Absolute path to the STEP file (.step or .stp). |

### get_file_info

**Get 3D file info.** Get metadata about a 3D file including format, parts, triangles, vertices, and bounding box; for glTF/GLB also its textures, which extract_textures_dir writes out as PNG/JPEG files.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_path` | string | yes | Absolute path to the 3D file. |
| `extract_textures_dir` | string | no | glTF/GLB only: absolute folder to write every texture image into (WebP and KTX2 decoded to PNG); each texture then names its file. |

### get_license_status

**Get license and trial status.** Get the license and trial state before converting: registered or trial, free conversions left (freeFilesRemaining), and what exports become after them (afterFreeFiles: draft_quality = Draft tessellation and a trial note in the file). nextOutput says what the next export will be: clean or draft_quality. On a trial, buyUrl is the buy page and listPriceUsd the lifetime licence price in US dollars (a sale on the buy page can be lower); both are null when registered.

No parameters.

### list_formats

**List supported formats.** List all supported 3D file formats with import/export capabilities.

No parameters.

### register_license

**Register a license key.** Register a purchased 3D CAD Converter license on this computer: the email address the license was bought with and the license key from the purchase email. The key is checked with the ContentaSoft license server. Once registered, every output is full quality with no trial mark, here, on the command line and in the desktop app. The answer carries the license state after the attempt (the same object as get_license_status).

| Parameter | Type | Required | Description |
|---|---|---|---|
| `email` | string | yes | The email address the license was bought with. |
| `key` | string | yes | The license key from the purchase email, with or without its dashes. |

### render_model

**Render 3D model thumbnails.** Draw lit PNG thumbnails of a 3D file (any input format) from front, side, top, iso, back or bottom, to look at the model, its credits or the lettering on its textures. Writes only the PNGs; uses no trial file.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_path` | string | yes | Absolute path to the 3D file. |
| `output_path` | string | yes | Absolute PNG path; with several views each gets -\<view\> before .png; an existing folder gets \<model\>-\<view\>.png. |
| `views` | array of string | no | Views to draw (default front, side, top, iso). |
| `size` | integer | no | Width and height in pixels (default 512). Range: 64 to 4096. |
