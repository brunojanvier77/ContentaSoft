# AI Video Enhancer Studio: command-line reference

`aivideoenhancer` is the command line of AI Video Enhancer Studio. It is installed with the desktop app, in `%LOCALAPPDATA%\Programs\AIVideoEnhancerStudio\`, and the installer adds that folder to your user `PATH`.

Generated from `aivideoenhancer catalog`, version 2026.7.23. The same reference is on the website: [www.contenta-software.com/aivideoenhancer/docs/cli.html](https://www.contenta-software.com/aivideoenhancer/docs/cli.html).

```powershell
aivideoenhancer <command> [arguments] [options]
aivideoenhancer enhance tape.avi -o restored --preset tape_restoration
```

Every command has `--help`. Older spellings of some commands and options still work, so existing scripts keep running, but this reference and `--help` leave them out. The MCP server (`aivideoenhancer serve`) is documented in [mcp-server.md](mcp-server.md).

## Commands

| Command | What it does |
|---|---|
| [`aivideoenhancer batch <input>`](#aivideoenhancer-batch) | Enhance every video in a folder (enhance with folder filters). |
| [`aivideoenhancer catalog`](#aivideoenhancer-catalog) | Print every CLI command and MCP tool as JSON (for AI agents, docs and the website). |
| [`aivideoenhancer diagnose`](#aivideoenhancer-diagnose) | Generate a freeze-diagnostic zip (forensic log + Windows Event Log + GPU state). |
| [`aivideoenhancer enhance <input>`](#aivideoenhancer-enhance) | Enhance video files with AI. |
| [`aivideoenhancer extract-frames <input> --output <output>`](#aivideoenhancer-extract-frames) | Extract still frames from video(s) (optionally AI-upscale). |
| [`aivideoenhancer formats`](#aivideoenhancer-formats) | List the video containers it reads and writes, the output codecs and the still formats. |
| [`aivideoenhancer info <file>`](#aivideoenhancer-info) | Show a video file's metadata: codec, resolution, duration, frame rate, bitrate, audio, HDR. |
| [`aivideoenhancer presets`](#aivideoenhancer-presets) | List available enhancement presets. |
| [`aivideoenhancer register <email> <key>`](#aivideoenhancer-register) | Register your license key. |
| [`aivideoenhancer remix`](#aivideoenhancer-remix) | Suggest and render highlight-reel remixes from a video. |
| [`aivideoenhancer remix render <input> --clips <clips> --output <output>`](#aivideoenhancer-remix-render) | Render clip ranges into one H.264 MP4. |
| [`aivideoenhancer remix suggest <input>`](#aivideoenhancer-remix-suggest) | Suggest highlight clip ranges via scene detection. |
| [`aivideoenhancer serve`](#aivideoenhancer-serve) | Start MCP server for AI agent integration. |
| [`aivideoenhancer status`](#aivideoenhancer-status) | Show system status, GPU capabilities and license / trial state. |
| [`aivideoenhancer watch <input>`](#aivideoenhancer-watch) | Watch a folder and enhance each video that arrives (until Ctrl+C). |

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

### aivideoenhancer batch

Enhance every video in a folder (enhance with folder filters).

```
aivideoenhancer batch <input> [options]
```

| Argument | Description |
|---|---|
| `<input>` | Folder of videos to enhance. |

| Option | Type | Description |
|---|---|---|
| `--output` | string | Output directory (default: beside each source). |
| `--preset` | string | Preset name (e.g. tape_restoration for VHS/Hi8/MiniDV captures: CPU only, no graphics card needed; `presets` lists them all). With the other presets an interlaced source (DV, DVD, TV capture) gets --deinterlace yadif automatically unless you pass --deinterlace yourself. |
| `--profile` | string | Load the enhancement settings from a JSON file (the EnhancementProfile fields, e.g. {"upscale":"X2","denoise":"Light","outputCodec":"H265","crfValue":18}); the options below override it. Not with --preset. |
| `--upscale` | string | Upscale: off\|enhance\|x2\|x3\|x4 (enhance = clean up at same resolution). |
| `--denoise` | string | Denoise: off\|light\|medium\|strong. |
| `--denoise-method` | string | Denoise method: nlmeans\|hqdn3d\|chroma (default: hqdn3d; chroma = colour noise reduced much harder than luma, for tape). |
| `--stabilize` | string | Stabilize: off\|light\|medium\|strong. |
| `--deinterlace` | string | Deinterlace: off\|yadif\|yadifbob\|bwdif\|bwdifbob (bwdifbob = full motion: one frame per field, 25i becomes 50p, 29.97i becomes 59.94p). With --preset and no --deinterlace, interlaced sources get yadif; --deinterlace off keeps it off. |
| `--sharpen` | string | Sharpen: off\|light\|medium\|strong. |
| `--rolling-shutter` | string | Rolling-shutter (jello) correction: off\|light\|medium\|strong. |
| `--interpolate` | string | Target frame rate for smoother motion (RIFE): off\|30\|60. Only raises the rate — a clip already at/above the target is left unchanged. |
| `--codec` | string | Output codec: h264\|h265\|av1\|vp9. |
| `--crf` | integer | Quality (CRF value, 1-51, lower=better). |
| `--upscale-tile` | integer | Upscale tile size override (0=auto). Accepted for script compatibility; NVIDIA Video Super Resolution has no tiles and ignores it. |
| `--frame-batch` | integer | Frames per AI-runner invocation (0=auto, the default: sized from free VRAM). |
| `--temp-dir` | string | Scratch folder for frame extraction / interpolation temp data (default: the desktop app's Settings value, or system temp). |
| `--skip-existing` | boolean | Leave a video alone when its output file already exists (default: write a renamed copy beside it). A skipped video is not enhanced and costs no free file. |
| `--overwrite` | boolean | Replace an output file that already exists (default: write a renamed copy beside it). Not with --skip-existing. |
| `--compare` | boolean | Also write a side-by-side before/after video (source left, enhanced right, labelled underneath) next to each output as \<name\>\_compare.mp4. |
| `--compare-labels` | string | Custom labels for --compare, as "FIRST\|SECOND" (default: "BEFORE\|AFTER"). |
| `--compare-layout` | string | Layout for --compare: horizontal\|vertical. horizontal (default) = side by side; vertical = stacked top/bottom, which avoids the very wide 32:9 frame two 16:9 panels make side by side. |
| `--compare-seconds` | number | Length of the --compare video in seconds, taken a third of the way into the video (default: the whole video; 30 gives a short clip to show a customer). |
| `--hum` | string | Remove mains hum from the audio: off\|auto\|50\|60 (notches at the mains frequency and its harmonics; auto = 50 Hz for a 25/50 fps picture, 60 Hz for 29.97/59.94). Used with a preset or another enhancement option; the audio is re-encoded. |
| `--crop-bottom` | boolean | With --preset tape_restoration: crop the head-switching noise band at the bottom of a VHS/Video8/Hi8 capture (12 lines of 576, 8 of 480). The picture keeps its proportions, it is not stretched to fill. |
| `--stabilize-keep` | string | With --stabilize (or a preset that stabilizes): keep at least this percent of the picture, 50-100, or auto (default). Stabilizing zooms in to hide the moving edge, by as much as the shake needs; a limit stops the zoom there and leaves the part of the movement it has no room for. 100 = no zoom at all, the edge of the picture then moves. |
| `--hdr` | string | HDR10 / HLG sources: keep\|sdr (default: keep). keep = the output stays 10-bit HDR when the job has no AI stage, no stabilization, no rolling-shutter correction and no NLMeans denoise, and the codec is h265, av1 or vp9; otherwise the picture is tone-mapped to SDR and a line says so. sdr = always tone-map to SDR (BT.709). |
| `--upscale-engine` | string | What upscales x2\|x3\|x4: auto\|rtx\|standard. auto (default) = NVIDIA RTX Video Super Resolution where it can run, otherwise the standard upscaler (Lanczos resize + sharpening on the CPU, not AI; any PC). The engine is printed before the job starts. |
| `--include` | string[] | Only videos matching this pattern (e.g. \*.mp4, or mp4;mov); repeat the option or separate patterns with ';' or ','. |
| `--exclude` | string[] | Leave out videos whose name matches (same pattern forms as --include). |
| `--recursive` | boolean | Include subfolders (the output folder is never searched when it is inside the input). |
| `--workers` | integer | Videos enhanced at the same time (default 1; AI stages share one GPU, so more rarely helps). |
| `--report` | string | Write a CSV report, one row per video (input, status, output, trial_watermarked, error), to this file. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### aivideoenhancer catalog

Print every CLI command and MCP tool as JSON (for AI agents, docs and the website).

```
aivideoenhancer catalog
```

### aivideoenhancer diagnose

Generate a freeze-diagnostic zip (forensic log + Windows Event Log + GPU state).

```
aivideoenhancer diagnose
```

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### aivideoenhancer enhance

Enhance video files with AI.

```
aivideoenhancer enhance <input> [options]
```

| Argument | Description |
|---|---|
| `<input>` | Input video file or directory (a directory is searched with its subfolders; `batch` has the folder filters). |

| Option | Type | Description |
|---|---|---|
| `--output` | string | Output file or folder. A folder: an existing one, a path ending in \\ or /, or a name without an extension (the output is named after the input); anything else is the output file; a file must end in .mp4, .mov or .mkv (it picks the container). Default: beside the source. |
| `--preset` | string | Preset name (e.g. tape_restoration for VHS/Hi8/MiniDV captures: CPU only, no graphics card needed; `presets` lists them all). With the other presets an interlaced source (DV, DVD, TV capture) gets --deinterlace yadif automatically unless you pass --deinterlace yourself. |
| `--profile` | string | Load the enhancement settings from a JSON file (the EnhancementProfile fields, e.g. {"upscale":"X2","denoise":"Light","outputCodec":"H265","crfValue":18}); the options below override it. Not with --preset. |
| `--upscale` | string | Upscale: off\|enhance\|x2\|x3\|x4 (enhance = clean up at same resolution). |
| `--denoise` | string | Denoise: off\|light\|medium\|strong. |
| `--denoise-method` | string | Denoise method: nlmeans\|hqdn3d\|chroma (default: hqdn3d; chroma = colour noise reduced much harder than luma, for tape). |
| `--stabilize` | string | Stabilize: off\|light\|medium\|strong. |
| `--deinterlace` | string | Deinterlace: off\|yadif\|yadifbob\|bwdif\|bwdifbob (bwdifbob = full motion: one frame per field, 25i becomes 50p, 29.97i becomes 59.94p). With --preset and no --deinterlace, interlaced sources get yadif; --deinterlace off keeps it off. |
| `--sharpen` | string | Sharpen: off\|light\|medium\|strong. |
| `--rolling-shutter` | string | Rolling-shutter (jello) correction: off\|light\|medium\|strong. |
| `--interpolate` | string | Target frame rate for smoother motion (RIFE): off\|30\|60. Only raises the rate — a clip already at/above the target is left unchanged. |
| `--codec` | string | Output codec: h264\|h265\|av1\|vp9. |
| `--crf` | integer | Quality (CRF value, 1-51, lower=better). |
| `--upscale-tile` | integer | Upscale tile size override (0=auto). Accepted for script compatibility; NVIDIA Video Super Resolution has no tiles and ignores it. |
| `--frame-batch` | integer | Frames per AI-runner invocation (0=auto, the default: sized from free VRAM). |
| `--temp-dir` | string | Scratch folder for frame extraction / interpolation temp data (default: the desktop app's Settings value, or system temp). |
| `--skip-existing` | boolean | Leave a video alone when its output file already exists (default: write a renamed copy beside it). A skipped video is not enhanced and costs no free file. |
| `--overwrite` | boolean | Replace an output file that already exists (default: write a renamed copy beside it). Not with --skip-existing. |
| `--compare` | boolean | Also write a side-by-side before/after video (source left, enhanced right, labelled underneath) next to each output as \<name\>\_compare.mp4. |
| `--compare-labels` | string | Custom labels for --compare, as "FIRST\|SECOND" (default: "BEFORE\|AFTER"). |
| `--compare-layout` | string | Layout for --compare: horizontal\|vertical. horizontal (default) = side by side; vertical = stacked top/bottom, which avoids the very wide 32:9 frame two 16:9 panels make side by side. |
| `--compare-seconds` | number | Length of the --compare video in seconds, taken a third of the way into the video (default: the whole video; 30 gives a short clip to show a customer). |
| `--clip` | string | Export only a 10-second clip starting at this time (seconds, or m:ss / h:mm:ss) of ONE input file, named \<name\>-clip. On the trial a clip is clean and full resolution and does not use a free export; clips from one video can cover up to 30 seconds a day. |
| `--hum` | string | Remove mains hum from the audio: off\|auto\|50\|60 (notches at the mains frequency and its harmonics; auto = 50 Hz for a 25/50 fps picture, 60 Hz for 29.97/59.94). Used with a preset or another enhancement option; the audio is re-encoded. |
| `--crop-bottom` | boolean | With --preset tape_restoration: crop the head-switching noise band at the bottom of a VHS/Video8/Hi8 capture (12 lines of 576, 8 of 480). The picture keeps its proportions, it is not stretched to fill. |
| `--stabilize-keep` | string | With --stabilize (or a preset that stabilizes): keep at least this percent of the picture, 50-100, or auto (default). Stabilizing zooms in to hide the moving edge, by as much as the shake needs; a limit stops the zoom there and leaves the part of the movement it has no room for. 100 = no zoom at all, the edge of the picture then moves. |
| `--hdr` | string | HDR10 / HLG sources: keep\|sdr (default: keep). keep = the output stays 10-bit HDR when the job has no AI stage, no stabilization, no rolling-shutter correction and no NLMeans denoise, and the codec is h265, av1 or vp9; otherwise the picture is tone-mapped to SDR and a line says so. sdr = always tone-map to SDR (BT.709). |
| `--upscale-engine` | string | What upscales x2\|x3\|x4: auto\|rtx\|standard. auto (default) = NVIDIA RTX Video Super Resolution where it can run, otherwise the standard upscaler (Lanczos resize + sharpening on the CPU, not AI; any PC). The engine is printed before the job starts. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### aivideoenhancer extract-frames

Extract still frames from video(s) (optionally AI-upscale).

```
aivideoenhancer extract-frames <input> --output <output> [options]
```

| Argument | Description |
|---|---|
| `<input>` | Input video file or directory. |

| Option | Type | Description |
|---|---|---|
| `--output` | string, required | Output root directory (per-video subfolders created). |
| `--preset` | string | Sampling preset: storyboard\|thumbnails\|social\|scenes\|custom (scenes = one frame per detected scene change). |
| `--count` | integer | Evenly spaced frame count (implies custom even strategy); with --preset scenes it caps the top-N scenes by score. |
| `--timestamps` | string | Comma-separated seek positions in seconds (the same positions for every video). |
| `--upscale` | string | Upscale: off\|enhance\|x2\|x3\|x4 (default: off). |
| `--auto-upscale-below` | integer | If \>0 and --upscale off, use x2 when height is below this (0=off). |
| `--format` | enum | Still format: png (lossless, default), jpg or webp. |
| `--quality` | integer | Lossy quality 1-100 for jpg/webp (default 90; ignored for png). |
| `--sharpest` | boolean | Per frame, compare 5 nearby candidates (+/-0.4s, +/-0.2s, exact) by Laplacian-variance sharpness and keep the sharpest (~5x extra small seeks per frame). |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### aivideoenhancer formats

List the video containers it reads and writes, the output codecs and the still formats.

```
aivideoenhancer formats [options]
```

| Option | Type | Description |
|---|---|---|
| `--input` | boolean | Show only what is read. |
| `--output` | boolean | Show only what is written. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### aivideoenhancer info

Show a video file's metadata: codec, resolution, duration, frame rate, bitrate, audio, HDR.

```
aivideoenhancer info <file>
```

| Argument | Description |
|---|---|
| `<file>` | Video file to inspect. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### aivideoenhancer presets

List available enhancement presets.

```
aivideoenhancer presets
```

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### aivideoenhancer register

Register your license key.

```
aivideoenhancer register <email> <key>
```

| Argument | Description |
|---|---|
| `<email>` | Registration email. |
| `<key>` | License key. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### aivideoenhancer remix

Suggest and render highlight-reel remixes from a video.

Subcommands: `aivideoenhancer remix render`, `aivideoenhancer remix suggest`.

### aivideoenhancer remix render

Render clip ranges into one H.264 MP4.

```
aivideoenhancer remix render <input> --clips <clips> --output <output> [options]
```

| Argument | Description |
|---|---|
| `<input>` | Input video file. |

| Option | Type | Description |
|---|---|---|
| `--clips` | string, required | Clip ranges: "start-end,start-end,..." (sec or hh:mm:ss). |
| `--output` | string, required | Output MP4 path. |
| `--max-duration` | number | Cap total output length (seconds). |
| `--mute` | boolean | Drop audio. |
| `--width` | integer | Scale output to this width (height auto, even). |
| `--aspect` | enum | Center-crop to an aspect ratio: original, 9:16 (vertical Shorts/Reels), 1:1 (square) or 16:9 (wide). |
| `--music` | string | Background music audio file (mp3/wav/m4a/flac); looped to cover the reel, 2s fade-out at the end. |
| `--music-volume` | integer | Music volume 0-100 (default 30). |
| `--duck` | boolean | Duck the music under the original audio (sidechain compression); needs source audio, ignored with --mute. |

### aivideoenhancer remix suggest

Suggest highlight clip ranges via scene detection.

```
aivideoenhancer remix suggest <input> [options]
```

| Argument | Description |
|---|---|
| `<input>` | Input video file. |

| Option | Type | Description |
|---|---|---|
| `--duration` | number | Target total highlight duration in seconds. |
| `--clips` | integer | Number of clips (default derived from --duration, 5-8). |
| `--window-start` | string | Analysis window start (sec or hh:mm:ss); required for videos \> 4 hours. |
| `--window-end` | string | Analysis window end (sec or hh:mm:ss); defaults to start + up to 4 hours. |
| `--min-clip` | number | Minimum clip length (seconds). |
| `--max-clip` | number | Maximum clip length (seconds). |
| `--threshold` | number | Scene-detection threshold 0-100 (higher = fewer cuts). |
| `--audio-weight` | number | Blend weight 0-1 for audio-loudness spikes vs scene changes (0 = scene detection only); finds loud moments in static-camera footage. |

### aivideoenhancer serve

Start MCP server for AI agent integration.

```
aivideoenhancer serve
```

### aivideoenhancer status

Show system status, GPU capabilities and license / trial state.

```
aivideoenhancer status
```

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### aivideoenhancer watch

Watch a folder and enhance each video that arrives (until Ctrl+C).

```
aivideoenhancer watch <input> [options]
```

| Argument | Description |
|---|---|
| `<input>` | Folder to watch. |

| Option | Type | Description |
|---|---|---|
| `--output` | string | Output directory (default: beside each source). |
| `--preset` | string | Preset name (e.g. tape_restoration for VHS/Hi8/MiniDV captures: CPU only, no graphics card needed; `presets` lists them all). With the other presets an interlaced source (DV, DVD, TV capture) gets --deinterlace yadif automatically unless you pass --deinterlace yourself. |
| `--profile` | string | Load the enhancement settings from a JSON file (the EnhancementProfile fields, e.g. {"upscale":"X2","denoise":"Light","outputCodec":"H265","crfValue":18}); the options below override it. Not with --preset. |
| `--upscale` | string | Upscale: off\|enhance\|x2\|x3\|x4 (enhance = clean up at same resolution). |
| `--denoise` | string | Denoise: off\|light\|medium\|strong. |
| `--denoise-method` | string | Denoise method: nlmeans\|hqdn3d\|chroma (default: hqdn3d; chroma = colour noise reduced much harder than luma, for tape). |
| `--stabilize` | string | Stabilize: off\|light\|medium\|strong. |
| `--deinterlace` | string | Deinterlace: off\|yadif\|yadifbob\|bwdif\|bwdifbob (bwdifbob = full motion: one frame per field, 25i becomes 50p, 29.97i becomes 59.94p). With --preset and no --deinterlace, interlaced sources get yadif; --deinterlace off keeps it off. |
| `--sharpen` | string | Sharpen: off\|light\|medium\|strong. |
| `--rolling-shutter` | string | Rolling-shutter (jello) correction: off\|light\|medium\|strong. |
| `--interpolate` | string | Target frame rate for smoother motion (RIFE): off\|30\|60. Only raises the rate — a clip already at/above the target is left unchanged. |
| `--codec` | string | Output codec: h264\|h265\|av1\|vp9. |
| `--crf` | integer | Quality (CRF value, 1-51, lower=better). |
| `--upscale-tile` | integer | Upscale tile size override (0=auto). Accepted for script compatibility; NVIDIA Video Super Resolution has no tiles and ignores it. |
| `--frame-batch` | integer | Frames per AI-runner invocation (0=auto, the default: sized from free VRAM). |
| `--temp-dir` | string | Scratch folder for frame extraction / interpolation temp data (default: the desktop app's Settings value, or system temp). |
| `--skip-existing` | boolean | Leave a video alone when its output file already exists (default: write a renamed copy beside it). A skipped video is not enhanced and costs no free file. |
| `--overwrite` | boolean | Replace an output file that already exists (default: write a renamed copy beside it). Not with --skip-existing. |
| `--compare` | boolean | Also write a side-by-side before/after video (source left, enhanced right, labelled underneath) next to each output as \<name\>\_compare.mp4. |
| `--compare-labels` | string | Custom labels for --compare, as "FIRST\|SECOND" (default: "BEFORE\|AFTER"). |
| `--compare-layout` | string | Layout for --compare: horizontal\|vertical. horizontal (default) = side by side; vertical = stacked top/bottom, which avoids the very wide 32:9 frame two 16:9 panels make side by side. |
| `--compare-seconds` | number | Length of the --compare video in seconds, taken a third of the way into the video (default: the whole video; 30 gives a short clip to show a customer). |
| `--hum` | string | Remove mains hum from the audio: off\|auto\|50\|60 (notches at the mains frequency and its harmonics; auto = 50 Hz for a 25/50 fps picture, 60 Hz for 29.97/59.94). Used with a preset or another enhancement option; the audio is re-encoded. |
| `--crop-bottom` | boolean | With --preset tape_restoration: crop the head-switching noise band at the bottom of a VHS/Video8/Hi8 capture (12 lines of 576, 8 of 480). The picture keeps its proportions, it is not stretched to fill. |
| `--stabilize-keep` | string | With --stabilize (or a preset that stabilizes): keep at least this percent of the picture, 50-100, or auto (default). Stabilizing zooms in to hide the moving edge, by as much as the shake needs; a limit stops the zoom there and leaves the part of the movement it has no room for. 100 = no zoom at all, the edge of the picture then moves. |
| `--hdr` | string | HDR10 / HLG sources: keep\|sdr (default: keep). keep = the output stays 10-bit HDR when the job has no AI stage, no stabilization, no rolling-shutter correction and no NLMeans denoise, and the codec is h265, av1 or vp9; otherwise the picture is tone-mapped to SDR and a line says so. sdr = always tone-map to SDR (BT.709). |
| `--upscale-engine` | string | What upscales x2\|x3\|x4: auto\|rtx\|standard. auto (default) = NVIDIA RTX Video Super Resolution where it can run, otherwise the standard upscaler (Lanczos resize + sharpening on the CPU, not AI; any PC). The engine is printed before the job starts. |
| `--include` | string[] | Only videos matching this pattern (e.g. \*.mp4, or mp4;mov); repeat the option or separate patterns with ';' or ','. |
| `--exclude` | string[] | Leave out videos whose name matches (same pattern forms as --include). |
| `--recursive` | boolean | Watch subfolders too (the output folder is never watched when it is inside the input). |
| `--interval` | integer | Seconds between two looks at the folder (default 10, at least 2). A new file is taken once it held still for one interval. |
| `--report` | string | When the watch stops, write a CSV report, one row per video (input, status, output, trial_watermarked, error), to this file. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.
