# 3D CAD Converter: command-line reference

`cadconvert` is the command line of 3D CAD Converter. It is installed with the desktop app, in `%LOCALAPPDATA%\Programs\CadConverter\`, and the installer adds that folder to your user `PATH`.

Generated from `cadconvert catalog`, version 1.0.34. The same reference is on the website: [www.contenta-software.com/3dcadconverter/docs/cli.html](https://www.contenta-software.com/3dcadconverter/docs/cli.html).

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
| [`cadconvert status`](#cadconvert-status) | Show the license and trial state: registered or trial, free conversions left, trial days left, and what exports become after the trial (Draft quality with a trial note). |
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
| `--max-size` | string | glTF/GLB only: the largest file to write, in MB (5, 0.5 or 15MB; 1 MB = 1,000,000 bytes). A file over it is written again with fewer triangles until it fits, down to 5% of its triangles. One that still does not fit is written at the smallest size reached, reported as over budget, and the exit code is 4 (partial). |
| `--profile` | string | Start from a saved conversion profile (JSON, as the app saves it in %APPDATA%\\CadConverter\\profiles). Options given on the command line override the profile's values. |
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
| `--max-size` | string | glTF/GLB only: the largest file to write, in MB (5, 0.5 or 15MB; 1 MB = 1,000,000 bytes). A file over it is written again with fewer triangles until it fits, down to 5% of its triangles. One that still does not fit is written at the smallest size reached, reported as over budget, and the exit code is 4 (partial). |
| `--profile` | string | Start from a saved conversion profile (JSON, as the app saves it in %APPDATA%\\CadConverter\\profiles). Options given on the command line override the profile's values. |
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
cadconvert info <file>
```

| Argument | Description |
|---|---|
| `<file>` | 3D file to inspect. |

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

Show the license and trial state: registered or trial, free conversions left, trial days left, and what exports become after the trial (Draft quality with a trial note).

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
| `--max-size` | string | glTF/GLB only: the largest file to write, in MB (5, 0.5 or 15MB; 1 MB = 1,000,000 bytes). A file over it is written again with fewer triangles until it fits, down to 5% of its triangles. One that still does not fit is written at the smallest size reached, reported as over budget, and the exit code is 4 (partial). |
| `--profile` | string | Start from a saved conversion profile (JSON, as the app saves it in %APPDATA%\\CadConverter\\profiles). Options given on the command line override the profile's values. |
| `--overwrite` | boolean | Replace an output file that already exists. |
| `--skip-existing` | boolean | Leave an input alone when its output file already exists, so a re-run only converts what is missing (a skipped file is not converted and costs no free conversion). |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.
