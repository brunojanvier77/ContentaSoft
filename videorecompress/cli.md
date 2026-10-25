# VideoRecompress Studio: command-line reference

`videorecompress` is the command line of VideoRecompress Studio. It is installed with the desktop app, in `%LOCALAPPDATA%\Programs\VideoRecompressStudio\`, and the installer adds that folder to your user `PATH`.

Generated from `videorecompress catalog`, version 2026.2.28. The same reference is on the website: [www.contenta-software.com/videorecompress/docs/cli.html](https://www.contenta-software.com/videorecompress/docs/cli.html).

```powershell
videorecompress <command> [arguments] [options]
videorecompress recompress video.mp4 -o compressed --preset phone_archive
```

Every command has `--help`. Older spellings of some commands and options still work, so existing scripts keep running, but this reference and `--help` leave them out. The MCP server (`videorecompress serve`) is documented in [mcp-server.md](mcp-server.md).

## Commands

| Command | What it does |
|---|---|
| [`videorecompress batch <input-dir>`](#videorecompress-batch) | Batch recompress all videos in a directory. |
| [`videorecompress catalog`](#videorecompress-catalog) | Print every CLI command and MCP tool as JSON (for AI agents, docs and the website). |
| [`videorecompress formats`](#videorecompress-formats) | List the containers and codecs it reads and writes. |
| [`videorecompress info <file>`](#videorecompress-info) | Show video file information (codec, resolution, duration, HDR, audio, subtitles). |
| [`videorecompress presets`](#videorecompress-presets) | List available recompression presets. |
| [`videorecompress profile`](#videorecompress-profile) | Save, load, or validate recompression profiles. |
| [`videorecompress profile load <file>`](#videorecompress-profile-load) | Load and display a profile from JSON. |
| [`videorecompress profile save <file>`](#videorecompress-profile-save) | Build a profile from options and save to JSON. |
| [`videorecompress profile validate <file>`](#videorecompress-profile-validate) | Validate a profile JSON file. |
| [`videorecompress recompress <input>`](#videorecompress-recompress) | Recompress a single video file. |
| [`videorecompress register <email> <key>`](#videorecompress-register) | Register your license key. |
| [`videorecompress serve`](#videorecompress-serve) | Start MCP server for AI agent integration. |
| [`videorecompress status`](#videorecompress-status) | Show tool availability, license state, and GPU info. |
| [`videorecompress watch <dir>`](#videorecompress-watch) | Monitor a folder and auto-recompress new video files. |

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

### videorecompress batch

Batch recompress all videos in a directory.

```
videorecompress batch <input-dir> [options]
```

| Argument | Description |
|---|---|
| `<input-dir>` | Path to the directory containing video files. |

| Option | Type | Description |
|---|---|---|
| `--include` | string[] | Only videos matching this pattern (e.g. "\*.mp4"); repeat the option or separate patterns with ';' or ','. |
| `--exclude` | string[] | Leave out videos matching this pattern (same forms as --include). |
| `--recursive` | boolean | Include subdirectories. |
| `--workers` | integer | Number of parallel workers. |
| `--report` | string | Write a CSV report, one row per input file (file, status, output, outputCodec, trialWatermarked, error). |
| `--output` | string | Output directory. |
| `--codec` | string | Video codec: h264, h265, av1, vp9, copy. |
| `--crf` | integer | Constant Rate Factor (0-63). |
| `--quality-mode` | string | Quality mode: crf, cbr, twopass. |
| `--bitrate` | integer | Target bitrate in kbps (for cbr/twopass). |
| `--encoder-preset` | string | Encoder speed preset (ultrafast..veryslow, or an SVT-AV1 number 0-13; mapped to what each encoder accepts). |
| `--preset` | string | Named recompression preset to use as base settings (e.g. phone_archive — run 'videorecompress presets' for the list). Explicit flags override preset values. |
| `--hw-accel` | string | Hardware acceleration: auto, nvenc, qsv, amf, software. |
| `--audio-mode` | string | Audio mode: copy, reencode, remove. |
| `--audio-codec` | string | Audio codec: aac, opus, mp3, flac, ac3, eac3. |
| `--audio-bitrate` | integer | Audio bitrate in kbps. |
| `--container` | string | Container format: mp4, mkv, webm. |
| `--width` | integer | Target width (0 = keep original). |
| `--height` | integer | Target height (0 = keep original). |
| `--denoise` | string | Denoise: off, light, medium, strong. |
| `--deinterlace` | string | Deinterlace: off, yadif, yadif-bob. |
| `--watermark-text` | string | Apply a text watermark. |
| `--watermark-image` | string | Apply an image watermark (PNG/JPG path). |
| `--watermark-position` | string | Watermark position: TopLeft, TopCenter, TopRight, CenterLeft, Center, CenterRight, BottomLeft, BottomCenter, BottomRight. |
| `--watermark-opacity` | integer | Watermark opacity 0-100. |
| `--watermark-scale` | integer | Watermark size as % of video width (5-100). |
| `--trim-mode` | string | Trim mode: none, skip-start, skip-end, keep-first, custom. |
| `--trim-start` | number | Trim start time in seconds (for custom range). |
| `--trim-end` | number | Trim end time in seconds (for custom range). |
| `--trim-duration` | number | Trim duration in seconds (for skip-start, skip-end, keep-first). |
| `--fps` | number | Target frame rate. |
| `--fps-mode` | string | Frame rate mode: auto, cfr, vfr. |
| `--hdr` | string | HDR sources (HDR10, HLG): keep (stay 10-bit HDR where the codec can hold it; H.264 is always converted) or sdr (tone-map to SDR BT.709). SDR sources are not affected. |
| `--overwrite` | boolean | Overwrite existing output files. |
| `--skip-existing` | boolean | Skip a file whose output already exists, so a re-run only does what is missing. |
| `--profile` | string | Load settings from a JSON profile file. |
| `--measure-quality` | boolean | Measure SSIM/VMAF quality after encoding. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### videorecompress catalog

Print every CLI command and MCP tool as JSON (for AI agents, docs and the website).

```
videorecompress catalog
```

### videorecompress formats

List the containers and codecs it reads and writes.

```
videorecompress formats [options]
```

| Option | Type | Description |
|---|---|---|
| `--input` | boolean | Show what it reads only. |
| `--output` | boolean | Show what it writes only. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### videorecompress info

Show video file information (codec, resolution, duration, HDR, audio, subtitles).

```
videorecompress info <file>
```

| Argument | Description |
|---|---|
| `<file>` | Path to the video file. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### videorecompress presets

List available recompression presets.

```
videorecompress presets [options]
```

| Option | Type | Description |
|---|---|---|
| `--preset` | string | Show full details for a specific preset by ID. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### videorecompress profile

Save, load, or validate recompression profiles.

Subcommands: `videorecompress profile load`, `videorecompress profile save`, `videorecompress profile validate`.

### videorecompress profile load

Load and display a profile from JSON.

```
videorecompress profile load <file>
```

| Argument | Description |
|---|---|
| `<file>` | Path to the profile JSON file. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### videorecompress profile save

Build a profile from options and save to JSON.

```
videorecompress profile save <file> [options]
```

| Argument | Description |
|---|---|
| `<file>` | Path to save the profile JSON file. |

| Option | Type | Description |
|---|---|---|
| `--output` | string | Output directory. |
| `--codec` | string | Video codec: h264, h265, av1, vp9, copy. |
| `--crf` | integer | Constant Rate Factor (0-63). |
| `--quality-mode` | string | Quality mode: crf, cbr, twopass. |
| `--bitrate` | integer | Target bitrate in kbps (for cbr/twopass). |
| `--encoder-preset` | string | Encoder speed preset (ultrafast..veryslow, or an SVT-AV1 number 0-13; mapped to what each encoder accepts). |
| `--preset` | string | Named recompression preset to use as base settings (e.g. phone_archive — run 'videorecompress presets' for the list). Explicit flags override preset values. |
| `--hw-accel` | string | Hardware acceleration: auto, nvenc, qsv, amf, software. |
| `--audio-mode` | string | Audio mode: copy, reencode, remove. |
| `--audio-codec` | string | Audio codec: aac, opus, mp3, flac, ac3, eac3. |
| `--audio-bitrate` | integer | Audio bitrate in kbps. |
| `--container` | string | Container format: mp4, mkv, webm. |
| `--width` | integer | Target width (0 = keep original). |
| `--height` | integer | Target height (0 = keep original). |
| `--denoise` | string | Denoise: off, light, medium, strong. |
| `--deinterlace` | string | Deinterlace: off, yadif, yadif-bob. |
| `--watermark-text` | string | Apply a text watermark. |
| `--watermark-image` | string | Apply an image watermark (PNG/JPG path). |
| `--watermark-position` | string | Watermark position: TopLeft, TopCenter, TopRight, CenterLeft, Center, CenterRight, BottomLeft, BottomCenter, BottomRight. |
| `--watermark-opacity` | integer | Watermark opacity 0-100. |
| `--watermark-scale` | integer | Watermark size as % of video width (5-100). |
| `--trim-mode` | string | Trim mode: none, skip-start, skip-end, keep-first, custom. |
| `--trim-start` | number | Trim start time in seconds (for custom range). |
| `--trim-end` | number | Trim end time in seconds (for custom range). |
| `--trim-duration` | number | Trim duration in seconds (for skip-start, skip-end, keep-first). |
| `--fps` | number | Target frame rate. |
| `--fps-mode` | string | Frame rate mode: auto, cfr, vfr. |
| `--hdr` | string | HDR sources (HDR10, HLG): keep (stay 10-bit HDR where the codec can hold it; H.264 is always converted) or sdr (tone-map to SDR BT.709). SDR sources are not affected. |
| `--overwrite` | boolean | Overwrite existing output files. |
| `--skip-existing` | boolean | Skip a file whose output already exists, so a re-run only does what is missing. |
| `--profile` | string | Load settings from a JSON profile file. |
| `--measure-quality` | boolean | Measure SSIM/VMAF quality after encoding. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### videorecompress profile validate

Validate a profile JSON file.

```
videorecompress profile validate <file>
```

| Argument | Description |
|---|---|
| `<file>` | Path to the profile JSON file to validate. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### videorecompress recompress

Recompress a single video file.

```
videorecompress recompress <input> [options]
```

| Argument | Description |
|---|---|
| `<input>` | Path to the video file. |

| Option | Type | Description |
|---|---|---|
| `--output` | string | Output file or folder. A folder: an existing one, a path ending in \\ or /, or a name without an extension (the output is named after the input); anything else is the output file; a file's extension picks the container (.mp4 .mkv .webm .mov .ts). Default: the input's folder. |
| `--codec` | string | Video codec: h264, h265, av1, vp9, copy. |
| `--crf` | integer | Constant Rate Factor (0-63). |
| `--quality-mode` | string | Quality mode: crf, cbr, twopass. |
| `--bitrate` | integer | Target bitrate in kbps (for cbr/twopass). |
| `--encoder-preset` | string | Encoder speed preset (ultrafast..veryslow, or an SVT-AV1 number 0-13; mapped to what each encoder accepts). |
| `--preset` | string | Named recompression preset to use as base settings (e.g. phone_archive — run 'videorecompress presets' for the list). Explicit flags override preset values. |
| `--hw-accel` | string | Hardware acceleration: auto, nvenc, qsv, amf, software. |
| `--audio-mode` | string | Audio mode: copy, reencode, remove. |
| `--audio-codec` | string | Audio codec: aac, opus, mp3, flac, ac3, eac3. |
| `--audio-bitrate` | integer | Audio bitrate in kbps. |
| `--container` | string | Container format: mp4, mkv, webm. |
| `--width` | integer | Target width (0 = keep original). |
| `--height` | integer | Target height (0 = keep original). |
| `--denoise` | string | Denoise: off, light, medium, strong. |
| `--deinterlace` | string | Deinterlace: off, yadif, yadif-bob. |
| `--watermark-text` | string | Apply a text watermark. |
| `--watermark-image` | string | Apply an image watermark (PNG/JPG path). |
| `--watermark-position` | string | Watermark position: TopLeft, TopCenter, TopRight, CenterLeft, Center, CenterRight, BottomLeft, BottomCenter, BottomRight. |
| `--watermark-opacity` | integer | Watermark opacity 0-100. |
| `--watermark-scale` | integer | Watermark size as % of video width (5-100). |
| `--trim-mode` | string | Trim mode: none, skip-start, skip-end, keep-first, custom. |
| `--trim-start` | number | Trim start time in seconds (for custom range). |
| `--trim-end` | number | Trim end time in seconds (for custom range). |
| `--trim-duration` | number | Trim duration in seconds (for skip-start, skip-end, keep-first). |
| `--fps` | number | Target frame rate. |
| `--fps-mode` | string | Frame rate mode: auto, cfr, vfr. |
| `--hdr` | string | HDR sources (HDR10, HLG): keep (stay 10-bit HDR where the codec can hold it; H.264 is always converted) or sdr (tone-map to SDR BT.709). SDR sources are not affected. |
| `--overwrite` | boolean | Overwrite existing output files. |
| `--skip-existing` | boolean | Skip a file whose output already exists, so a re-run only does what is missing. |
| `--profile` | string | Load settings from a JSON profile file. |
| `--measure-quality` | boolean | Measure SSIM/VMAF quality after encoding. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### videorecompress register

Register your license key.

```
videorecompress register <email> <key>
```

| Argument | Description |
|---|---|
| `<email>` | Registration email. |
| `<key>` | License key. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### videorecompress serve

Start MCP server for AI agent integration.

```
videorecompress serve
```

### videorecompress status

Show tool availability, license state, and GPU info.

```
videorecompress status
```

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### videorecompress watch

Monitor a folder and auto-recompress new video files.

```
videorecompress watch <dir> [options]
```

| Argument | Description |
|---|---|
| `<dir>` | Path to the folder to watch. |

| Option | Type | Description |
|---|---|---|
| `--preset` | string | Preset ID to use (e.g. security_archive, phone_archive, max_savings; default: phone_archive). |
| `--output` | string | Output directory for recompressed files (default: same as source). |
| `--recursive` | boolean | Include subfolders. |
| `--include` | string[] | Only videos matching this pattern (e.g. "\*.mp4"); repeat the option or separate patterns with ';' or ','. |
| `--exclude` | string[] | Leave out videos matching this pattern (same forms as --include). |
| `--watermark-text` | string | Apply a text watermark to all videos. |
| `--watermark-image` | string | Apply an image/logo watermark from a file (PNG, JPG). |
| `--watermark-position` | string | Watermark position: TopLeft, TopCenter, TopRight, CenterLeft, Center, CenterRight, BottomLeft, BottomCenter, BottomRight. |
| `--watermark-opacity` | integer | Watermark opacity 0-100. |
| `--watermark-scale` | integer | Watermark size as % of video width (5-100). |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.
