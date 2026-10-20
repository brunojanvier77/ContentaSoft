# Contenta Converter: command-line reference

`contenta` is the command line of Contenta Converter. It is installed with the desktop app, in `%LOCALAPPDATA%\Programs\ContentaConverter\`, and the installer adds that folder to your user `PATH`.

Generated from `contenta catalog`, version 9.0.43. The same reference is on the website: [www.contenta-converter.com/docs/cli.html](https://www.contenta-converter.com/docs/cli.html).

```powershell
contenta <command> [arguments] [options]
contenta convert photo.cr2 -o out --format jpg
```

Every command has `--help`. Older spellings of some commands and options still work, so existing scripts keep running, but this reference and `--help` leave them out. The MCP server (`contenta serve`) is documented in [mcp-server.md](mcp-server.md).

## Commands

| Command | What it does |
|---|---|
| [`contenta ai-transform <input>`](#contenta-ai-transform) | AI-powered image transformation via Google Gemini. |
| [`contenta batch <input-dir>`](#contenta-batch) | Batch convert images in a directory. With --json, stdout is one result object {"type":"summary","success":true\|false,"exitCode":N,"succeeded":N,...}. Add --progress for one line per finished output before it, {"type":"progress","file":NAME,"status":"done\|skipped\|failed","completed":N,"total":T} (the result object stays last): skip the lines that start with {"type":"progress". |
| [`contenta catalog`](#contenta-catalog) | Print every CLI command and MCP tool as JSON (for AI agents, docs and the website). |
| [`contenta convert <input>`](#contenta-convert) | Convert a single image file. |
| [`contenta effects`](#contenta-effects) | List all available image effects. |
| [`contenta formats`](#contenta-formats) | List supported input and output formats. |
| [`contenta info <file>`](#contenta-info) | Show format detection and metadata for an image file. |
| [`contenta pdf-album <files>...`](#contenta-pdf-album) | Create a PDF photo album from multiple images. During the trial the PDF always carries the trial watermark. |
| [`contenta pdf-merge <files>...`](#contenta-pdf-merge) | Append multiple PDF files into a single PDF (page-level merge, no rasterization). During the trial the PDF always carries the trial watermark. |
| [`contenta profile`](#contenta-profile) | Save, load, or validate conversion profiles. |
| [`contenta profile load <file>`](#contenta-profile-load) | Load and display a profile from a JSON file. |
| [`contenta profile save <file>`](#contenta-profile-save) | Save current options as a profile JSON file. |
| [`contenta profile validate <file>`](#contenta-profile-validate) | Validate a profile JSON file. |
| [`contenta register <email> <key>`](#contenta-register) | Register your license key. |
| [`contenta serve`](#contenta-serve) | Start MCP server for AI agent integration. |
| [`contenta slideshow <files>...`](#contenta-slideshow) | Create a video slideshow from images. During the trial the video always ends with the branding end card. |
| [`contenta status`](#contenta-status) | Show license status, tools availability, and version info. |
| [`contenta watch <dir>`](#contenta-watch) | Watch a directory and auto-convert new/changed images. |

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

### contenta ai-transform

AI-powered image transformation via Google Gemini.

```
contenta ai-transform <input> [options]
```

| Argument | Description |
|---|---|
| `<input>` | Input image file path. |

| Option | Type | Description |
|---|---|---|
| `--prompt` | string | AI transformation prompt (required unless --remove-background). |
| `--api-key` | string | Gemini API key (or set GEMINI_API_KEY env var). |
| `--model` | string | Model tier: fast (default), pro or lite; or an explicit Gemini model ID. |
| `--size` | string | Output size: 1K (default), 2K or 4K (lite makes 1K only). |
| `--output` | string | Output file path. |
| `--remove-background` | boolean | Remove the background (no prompt needed; a given prompt is not used). |
| `--bg-mode` | string | Background mode: white, transparent, custom. |
| `--bg-color` | string | Custom background color (hex #RRGGBB). |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### contenta batch

Batch convert images in a directory. With --json, stdout is one result object {"type":"summary","success":true|false,"exitCode":N,"succeeded":N,...}. Add --progress for one line per finished output before it, {"type":"progress","file":NAME,"status":"done|skipped|failed","completed":N,"total":T} (the result object stays last): skip the lines that start with {"type":"progress".

```
contenta batch <input-dir> [options]
```

| Argument | Description |
|---|---|
| `<input-dir>` | Input directory. |

| Option | Type | Description |
|---|---|---|
| `--include` | string[] | Only files matching this pattern (e.g. \*.jpg, or jpg;png); repeat the option or separate patterns with ';' or ','. |
| `--exclude` | string[] | Leave out files matching this pattern: a glob such as \*.png or thumb\_\*, or text in the path below the folder; repeat the option or separate patterns with ';' or ','. |
| `--recursive` | boolean | Include subdirectories. |
| `--workers` | integer | Parallel workers (default: automatic, see WorkerPolicy). |
| `--report` | string | Write a CSV report to this file: one row per output attempted, with the input, done/skipped/failed, the files written and the error. |
| `--output` | string | Output directory. |
| `--format` | string | Output format. |
| `--quality` | integer | Quality 1-100. |
| `--resize` | string | Resize to WxH. |
| `--resize-mode` | string | Resize mode: fit, fill, stretch, longest-edge, shortest-edge, fit-with-background, crop-to-aspect (default: fit). fit-with-background fits the picture inside --resize and fills the rest with --background. |
| `--background-color` | string | Colour around the picture with --resize-mode fit-with-background: white, black, grey, lightgrey or #RRGGBB (default: white). --resize 2000x2000 --resize-mode fit-with-background --background white makes white square product images. |
| `--resize-percent` | integer | Resize by percentage. |
| `--dont-enlarge` | boolean | Don't upscale smaller images. |
| `--dpi` | integer | Set DPI. |
| `--effects` | string[] | Effects, separated by spaces or commas, each name or name:key=value (e.g. sepia,sharpen:window=5). Run 'contenta effects' for names and parameters. |
| `--lut` | string | Apply a 3D LUT (.cube file) to every image, after resize and the other effects. With --profile it is added to the profile, in place of the profile's own LUT if it has one. |
| `--lut-strength` | integer | How much of the LUT to apply, 0 to 100 (default 100). Needs --lut, or a --profile that has a LUT. |
| `--watermark` | string | Add a white text watermark (width = 10% of the image). |
| `--watermark-position` | integer | Watermark position 0-8 (default: 8 = bottom-right). |
| `--watermark-opacity` | integer | Watermark opacity 0-100, 100 = fully visible (default: 70). |
| `--copyright` | string | Copyright metadata. |
| `--creator` | string | Creator metadata. |
| `--filename-pattern` | string | Output filename pattern (default: {originalname}). Tokens: {originalname} {counter:3} {today} {datetime} {size} and the Batch Rename tokens {name} {ext} {date} {date:FORMAT} {time} {year} {month} {camera} {make} {lens} {iso} {aperture} {focal} {w} {h} {gps} {seq} {seq:N} {folder}, filled from the source photo ({date} = date taken, {today} = the run date). |
| `--overwrite` | boolean | Overwrite existing. |
| `--skip-existing` | boolean | Skip an input whose output already exists (reported as skipped), so a re-run only converts what is missing. |
| `--color-space` | string | Color space: srgb, adobe-rgb, prophoto-rgb, preserve. srgb (the default) converts a source saved in another color space, such as an iPhone's Display P3; preserve leaves it as it is. |
| `--png-compression` | integer | PNG compression, zlib level 0-9 (0 = none/fastest, 9 = smallest; default 2). |
| `--webp-lossless` | boolean | WebP lossless. |
| `--jxl-lossless` | boolean | JXL lossless. |
| `--preserve-metadata` | boolean | Preserve metadata. |
| `--profile` | string | Load profile from JSON. |
| `--lens-corrections` | boolean | Apply the lens corrections a DNG carries (distortion, vignetting, shading; on by default; --lens-corrections false turns them off). |
| `--title` | string | Set title metadata. |
| `--description` | string | Set description metadata. |
| `--keywords` | string | Set keywords (comma-separated). |
| `--rights` | string | Set rights metadata. |
| `--creator-city` | string | Creator city. |
| `--creator-country` | string | Creator country. |
| `--creator-email` | string | Creator email. |
| `--creator-url` | string | Creator URL. |
| `--gps` | string | GPS coordinates (lat,lon or lat,lon,alt). |
| `--datetime` | string | Date/time override (ISO 8601). |
| `--aspect-ratio` | string | Custom aspect ratio (e.g., 16:9). |
| `--dpi-x` | integer | Horizontal DPI. |
| `--dpi-y` | integer | Vertical DPI. |
| `--sizes` | string | Emit multiple sizes at once: pixels (16,32,48,64) or scale factors (1x,2x,3x). |
| `--size-preset` | string | Built-in size set: favicon, appicon, srcset. |
| `--base-size` | integer | Base edge in px that scale factors (Nx) multiply (default: 512). |
| `--subfolder-mode` | string | Subfolder mode: flatten, preserve, template. |
| `--subfolder-pattern` | string | Subfolder template pattern. |
| `--backup-originals` | boolean | Backup originals. |
| `--backup-dir` | string | Backup directory. |
| `--pdf-page` | integer | Convert only this page (0-based: 0 is the first) of a multi-page PDF or TIFF. Without it every page is converted: one file each (name_page001.jpg, name_page002.jpg, ...), or one multi-page file when the output format is pdf or tiff. |
| `--zip-output` | boolean | After the batch, also pack every converted file into contenta-converter-images.zip in the output folder (the files stay in place; an existing archive of that name is replaced). |
| `--zip-split-size` | integer | With --zip-output: split into independent archives (contenta-converter-images_part2.zip, \_part3...), each holding at most this many MB of files, every one opens on its own; a single larger file gets an archive to itself (default: 25). |
| `--rename-pattern` | string | After the batch, rename every converted file with the app's Batch Rename tokens: {name} {ext} {date} {date:FORMAT} {time} {year} {month} {camera} {make} {lens} {iso} {aperture} {focal} {w} {h} {gps} {seq} {seq:N} {folder}. The extension is kept; a name already taken gets \_1, \_2... |
| `--white-balance` | string | RAW white balance: camera, auto, manual. |
| `--color-temperature` | integer | Manual WB: temperature of the light the picture was taken in, in Kelvin (2000-10000, default 5500 = daylight; a lower value gives a cooler picture, a higher one a warmer picture). |
| `--denoising` | boolean | Apply wavelet denoising during RAW decode (off by default). |
| `--denoising-level` | integer | Denoising threshold (0-1000, default 300); giving a level turns denoising on. |
| `--sharpness-boost` | boolean | Sharpen after RAW decode (on by default for RAW input, as in the app; --sharpness-boost false turns it off; off by default with --raw-16bit). |
| `--color-boost` | boolean | Apply saturation boost after RAW decode. |
| `--contrast-boost` | boolean | Apply contrast boost after RAW decode. |
| `--exposure` | number | RAW exposure compensation in stops (EV), -3.0 to 3.0, default 0: +1 doubles the light, -1 halves it. |
| `--fast-raw-decode` | boolean | Faster RAW decode at half resolution when the output is at most half the size of the sensor image (slightly softer; for web and preview exports). |
| `--raw-16bit` | boolean | Decode RAW at 16 bits per channel and write a 16-bit TIFF. Needs -f tiff and no effects, RAW boosts, watermark or embedded preview (they work on 8 bits per channel); otherwise the file is written with 8 bits and the result carries a warning. Turns the default RAW sharpening off. |
| `--embedded-preview` | boolean | Convert the JPEG preview the camera stored in the RAW file instead of developing the RAW data: the fastest way, but the preview can be smaller than the full picture, already has the camera's look, and is 8-bit; white balance, denoising and exposure do not apply to it. A file without a preview is developed as usual. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### contenta catalog

Print every CLI command and MCP tool as JSON (for AI agents, docs and the website).

```
contenta catalog
```

### contenta convert

Convert a single image file.

```
contenta convert <input> [options]
```

| Argument | Description |
|---|---|
| `<input>` | Input image file path. |

| Option | Type | Description |
|---|---|---|
| `--output` | string | Output directory (default: same as input). |
| `--format` | string | Output format (jpg, png, webp, tiff, bmp, gif, jxl, heic, avif, svg, pdf, ico). |
| `--quality` | integer | Quality 1-100. |
| `--resize` | string | Resize to WxH (e.g., 1920x1080). |
| `--resize-mode` | string | Resize mode: fit, fill, stretch, longest-edge, shortest-edge, fit-with-background, crop-to-aspect (default: fit). fit-with-background fits the picture inside --resize and fills the rest with --background. |
| `--background-color` | string | Colour around the picture with --resize-mode fit-with-background: white, black, grey, lightgrey or #RRGGBB (default: white). --resize 2000x2000 --resize-mode fit-with-background --background white makes white square product images. |
| `--resize-percent` | integer | Resize by percentage (1-100). |
| `--dont-enlarge` | boolean | Don't upscale smaller images. |
| `--dpi` | integer | Set output DPI. |
| `--effects` | string[] | Effects, separated by spaces or commas, each name or name:key=value (e.g. sepia,sharpen:window=5). Run 'contenta effects' for names and parameters. |
| `--lut` | string | Apply a 3D LUT (.cube file) to every image, after resize and the other effects. With --profile it is added to the profile, in place of the profile's own LUT if it has one. |
| `--lut-strength` | integer | How much of the LUT to apply, 0 to 100 (default 100). Needs --lut, or a --profile that has a LUT. |
| `--watermark` | string | Add a white text watermark (width = 10% of the image). |
| `--watermark-position` | integer | Watermark position 0-8 (default: 8 = bottom-right). |
| `--watermark-opacity` | integer | Watermark opacity 0-100, 100 = fully visible (default: 70). |
| `--copyright` | string | Set copyright metadata. |
| `--creator` | string | Set creator metadata. |
| `--filename-pattern` | string | Output filename pattern (default: {originalname}). Tokens: {originalname} {counter:3} {today} {datetime} {size} and the Batch Rename tokens {name} {ext} {date} {date:FORMAT} {time} {year} {month} {camera} {make} {lens} {iso} {aperture} {focal} {w} {h} {gps} {seq} {seq:N} {folder}, filled from the source photo ({date} = date taken, {today} = the run date). |
| `--overwrite` | boolean | Overwrite existing files. |
| `--skip-existing` | boolean | Skip an input whose output already exists (reported as skipped), so a re-run only converts what is missing. |
| `--color-space` | string | Color space: srgb, adobe-rgb, prophoto-rgb, preserve. srgb (the default) converts a source saved in another color space, such as an iPhone's Display P3; preserve leaves it as it is. |
| `--png-compression` | integer | PNG compression, zlib level 0-9 (0 = none/fastest, 9 = smallest; default 2). |
| `--webp-lossless` | boolean | WebP lossless mode. |
| `--jxl-lossless` | boolean | JXL lossless mode. |
| `--preserve-metadata` | boolean | Preserve original metadata. |
| `--profile` | string | Load conversion profile from JSON file. |
| `--lens-corrections` | boolean | Apply the lens corrections a DNG carries (distortion, vignetting, shading; on by default; --lens-corrections false turns them off). |
| `--title` | string | Set title metadata. |
| `--description` | string | Set description metadata. |
| `--keywords` | string | Set keywords metadata (comma-separated). |
| `--rights` | string | Set rights/usage terms metadata. |
| `--creator-city` | string | Set creator city metadata. |
| `--creator-country` | string | Set creator country metadata. |
| `--creator-email` | string | Set creator email metadata. |
| `--creator-url` | string | Set creator URL metadata. |
| `--gps` | string | Set GPS coordinates (lat,lon or lat,lon,alt). |
| `--datetime` | string | Set date/time override (ISO 8601). |
| `--aspect-ratio` | string | Custom aspect ratio (e.g., 16:9). |
| `--dpi-x` | integer | Set horizontal DPI. |
| `--dpi-y` | integer | Set vertical DPI. |
| `--sizes` | string | Emit multiple sizes at once: pixels (16,32,48,64) or scale factors (1x,2x,3x). |
| `--size-preset` | string | Built-in size set: favicon, appicon, srcset. |
| `--base-size` | integer | Base edge in px that scale factors (Nx) multiply (default: 512). |
| `--subfolder-mode` | string | Subfolder mode: flatten, preserve, template. |
| `--subfolder-pattern` | string | Subfolder template pattern (e.g., {year}/{month}). |
| `--backup-originals` | boolean | Backup original files before conversion. |
| `--backup-dir` | string | Backup directory path. |
| `--pdf-page` | integer | Convert only this page (0-based: 0 is the first) of a multi-page PDF or TIFF. Without it every page is converted: one file each (name_page001.jpg, name_page002.jpg, ...), or one multi-page file when the output format is pdf or tiff. |
| `--white-balance` | string | RAW white balance: camera, auto, manual. |
| `--color-temperature` | integer | Manual WB: temperature of the light the picture was taken in, in Kelvin (2000-10000, default 5500 = daylight; a lower value gives a cooler picture, a higher one a warmer picture). |
| `--denoising` | boolean | Apply wavelet denoising during RAW decode (off by default). |
| `--denoising-level` | integer | Denoising threshold (0-1000, default 300); giving a level turns denoising on. |
| `--sharpness-boost` | boolean | Sharpen after RAW decode (on by default for RAW input, as in the app; --sharpness-boost false turns it off; off by default with --raw-16bit). |
| `--color-boost` | boolean | Apply saturation boost after RAW decode. |
| `--contrast-boost` | boolean | Apply contrast boost after RAW decode. |
| `--exposure` | number | RAW exposure compensation in stops (EV), -3.0 to 3.0, default 0: +1 doubles the light, -1 halves it. |
| `--fast-raw-decode` | boolean | Faster RAW decode at half resolution when the output is at most half the size of the sensor image (slightly softer; for web and preview exports). |
| `--raw-16bit` | boolean | Decode RAW at 16 bits per channel and write a 16-bit TIFF. Needs -f tiff and no effects, RAW boosts, watermark or embedded preview (they work on 8 bits per channel); otherwise the file is written with 8 bits and the result carries a warning. Turns the default RAW sharpening off. |
| `--embedded-preview` | boolean | Convert the JPEG preview the camera stored in the RAW file instead of developing the RAW data: the fastest way, but the preview can be smaller than the full picture, already has the camera's look, and is 8-bit; white balance, denoising and exposure do not apply to it. A file without a preview is developed as usual. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### contenta effects

List all available image effects.

```
contenta effects
```

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### contenta formats

List supported input and output formats.

```
contenta formats [options]
```

| Option | Type | Description |
|---|---|---|
| `--input` | boolean | Show input formats only. |
| `--output` | boolean | Show output formats only. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### contenta info

Show format detection and metadata for an image file.

```
contenta info <file>
```

| Argument | Description |
|---|---|
| `<file>` | Image file path. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### contenta pdf-album

Create a PDF photo album from multiple images. During the trial the PDF always carries the trial watermark.

```
contenta pdf-album <files>... [options]
```

| Argument | Description |
|---|---|
| `<files>` | Input image files. |

| Option | Type | Description |
|---|---|---|
| `--output` | string | Output PDF path. |
| `--photos-per-page` | integer | Photos per page (1, 2, 4, 6, 8, 10, 12, 16, 24, 48). |
| `--page-size` | string | Page size: A4, Letter, Legal. |
| `--orientation` | string | Orientation: Auto, Portrait, Landscape. |
| `--background` | string | Background color (hex, e.g., #FF000000). |
| `--background-image` | string | Background image path. |
| `--background-mode` | string | Background image mode: stretch, center, tile. |
| `--title-page` | string | Title page image path. |
| `--title-page-mode` | string | Title page image mode: stretch, center, tile. |
| `--show-filenames` | boolean | Show filenames below photos. |
| `--show-filenames-color` | string | Filename label color (hex, e.g., #FFFFFFFF). |
| `--embed-quality` | integer | JPEG quality 1-100 for non-JPEG inputs (PNG, TIFF, RAW...); JPEG inputs are embedded unchanged. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### contenta pdf-merge

Append multiple PDF files into a single PDF (page-level merge, no rasterization). During the trial the PDF always carries the trial watermark.

```
contenta pdf-merge <files>... [options]
```

| Argument | Description |
|---|---|
| `<files>` | Input PDF files (appended in argument order). |

| Option | Type | Description |
|---|---|---|
| `--output` | string | Output PDF path. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### contenta profile

Save, load, or validate conversion profiles.

Subcommands: `contenta profile load`, `contenta profile save`, `contenta profile validate`.

### contenta profile load

Load and display a profile from a JSON file.

```
contenta profile load <file>
```

| Argument | Description |
|---|---|
| `<file>` | Profile JSON path to load. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### contenta profile save

Save current options as a profile JSON file.

```
contenta profile save <file> [options]
```

| Argument | Description |
|---|---|
| `<file>` | Output profile JSON path. |

| Option | Type | Description |
|---|---|---|
| `--format` | string | Output format. |
| `--quality` | integer | Quality 1-100. |
| `--resize` | string | Resize to WxH. |
| `--resize-mode` | string | Resize mode: fit, fill, stretch, longest-edge, shortest-edge, fit-with-background, crop-to-aspect (default: fit). fit-with-background fits the picture inside --resize and fills the rest with --background. |
| `--background-color` | string | Colour around the picture with --resize-mode fit-with-background: white, black, grey, lightgrey or #RRGGBB (default: white). --resize 2000x2000 --resize-mode fit-with-background --background white makes white square product images. |
| `--effects` | string[] | Effects, separated by spaces or commas, each name or name:key=value (e.g. sepia,sharpen:window=5). Run 'contenta effects' for names and parameters. |
| `--copyright` | string | Copyright metadata. |
| `--creator` | string | Creator metadata. |
| `--overwrite` | boolean | Overwrite existing files. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### contenta profile validate

Validate a profile JSON file.

```
contenta profile validate <file>
```

| Argument | Description |
|---|---|
| `<file>` | Profile JSON path to validate. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### contenta register

Register your license key.

```
contenta register <email> <key>
```

| Argument | Description |
|---|---|
| `<email>` | Registration email. |
| `<key>` | 25-character license key. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### contenta serve

Start MCP server for AI agent integration.

```
contenta serve
```

### contenta slideshow

Create a video slideshow from images. During the trial the video always ends with the branding end card.

```
contenta slideshow <files>... [options]
```

| Argument | Description |
|---|---|
| `<files>` | Input image files. |

| Option | Type | Description |
|---|---|---|
| `--output` | string | Output video file path. |
| `--template` | string | Template: tiktok, shorts, facebook-reels, snapchat, youtube, linkedin, twitter, instagram, pinterest. |
| `--duration` | integer | Duration per slide in ms. |
| `--transition` | integer | Transition duration in ms. |
| `--ken-burns` | string | Ken Burns mode: zoom-in, zoom-out, pan-left, pan-right, pan-up, pan-down, alternating, off. |
| `--no-branding` | boolean | Disable branding end card (registered copies only; during the trial the end card is always added). |
| `--audio` | string | Background audio file path (MP3, WAV, M4A). |
| `--loop-audio` | boolean | Loop audio if shorter than video. |
| `--format` | string | Output format: mp4, webm. |
| `--quality` | integer | CRF quality (0-51, lower = better). |
| `--fps` | integer | Frames per second. |
| `--encoder` | string | Video encoder: auto, libx264, h264_nvenc, h264_qsv, h264_amf. |
| `--text-overlay` | string[] | Text overlay (format: text or slide:text:position). |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### contenta status

Show license status, tools availability, and version info.

```
contenta status
```

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

### contenta watch

Watch a directory and auto-convert new/changed images.

```
contenta watch <dir> [options]
```

| Argument | Description |
|---|---|
| `<dir>` | Directory to watch. |

| Option | Type | Description |
|---|---|---|
| `--include` | string[] | Only files matching this pattern (e.g. \*.jpg, or jpg;png); repeat the option or separate patterns with ';' or ','. |
| `--exclude` | string[] | Leave out files matching this pattern (as batch --exclude: a glob such as \*.png or thumb\_\*, or text in the path below the folder); repeat the option or separate patterns with ';' or ','. |
| `--debounce` | integer | Debounce delay in ms. |
| `--recursive` | boolean | Watch subdirectories. |
| `--output` | string | Output directory. |
| `--format` | string | Output format. |
| `--quality` | integer | Quality 1-100. |
| `--profile` | string | Load profile from JSON. |
| `--white-balance` | string | RAW white balance: camera, auto, manual. |
| `--color-temperature` | integer | Manual WB: temperature of the light the picture was taken in, in Kelvin (2000-10000, default 5500 = daylight; a lower value gives a cooler picture, a higher one a warmer picture). |
| `--denoising` | boolean | Apply wavelet denoising during RAW decode (off by default). |
| `--denoising-level` | integer | Denoising threshold (0-1000, default 300); giving a level turns denoising on. |
| `--sharpness-boost` | boolean | Sharpen after RAW decode (on by default for RAW input, as in the app; --sharpness-boost false turns it off; off by default with --raw-16bit). |
| `--color-boost` | boolean | Apply saturation boost after RAW decode. |
| `--contrast-boost` | boolean | Apply contrast boost after RAW decode. |
| `--exposure` | number | RAW exposure compensation in stops (EV), -3.0 to 3.0, default 0: +1 doubles the light, -1 halves it. |
| `--fast-raw-decode` | boolean | Faster RAW decode at half resolution when the output is at most half the size of the sensor image (slightly softer; for web and preview exports). |
| `--raw-16bit` | boolean | Decode RAW at 16 bits per channel and write a 16-bit TIFF. Needs -f tiff and no effects, RAW boosts, watermark or embedded preview (they work on 8 bits per channel); otherwise the file is written with 8 bits and the result carries a warning. Turns the default RAW sharpening off. |
| `--embedded-preview` | boolean | Convert the JPEG preview the camera stored in the RAW file instead of developing the RAW data: the fastest way, but the preview can be smaller than the full picture, already has the camera's look, and is 8-bit; white balance, denoising and exposure do not apply to it. A file without a preview is developed as usual. |

Output options: `--json`, `--plain`, `--quiet`, `--verbose`, `--progress`.

## Effects

Effect names for `--effects` and the MCP `effects` parameter, as `contenta effects` lists them.

| Effect | Category | What it does | Parameters |
|---|---|---|---|
| `blackwhite` | Color | Black & White | r: 0 to 100 (default 30), g: 0 to 100 (default 59), b: 0 to 100 (default 11) |
| `sepia` | Color | Sepia | none |
| `colorbrightnesscontrast` | Color | Brightness / Contrast | brightness: -100 to 100 (default 0), contrast: -100 to 100 (default 0), gamma: 0.1 to 5.0 (default 1.0), invert: true\|false |
| `lighten` | Color | Lighten | level: 1 to 100 (default 30) |
| `darken` | Color | Darken | level: 1 to 100 (default 30) |
| `contrast` | Color | Contrast Only (maps to fastc 'colors') | level: -100 to 100 (default 20) |
| `colortone` | Color | Color Tone | red: -100 to 100 (default 0), green: -100 to 100 (default 0), blue: -100 to 100 (default 0), saturation: 0 to 100 (default 50) |
| `huesaturation` | Color | Hue / Saturation | hue: -180 to 180 (default 0), saturation: -100 to 100 (default 0) |
| `autocolorlevel` | Color | Auto Color Level | none |
| `colorlevel` | Color | Color Levels | input_low: 0 to 255 (default 0), input_high: 0 to 255 (default 255), output_low: 0 to 255 (default 0), output_high: 0 to 255 (default 255), gamma: 0.1 to 5.0 (default 1.0), color_channel: 0 to 3 (default 0) |
| `colorbalance` | Color | Color Balance | preserve_lum: true\|false, tone_region: 0 to 2 (default 1), cyan_red: -100 to 100 (default 0), magenta_green: -100 to 100 (default 0), yellow_blue: -100 to 100 (default 0) |
| `colortemp` | Color | Color Temperature | temperature: -100 to 100 (default 0), 0 = unchanged, negative = cooler, positive = warmer |
| `invert` | Color | Invert Colors | none |
| `histogramequalization` | Color | Histogram Equalization | none |
| `saturationequalization` | Color | Saturation Equalization | none |
| `splash` | Color | Splash (Color Keep) | level: 0 to 100 (default 50) |
| `posterize` | Color | Posterize | level: 2 to 32 (default 6) |
| `threshold` | Color | Threshold | level: 0 to 255 (default 128) |
| `solarize` | Color | Solarize | level: 0 to 255 (default 128) |
| `applylut` | Color | Apply a 3D LUT (.cube, in-process) | path: a .cube file, strength: 0 to 100 (default 100) |
| `autoenhance` | Enhance | Auto-Enhance (in-process) | strength: 0 to 100 (default 50) |
| `clarity` | Enhance | Clarity (in-process) | amount: 0 to 100 (default 40) |
| `dehaze` | Enhance | Dehaze (in-process) | amount: 0 to 100 (default 50) |
| `aidenoise` | Enhance | AI Denoise (in-process) | strength: 0 to 100 (default 50) |
| `blur` | Blur / Sharpen | Blur | window: 1 to 30 (default 3) |
| `sharpen` | Blur / Sharpen | Sharpen | window: 1 to 30 (default 3) |
| `median` | Blur / Sharpen | Median Filter | window: 1 to 10 (default 2) |
| `refocus` | Blur / Sharpen | Refocus (Unsharp Mask) | window: 1 to 20 (default 5) |
| `motionblur` | Blur / Sharpen | Motion Blur | speed: 1 to 50 (default 10), angle: 0 to 360 (default 0) |
| `radialblur` | Blur / Sharpen | Radial Blur | level: 1 to 30 (default 5) |
| `zoomblur` | Blur / Sharpen | Zoom Blur | level: 1 to 30 (default 5) |
| `pencilsketch` | Artistic | Pencil Sketch | pensize: 1 to 20 (default 3), rough: 0 to 100 (default 50) |
| `oilpaint` | Artistic | Oil Paint | pensize: 1 to 20 (default 4), rough: 0 to 100 (default 50) |
| `emboss` | Artistic | Emboss | angle: 0 to 360 (default 45) |
| `oldphoto` | Artistic | Old Photo | window: 1 to 20 (default 3) |
| `glow` | Artistic | Glow | soft: 1 to 30 (default 5), brightness: -50 to 50 (default 10), contrast: -50 to 50 (default -10) |
| `softportrait` | Artistic | Soft Portrait | soft: 1 to 30 (default 5), brightness: -50 to 50 (default 5), warm: 0 to 30 (default 10) |
| `softedge` | Artistic | Soft Edge | soft: 1 to 30 (default 5) |
| `noisy` | Artistic | Add Noise | amount: 1 to 50 (default 10), iscolor: true\|false |
| `mosaic` | Distortion | Mosaic / Pixelate | size: 2 to 80 (default 10) |
| `shift` | Distortion | Shift | level: 1 to 50 (default 10) |
| `ribbon` | Distortion | Ribbon | amplitude: 1 to 30 (default 10), frequency: 1 to 20 (default 5) |
| `bulge` | Distortion | Bulge | level: -100 to 100 (default 40) |
| `twist` | Distortion | Twist | twist: -90 to 90 (default 30), size: 10 to 100 (default 50) |
| `illusion` | Distortion | Illusion | level: 1 to 20 (default 5) |
| `wave` | Distortion | Wave | wavelength: 10 to 200 (default 40), amplitude: 1 to 50 (default 10) |
| `ripple` | Distortion | Ripple | length: 10 to 200 (default 40), amount: 1 to 50 (default 10), type: sin\|square |
| `perspective` | Correction | Perspective Correction (in-process) | vertical: -100 to 100 (default 0), horizontal: -100 to 100 (default 0) |
| `crop` | Transforms | Crop image (in-process) | left: 0 to 100, top: 0 to 100, right: 0 to 100, bottom: 0 to 100 (percent of the picture trimmed from each edge, default 0); or x, y, width, height: pixels from the top-left corner. The crop cuts the picture first and a resize then sizes what is left: both forms count on the picture BEFORE the resize (after a flip or rotate placed before the crop), so the rectangle is in pixels of the source, not of the resized output |
| `flip` | Transforms | Flip (in-process) | direction: horizontal\|vertical (default horizontal) |
| `rotate` | Transforms | Rotate (in-process) | angle: 0 to 360 (default 90) |
| `watermark` | Transforms | Text/image watermark (in-process) | text, placement 0-8, transparency 0-100, font |
