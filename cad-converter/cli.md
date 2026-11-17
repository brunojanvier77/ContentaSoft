# 3D CAD Converter: command-line reference

`cadconvert` is the command line of 3D CAD Converter. It is installed with the desktop app, in `%LOCALAPPDATA%\Programs\CadConverter\`, and the installer adds that folder to your user `PATH`.

Generated from `cadconvert catalog`, version 1.1.0. The same reference is on the website: [www.contenta-software.com/3dcadconverter/docs/cli.html](https://www.contenta-software.com/3dcadconverter/docs/cli.html).

```powershell
cadconvert <command> [arguments] [options]
cadconvert convert part.step -o part.stl --quality fine
```

Every command has `--help`. Older spellings of some commands and options still work, so existing scripts keep running, but this reference and `--help` leave them out. The MCP server (`cadconvert serve`) is documented in [mcp-server.md](mcp-server.md).

## Commands

| Command | What it does |
|---|---|
| [`cadconvert batch <input>`](#cadconvert-batch) | Batch convert a folder of 3D files. |
| [`cadconvert catalog`](#cadconvert-catalog) | Print every CLI command and MCP tool as JSON (for AI agents, docs and the website). |
| [`cadconvert convert <input>`](#cadconvert-convert) | Convert a single 3D file. |
| [`cadconvert formats`](#cadconvert-formats) | List all supported 3D formats. |
| [`cadconvert info <file>`](#cadconvert-info) | Display information about a 3D file. |
| [`cadconvert register <email> <key>`](#cadconvert-register) | Register your license. |
| [`cadconvert serve`](#cadconvert-serve) | Start MCP server for AI agent integration. |
| [`cadconvert status`](#cadconvert-status) | Show the license and trial state: registered or trial, free conversions left (the trial has no end date), and what exports become after them (Draft quality with a trial note). |
| [`cadconvert watch <input>`](#cadconvert-watch) | Watch a folder and auto-convert new 3D files. |

## Output options

These options work the same way in the four ContentaSoft command lines. `catalog` and `serve` take none of them: `catalog` always prints JSON and `serve` keeps stdout for the MCP protocol.

| Option | Description |
|---|---|
| `--json` | Print one JSON object on stdout (success, exitCode, error, warnings, errors, outputs + the command's fields); nothing else goes to stdout. |
| `--plain` | Plain text: no colours, no escape codes, ASCII tables. Automatic when stdout is a pipe or a file, or when NO_COLOR is set. |
| `--quiet` | Print errors only. |
| `--verbose` | Verbose output: details and, on an error, the stack trace. |
| `--progress` | With --json: before the result object, one line per finished item, {"type":"progress","file":NAME,"completed":N,"total":T}. |

## Exit codes

The same table in the four ContentaSoft command lines. `--help` ends with it and `catalog` lists it.

| Code | Name | Meaning |
|---|---|---|
| 0 | `ok` | OK: everything asked for was done. |
| 1 | `unexpected_error` | Unexpected error (a bug or a broken install); see the log. |
| 2 | `invalid_arguments` | Invalid arguments: unknown option, missing or malformed value; nothing was done. |
| 3 | `input_not_found` | Input not found or unreadable; nothing was done. |
| 4 | `partial_failure` | Partial failure: some items succeeded, some failed. |
| 5 | `all_failed` | All failed: no item succeeded. |
| 6 | `license_refused` | License or trial refused the run. |
| 130 | `cancelled` | Cancelled (Ctrl+C). |

## JSON output

With `--json` a command prints one JSON object on stdout and nothing else; logs go to stderr. Every object has these keys, beside the fields of the command itself:

| Key | Meaning |
|---|---|
| `success` | `true` when the run did everything it was asked to do |
| `exitCode` | the process exit code (see Exit codes) |
| `error` | one sentence when the run failed, otherwise `null` |
| `warnings` | notes about the run that did not stop it |
| `errors` | one entry per item that failed (a batch keeps going) |
| `outputs` | the files the run wrote |

Trial keys, present only when they apply:

| Key | Meaning |
|---|---|
| `trialWatermarked` | the output carries the trial watermark |
| `trialExport` | the output is a trial export |
| `trialNotice` | one sentence about the trial, for the user |
| `freeFilesRemaining` | free full-quality files left on this computer |
| `buyUrl` | where to buy a licence |

With `--json --progress` a long command first prints one progress line per finished item (NDJSON), then the result object last. A script that wants only the result skips the lines that start with `{"type":"progress"`. Progress keys:

| Key | Meaning |
|---|---|
| `type` | always `"progress"` |
| `file` | the file that finished |
| `completed` | items finished so far |
| `total` | items in the run |

## Command reference

### cadconvert batch

Batch convert a folder of 3D files.

```
cadconvert batch <input> [options]
```

| Argument | Description |
|---|---|
| `<input>` | Input folder. |

| Option | Type | Description |
|---|---|---|
| `--format` | string | Target format. |
| `--report` | string | Write a CSV report of the batch to this file: one row per input with input and output file, formats, triangles before and after, output size, size budget, within budget yes/no, duration and result or error. |
| `--output` | string | Output folder. Default: \<input\>\_converted next to the input folder. |
| `--include` | string[] | Only files matching this pattern, e.g. \*.step or parts\\\*.stp; repeat the option or separate patterns with ';' or ','. A pattern with \* or ? is matched against the file name and the path below the folder; other text is part of that path. |
| `--exclude` | string[] | Leave out files matching this pattern, e.g. \*\_old.step or archive; repeat the option or separate patterns with ';' or ','. Same matching as --include. |
| `--recursive` | boolean | Include subfolders (the default; -r false converts the folder itself only). |
| `--workers` | integer | Parallel workers (default: the number of processors). |
| `--quality` | string | Mesh quality for STEP/IGES/BREP sources: a preset (draft, standard, fine, ultrafine) or a linear deflection in mm (0.1 = standard; lower = finer). Default: standard. |
| `--angular` | string | Angular deflection in radians (0.5 = about 29 degrees; lower = smoother curves). Default: the preset's value (0.5 for standard and for a numeric --quality). |
| `--repair` | boolean | Repair STEP/IGES/BREP surfaces before meshing: fixes broken edges and faces and closes small gaps. No effect on mesh sources (STL, OBJ ...) |
| `--units` | string | Target units (mm, cm, in, m, ft). |
| `--binary` | boolean | Binary output for STL/PLY. |
| `--up-axis` | string | Rotate the model so this axis points up: y (game engines, glTF, three.js) or z (CAD, 3D printing). Default: unchanged. A STEP/IGES/BREP re-export cannot be rotated and says so. |
| `--decimate` | string | Reduce mesh sources (STL, OBJ, PLY, glTF, FBX, 3MF ...) to this share of their triangles: 0.25 or 25% keeps about a quarter. STEP/IGES/BREP are tessellated, not reduced - use --quality. |
| `--max-size` | string | glTF/GLB only: the largest file to write, in MB (5, 0.5 or 15MB; 1 MB = 1,000,000 bytes). A file over it is written again with fewer triangles until it fits, down to 5% of its triangles. One that still does not fit is written at the smallest size reached, reported as over budget, and the exit code is 4 (partial). On a glTF/GLB source with --textures, the texture size limit is lowered first. |
| `--compress` | boolean | glTF/GLB only: compress the meshes with meshopt (gltfpack); the file is often 5 to 10 times smaller. The viewer must support EXT_meshopt_compression (three.js, Babylon.js, Blender 3.0+, Unity glTFast). |
| `--assembly` | string | STEP assemblies: preserve (default) writes one file per top-level body of a STEP that has several; merge writes every body into one file. A one-root assembly is one file either way. For glTF/GLB output, merge also joins the meshes into one (node names are not kept), but only together with a web option (--textures, --lods, --quantize, --compress-fallback, --keep-extras); a plain conversion or --compress alone keeps the nodes. |
| `--textures` | string | glTF/GLB only: keep (default), ktx2 or webp. ktx2 converts every texture to KTX2 (Basis Universal), the one format that stays compressed in GPU memory; the viewer needs a KTX2 transcoder (three.js KTX2Loader, Babylon.js). webp makes the download smaller only. |
| `--texture-mode` | string | With --textures ktx2: auto (default: ETC1S for colour textures, UASTC for normal and data maps), etc1s (smallest) or uastc (best quality, larger). |
| `--texture-quality` | string | Texture encoding quality, 1 (smallest) to 10 (best). Default: 8. |
| `--texture-max-size` | string | Largest texture side in pixels when textures are converted, such as 2048. Default: no limit; 0 or none removes a limit a preset sets. |
| `--texture-pow2` | boolean | Resize converted textures to the nearest power of two (WebGL 1). With --texture-max-size the result can be one step above the limit. |
| `--lods` | string | glTF/GLB only: also write LOD files, one per ratio: 0.5,0.25 (or 0.5 0.25, or 50%,25%) writes name.lod1 and name.lod2. Each ratio is a target share of the main file's triangles; a mesh that cannot be simplified further without visible damage keeps more: the result says how many triangles each LOD file kept (lods in --json). With --max-size the LOD files are made from the already reduced main file. LOD files are written without textures (skins and animations stay). Ratios must fall; at most four; none removes the LOD files a preset sets. |
| `--instancing` | boolean | glTF/GLB only: write a mesh that is placed several times once (EXT_mesh_gpu_instancing). Node names are not kept. |
| `--quantize` | string | With --compress or --compress-fallback: standard (default), fine (more precise positions and normals, slightly larger) or off. A quantized mesh moves into a new unnamed child of its node, which carries the scale that undoes the quantization: node names stay, but code that expects the mesh directly on the named node must look one level down. off keeps the structure. |
| `--compress-fallback` | boolean | glTF/GLB only: compress with a fallback for viewers without meshopt support (a larger file). Implies --compress. |
| `--keep-extras` | boolean | glTF/GLB only: keep the source's extras data. Extras are dropped only when the file is compressed, textures are converted, or meshes are instanced or merged; a plain conversion keeps them. |
| `--strict` | boolean | Fail a file when compression, texture conversion or a LOD file could not be applied, instead of writing it with a warning (exit code 5; 4 when other files of a batch succeeded). For build scripts. |
| `--profile` | string | Start from a saved conversion profile (JSON, as the app saves it in %APPDATA%\\CadConverter\\profiles). Options given on the command line override the profile's values. |
| `--preset` | string | Start from a built-in preset: web-game (GLB, meshopt compression, KTX2 textures up to 2048 px), 3d-printing, visualization, engineering or mesh-review. Options given on the command line override the preset's values; --profile wins when both are given. |
| `--overwrite` | boolean | Replace an output file that already exists. |
| `--skip-existing` | boolean | Leave an input alone when its output file already exists, so a re-run only converts what is missing (a skipped file is not converted and costs no free conversion). |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### cadconvert catalog

Print every CLI command and MCP tool as JSON (for AI agents, docs and the website).

```
cadconvert catalog
```

### cadconvert convert

Convert a single 3D file.

```
cadconvert convert <input> [options]
```

| Argument | Description |
|---|---|
| `<input>` | Input 3D file. |

| Option | Type | Description |
|---|---|---|
| `--format` | string | Target format (stl, obj, step, iges, etc.) |
| `--output` | string | Output file or folder. Default: next to the input, with the -f format's extension. |
| `--quality` | string | Mesh quality for STEP/IGES/BREP sources: a preset (draft, standard, fine, ultrafine) or a linear deflection in mm (0.1 = standard; lower = finer). Default: standard. |
| `--angular` | string | Angular deflection in radians (0.5 = about 29 degrees; lower = smoother curves). Default: the preset's value (0.5 for standard and for a numeric --quality). |
| `--repair` | boolean | Repair STEP/IGES/BREP surfaces before meshing: fixes broken edges and faces and closes small gaps. No effect on mesh sources (STL, OBJ ...) |
| `--units` | string | Target units (mm, cm, in, m, ft). |
| `--binary` | boolean | Binary output for STL/PLY. |
| `--up-axis` | string | Rotate the model so this axis points up: y (game engines, glTF, three.js) or z (CAD, 3D printing). Default: unchanged. A STEP/IGES/BREP re-export cannot be rotated and says so. |
| `--decimate` | string | Reduce mesh sources (STL, OBJ, PLY, glTF, FBX, 3MF ...) to this share of their triangles: 0.25 or 25% keeps about a quarter. STEP/IGES/BREP are tessellated, not reduced - use --quality. |
| `--max-size` | string | glTF/GLB only: the largest file to write, in MB (5, 0.5 or 15MB; 1 MB = 1,000,000 bytes). A file over it is written again with fewer triangles until it fits, down to 5% of its triangles. One that still does not fit is written at the smallest size reached, reported as over budget, and the exit code is 4 (partial). On a glTF/GLB source with --textures, the texture size limit is lowered first. |
| `--compress` | boolean | glTF/GLB only: compress the meshes with meshopt (gltfpack); the file is often 5 to 10 times smaller. The viewer must support EXT_meshopt_compression (three.js, Babylon.js, Blender 3.0+, Unity glTFast). |
| `--assembly` | string | STEP assemblies: preserve (default) writes one file per top-level body of a STEP that has several; merge writes every body into one file. A one-root assembly is one file either way. For glTF/GLB output, merge also joins the meshes into one (node names are not kept), but only together with a web option (--textures, --lods, --quantize, --compress-fallback, --keep-extras); a plain conversion or --compress alone keeps the nodes. |
| `--textures` | string | glTF/GLB only: keep (default), ktx2 or webp. ktx2 converts every texture to KTX2 (Basis Universal), the one format that stays compressed in GPU memory; the viewer needs a KTX2 transcoder (three.js KTX2Loader, Babylon.js). webp makes the download smaller only. |
| `--texture-mode` | string | With --textures ktx2: auto (default: ETC1S for colour textures, UASTC for normal and data maps), etc1s (smallest) or uastc (best quality, larger). |
| `--texture-quality` | string | Texture encoding quality, 1 (smallest) to 10 (best). Default: 8. |
| `--texture-max-size` | string | Largest texture side in pixels when textures are converted, such as 2048. Default: no limit; 0 or none removes a limit a preset sets. |
| `--texture-pow2` | boolean | Resize converted textures to the nearest power of two (WebGL 1). With --texture-max-size the result can be one step above the limit. |
| `--lods` | string | glTF/GLB only: also write LOD files, one per ratio: 0.5,0.25 (or 0.5 0.25, or 50%,25%) writes name.lod1 and name.lod2. Each ratio is a target share of the main file's triangles; a mesh that cannot be simplified further without visible damage keeps more: the result says how many triangles each LOD file kept (lods in --json). With --max-size the LOD files are made from the already reduced main file. LOD files are written without textures (skins and animations stay). Ratios must fall; at most four; none removes the LOD files a preset sets. |
| `--instancing` | boolean | glTF/GLB only: write a mesh that is placed several times once (EXT_mesh_gpu_instancing). Node names are not kept. |
| `--quantize` | string | With --compress or --compress-fallback: standard (default), fine (more precise positions and normals, slightly larger) or off. A quantized mesh moves into a new unnamed child of its node, which carries the scale that undoes the quantization: node names stay, but code that expects the mesh directly on the named node must look one level down. off keeps the structure. |
| `--compress-fallback` | boolean | glTF/GLB only: compress with a fallback for viewers without meshopt support (a larger file). Implies --compress. |
| `--keep-extras` | boolean | glTF/GLB only: keep the source's extras data. Extras are dropped only when the file is compressed, textures are converted, or meshes are instanced or merged; a plain conversion keeps them. |
| `--strict` | boolean | Fail a file when compression, texture conversion or a LOD file could not be applied, instead of writing it with a warning (exit code 5; 4 when other files of a batch succeeded). For build scripts. |
| `--profile` | string | Start from a saved conversion profile (JSON, as the app saves it in %APPDATA%\\CadConverter\\profiles). Options given on the command line override the profile's values. |
| `--preset` | string | Start from a built-in preset: web-game (GLB, meshopt compression, KTX2 textures up to 2048 px), 3d-printing, visualization, engineering or mesh-review. Options given on the command line override the preset's values; --profile wins when both are given. |
| `--overwrite` | boolean | Replace an output file that already exists. |
| `--skip-existing` | boolean | Leave an input alone when its output file already exists, so a re-run only converts what is missing (a skipped file is not converted and costs no free conversion). |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### cadconvert formats

List all supported 3D formats.

```
cadconvert formats [options]
```

| Option | Type | Description |
|---|---|---|
| `--input` | boolean | Only the formats it reads (import). |
| `--output` | boolean | Only the formats it writes (export). |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### cadconvert info

Display information about a 3D file.

```
cadconvert info <file> [options]
```

| Argument | Description |
|---|---|
| `<file>` | 3D file to inspect. |

| Option | Type | Description |
|---|---|---|
| `--bom` | boolean | STEP only: also list the parts (name, quantity, solids, faces, size in mm), one row per unique part. |
| `--bom-csv` | string | STEP only: write the part list as CSV to this file (the same columns as the app's BOM export). |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### cadconvert register

Register your license.

```
cadconvert register <email> <key>
```

| Argument | Description |
|---|---|
| `<email>` | The email address the license was bought with. |
| `<key>` | License key. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### cadconvert serve

Start MCP server for AI agent integration.

```
cadconvert serve
```

### cadconvert status

Show the license and trial state: registered or trial, free conversions left (the trial has no end date), and what exports become after them (Draft quality with a trial note).

```
cadconvert status
```

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### cadconvert watch

Watch a folder and auto-convert new 3D files.

```
cadconvert watch <input> [options]
```

| Argument | Description |
|---|---|
| `<input>` | Folder to watch. |

| Option | Type | Description |
|---|---|---|
| `--format` | string | Target format. |
| `--report` | string | Keep a CSV report of this watch session in this file, rewritten after every file: one row per input with input and output file, formats, triangles before and after, output size, size budget, within budget yes/no, duration and result or error. |
| `--output` | string | Output folder. Default: \<input\>\_converted next to the watched folder. |
| `--include` | string[] | Only files matching this pattern, e.g. \*.step or parts\\\*.stp; repeat the option or separate patterns with ';' or ','. A pattern with \* or ? is matched against the file name and the path below the folder; other text is part of that path. |
| `--exclude` | string[] | Leave out files matching this pattern, e.g. \*\_old.step or archive; repeat the option or separate patterns with ';' or ','. Same matching as --include. |
| `--recursive` | boolean | Include subfolders (the default; -r false converts the folder itself only). |
| `--workers` | integer | Parallel workers (default: the number of processors). |
| `--quality` | string | Mesh quality for STEP/IGES/BREP sources: a preset (draft, standard, fine, ultrafine) or a linear deflection in mm (0.1 = standard; lower = finer). Default: standard. |
| `--angular` | string | Angular deflection in radians (0.5 = about 29 degrees; lower = smoother curves). Default: the preset's value (0.5 for standard and for a numeric --quality). |
| `--repair` | boolean | Repair STEP/IGES/BREP surfaces before meshing: fixes broken edges and faces and closes small gaps. No effect on mesh sources (STL, OBJ ...) |
| `--units` | string | Target units (mm, cm, in, m, ft). |
| `--binary` | boolean | Binary output for STL/PLY. |
| `--up-axis` | string | Rotate the model so this axis points up: y (game engines, glTF, three.js) or z (CAD, 3D printing). Default: unchanged. A STEP/IGES/BREP re-export cannot be rotated and says so. |
| `--decimate` | string | Reduce mesh sources (STL, OBJ, PLY, glTF, FBX, 3MF ...) to this share of their triangles: 0.25 or 25% keeps about a quarter. STEP/IGES/BREP are tessellated, not reduced - use --quality. |
| `--max-size` | string | glTF/GLB only: the largest file to write, in MB (5, 0.5 or 15MB; 1 MB = 1,000,000 bytes). A file over it is written again with fewer triangles until it fits, down to 5% of its triangles. One that still does not fit is written at the smallest size reached, reported as over budget, and the exit code is 4 (partial). On a glTF/GLB source with --textures, the texture size limit is lowered first. |
| `--compress` | boolean | glTF/GLB only: compress the meshes with meshopt (gltfpack); the file is often 5 to 10 times smaller. The viewer must support EXT_meshopt_compression (three.js, Babylon.js, Blender 3.0+, Unity glTFast). |
| `--assembly` | string | STEP assemblies: preserve (default) writes one file per top-level body of a STEP that has several; merge writes every body into one file. A one-root assembly is one file either way. For glTF/GLB output, merge also joins the meshes into one (node names are not kept), but only together with a web option (--textures, --lods, --quantize, --compress-fallback, --keep-extras); a plain conversion or --compress alone keeps the nodes. |
| `--textures` | string | glTF/GLB only: keep (default), ktx2 or webp. ktx2 converts every texture to KTX2 (Basis Universal), the one format that stays compressed in GPU memory; the viewer needs a KTX2 transcoder (three.js KTX2Loader, Babylon.js). webp makes the download smaller only. |
| `--texture-mode` | string | With --textures ktx2: auto (default: ETC1S for colour textures, UASTC for normal and data maps), etc1s (smallest) or uastc (best quality, larger). |
| `--texture-quality` | string | Texture encoding quality, 1 (smallest) to 10 (best). Default: 8. |
| `--texture-max-size` | string | Largest texture side in pixels when textures are converted, such as 2048. Default: no limit; 0 or none removes a limit a preset sets. |
| `--texture-pow2` | boolean | Resize converted textures to the nearest power of two (WebGL 1). With --texture-max-size the result can be one step above the limit. |
| `--lods` | string | glTF/GLB only: also write LOD files, one per ratio: 0.5,0.25 (or 0.5 0.25, or 50%,25%) writes name.lod1 and name.lod2. Each ratio is a target share of the main file's triangles; a mesh that cannot be simplified further without visible damage keeps more: the result says how many triangles each LOD file kept (lods in --json). With --max-size the LOD files are made from the already reduced main file. LOD files are written without textures (skins and animations stay). Ratios must fall; at most four; none removes the LOD files a preset sets. |
| `--instancing` | boolean | glTF/GLB only: write a mesh that is placed several times once (EXT_mesh_gpu_instancing). Node names are not kept. |
| `--quantize` | string | With --compress or --compress-fallback: standard (default), fine (more precise positions and normals, slightly larger) or off. A quantized mesh moves into a new unnamed child of its node, which carries the scale that undoes the quantization: node names stay, but code that expects the mesh directly on the named node must look one level down. off keeps the structure. |
| `--compress-fallback` | boolean | glTF/GLB only: compress with a fallback for viewers without meshopt support (a larger file). Implies --compress. |
| `--keep-extras` | boolean | glTF/GLB only: keep the source's extras data. Extras are dropped only when the file is compressed, textures are converted, or meshes are instanced or merged; a plain conversion keeps them. |
| `--strict` | boolean | Fail a file when compression, texture conversion or a LOD file could not be applied, instead of writing it with a warning (exit code 5; 4 when other files of a batch succeeded). For build scripts. |
| `--profile` | string | Start from a saved conversion profile (JSON, as the app saves it in %APPDATA%\\CadConverter\\profiles). Options given on the command line override the profile's values. |
| `--preset` | string | Start from a built-in preset: web-game (GLB, meshopt compression, KTX2 textures up to 2048 px), 3d-printing, visualization, engineering or mesh-review. Options given on the command line override the preset's values; --profile wins when both are given. |
| `--overwrite` | boolean | Replace an output file that already exists. |
| `--skip-existing` | boolean | Leave an input alone when its output file already exists, so a re-run only converts what is missing (a skipped file is not converted and costs no free conversion). |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.
