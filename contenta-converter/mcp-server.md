# Contenta Converter: MCP server

`contenta serve` exposes 13 tools over the [Model Context Protocol](https://modelcontextprotocol.io/), so an AI agent (Claude Code, Claude Desktop, Cursor, Windsurf, Codex and others) can work on files on your PC.

Generated from `contenta catalog`, version 9.0.43. The same reference is on the website: [www.contenta-converter.com/docs/cli.html](https://www.contenta-converter.com/docs/cli.html). The command line is documented in [cli.md](cli.md).

## Setup

```json
{
  "mcpServers": {
    "contenta-converter": {
      "command": "contenta",
      "args": ["serve"]
    }
  }
}
```

If your AI client cannot find `contenta`, give the full path, for example `"C:\\Users\\<you>\\AppData\\Local\\Programs\\ContentaConverter\\contenta.exe"`. The [MCP config guide](../mcp-config/) covers each client. In Claude Code one command adds it:

```powershell
claude mcp add contenta-converter -- contenta serve
```

Pass absolute paths to every tool.

## Protocol

| Property | Value |
|---|---|
| Transport | stdio, newline-delimited JSON-RPC 2.0 |
| Protocol versions | `2025-11-25`, `2025-06-18`, `2025-03-26`, `2024-11-05`. The server answers with the version the client asks for; for any other version it answers `2025-11-25` |
| Capabilities | `tools` only (no resources or prompts) |
| Server name | `contenta-converter` |
| Server version | `9.0.43` |

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
| [`ai_transform`](#ai_transform) | AI transform image (Gemini) | no | yes | no | yes |
| [`batch_convert`](#batch_convert) | Batch convert images | no | no | no | no |
| [`convert_image`](#convert_image) | Convert image | no | no | no | no |
| [`create_pdf_album`](#create_pdf_album) | Create PDF album | no | yes | no | no |
| [`create_slideshow`](#create_slideshow) | Create video slideshow | no | yes | no | no |
| [`detect_format`](#detect_format) | Detect image format | yes | no | yes | no |
| [`get_file_info`](#get_file_info) | Get image file info | yes | no | yes | no |
| [`get_license_status`](#get_license_status) | Get license and trial status | yes | no | yes | no |
| [`list_effects`](#list_effects) | List effects | yes | no | yes | no |
| [`list_formats`](#list_formats) | List supported formats | yes | no | yes | no |
| [`merge_pdfs`](#merge_pdfs) | Merge PDFs | no | yes | no | no |
| [`read_metadata`](#read_metadata) | Read image metadata | yes | no | yes | no |
| [`write_metadata`](#write_metadata) | Write image metadata | no | yes | no | no |

Destructive: the tool can replace a file that already exists. Network: the tool uses the internet.

## Tools

### ai_transform

**AI transform image (Gemini).** AI-powered image transformation via Google Gemini (edit, remove background, enhance). During the trial, results beyond the free clean outputs carry the trial watermark.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_path` | string | yes | Absolute path to the source image. |
| `prompt` | string | no | Transformation prompt describing what to do (e.g., 'Make it brighter'). Required unless remove_background is true. |
| `api_key` | string | no | Google Gemini API key (or set GEMINI_API_KEY env var). |
| `output_path` | string | no | Output file path (default: {name}\_ai.{ext} next to input). |
| `model` | string | no | Model tier: fast (default), pro or lite; or an explicit Gemini model ID. |
| `size` | string | no | Output size: 1K (default), 2K or 4K (lite makes 1K only). One of: `1K`, `2K`, `4K`. |
| `transparent_background` | boolean | no | Request transparent background (PNG output). |
| `remove_background` | boolean | no | Remove the background; no prompt needed (the CLI's --remove-background). |
| `bg_mode` | string | no | With remove_background: white (default), transparent or custom. One of: `white`, `transparent`, `custom`. |
| `bg_color` | string | no | With bg_mode custom: the background colour as #RRGGBB. |

### batch_convert

**Batch convert images.** Convert multiple images at once. Provide file paths or a directory. A multi-page PDF or TIFF converts every page: one output file per page, or one multi-page file when the output format is PDF or TIFF (pdf_page picks a single page); total counts outputs, inputs counts the files given.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_paths` | array of string | no | Array of absolute file paths to convert. |
| `input_dir` | string | no | Directory to scan for images (alternative to input_paths). |
| `output_dir` | string | no | Output directory. |
| `overwrite` | boolean | no | Replace an output file that already exists (default: false, a new name is chosen instead). |
| `skip_existing` | boolean | no | Leave an input alone when its output already exists (reported as skipped), so a re-run only converts what is missing (default: false). |
| `format` | string | no | Output format (jpg, png, webp, etc.) |
| `quality` | integer | no | Output quality. Range: 1 to 100. |
| `resize_width` | integer | no | Target width (alone: the height follows the aspect ratio). Minimum: 1. |
| `resize_height` | integer | no | Target height (alone: the width follows the aspect ratio). Minimum: 1. |
| `resize_mode` | string | no | Resize mode: fit (default), fill, stretch, longest-edge, shortest-edge, fit-with-background, crop-to-aspect. fill, stretch, fit-with-background and crop-to-aspect need both resize_width and resize_height. |
| `background` | string | no | Colour around the picture with resize_mode fit-with-background: white, black, grey, lightgrey or #RRGGBB (default: white). resize_width 2000 + resize_height 2000 + resize_mode fit-with-background + background white = white square product images. |
| `effects` | array of string | no | Effects to apply in order, each name or name:key=value (e.g. \["sepia", "sharpen:window=5"\]; a comma list "sepia,sharpen" works too). Names and parameters: list_effects. A crop cuts the picture BEFORE the resize (its x, y, width, height are pixels of the source; left, top, right, bottom are percent): the resize then sizes what is left. An unknown name fails the call before anything is converted. |
| `lut_path` | string | no | Absolute path to a 3D LUT (.cube) applied to every image, after resize and the other effects (the CLI's --lut). A file that is missing or not a 3D .cube LUT fails the call before anything is converted. |
| `lut_strength` | integer | no | How much of the LUT to apply, 0-100 (default 100). Needs lut_path. Range: 0 to 100. |
| `watermark_text` | string | no | Text watermark drawn on every output in white, its width 10% of the image (the CLI's --watermark). Nothing is drawn without it. |
| `watermark_position` | integer | no | Where the watermark goes, 0-8 read left to right, top to bottom: 0 top-left, 1 top-centre, 2 top-right, 3 middle-left, 4 centre, 5 middle-right, 6 bottom-left, 7 bottom-centre, 8 bottom-right (default: 8). Range: 0 to 8. |
| `watermark_opacity` | integer | no | Watermark opacity 0-100, 100 = fully visible (default: 70). Range: 0 to 100. |
| `lens_corrections` | boolean | no | Apply the lens corrections a DNG file carries: distortion, vignetting, shading (default: true). No effect on other inputs. |
| `pdf_page` | integer | no | Multi-page PDF or TIFF: convert only this page (0-based: 0 is the first page). Omit it to convert EVERY page: one output file each (name_page001.jpg, name_page002.jpg, ...), or one multi-page file when the output format is pdf or tiff; the result lists every file in outputs. Minimum: 0. |
| `white_balance` | string | no | RAW input only. RAW white balance: camera, auto, manual (default: camera). One of: `camera`, `auto`, `manual`. |
| `color_temperature` | integer | no | RAW input, white_balance manual. Manual WB: temperature of the light the picture was taken in, in Kelvin (2000-10000, default 5500 = daylight; a lower value gives a cooler picture, a higher one a warmer picture). Range: 2000 to 10000. |
| `denoising` | boolean | no | RAW input only. Apply wavelet denoising during RAW decode (off by default). |
| `denoising_level` | integer | no | RAW input only. Denoising threshold (0-1000, default 300); giving a level turns denoising on. Range: 0 to 1000. |
| `sharpness_boost` | boolean | no | RAW input only. Sharpen after RAW decode (on by default for RAW input, as in the app; false turns it off; off by default with raw_16bit). |
| `color_boost` | boolean | no | RAW input only. Apply saturation boost after RAW decode. |
| `contrast_boost` | boolean | no | RAW input only. Apply contrast boost after RAW decode. |
| `exposure` | number | no | RAW input only. RAW exposure compensation in stops (EV), -3.0 to 3.0, default 0: +1 doubles the light, -1 halves it. Range: -3 to 3. |
| `fast_raw_decode` | boolean | no | RAW input only. Faster RAW decode at half resolution when the output is at most half the size of the sensor image (slightly softer; for web and preview exports). |
| `raw_16bit` | boolean | no | RAW input only. Decode RAW at 16 bits per channel and write a 16-bit TIFF. Needs format tiff and no effects, RAW boosts, watermark or embedded preview (they work on 8 bits per channel); otherwise the file is written with 8 bits and the result carries a warning. Turns the default RAW sharpening off; a file written with 8 bits instead is named in the result's warnings. |
| `embedded_preview` | boolean | no | RAW input only. Convert the JPEG preview the camera stored in the RAW file instead of developing the RAW data: the fastest way, but the preview can be smaller than the full picture, already has the camera's look, and is 8-bit; white balance, denoising and exposure do not apply to it. A file without a preview is developed as usual (default: false). |
| `workers` | integer | no | Parallel workers (default: automatic - one per logical CPU, 2x for JPEG downscales to half size or less, one per physical core for RAW batches on 8+ core CPUs). Minimum: 1. |

### convert_image

**Convert image.** Convert a single image to a different format with optional resize, effects, and metadata. A multi-page PDF or TIFF converts every page: one output file per page, or one multi-page file when the output format is PDF or TIFF (pdf_page picks a single page); the result lists every file in outputs.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_path` | string | yes | Absolute path to the source image. |
| `output_dir` | string | no | Output directory (default: same as input). |
| `overwrite` | boolean | no | Replace an output file that already exists (default: false, a new name is chosen instead). |
| `skip_existing` | boolean | no | Leave an input alone when its output already exists (reported as skipped), so a re-run only converts what is missing (default: false). |
| `format` | string | no | Output format: jpg, png, webp, tiff, bmp, gif, jxl, heic, avif, svg, pdf. |
| `quality` | integer | no | Output quality (default: 90). Range: 1 to 100. |
| `resize_width` | integer | no | Target width in pixels (alone: the height follows the aspect ratio). Minimum: 1. |
| `resize_height` | integer | no | Target height in pixels (alone: the width follows the aspect ratio). Minimum: 1. |
| `resize_mode` | string | no | Resize mode: fit (default), fill, stretch, longest-edge, shortest-edge, fit-with-background, crop-to-aspect. fill, stretch, fit-with-background and crop-to-aspect need both resize_width and resize_height. |
| `background` | string | no | Colour around the picture with resize_mode fit-with-background: white, black, grey, lightgrey or #RRGGBB (default: white). resize_width 2000 + resize_height 2000 + resize_mode fit-with-background + background white = white square product images. |
| `effects` | array of string | no | Effects to apply in order, each name or name:key=value (e.g. \["sepia", "sharpen:window=5"\]; a comma list "sepia,sharpen" works too). Names and parameters: list_effects. A crop cuts the picture BEFORE the resize (its x, y, width, height are pixels of the source; left, top, right, bottom are percent): the resize then sizes what is left. An unknown name fails the call before anything is converted. |
| `lut_path` | string | no | Absolute path to a 3D LUT (.cube) applied to every image, after resize and the other effects (the CLI's --lut). A file that is missing or not a 3D .cube LUT fails the call before anything is converted. |
| `lut_strength` | integer | no | How much of the LUT to apply, 0-100 (default 100). Needs lut_path. Range: 0 to 100. |
| `watermark_text` | string | no | Text watermark drawn on every output in white, its width 10% of the image (the CLI's --watermark). Nothing is drawn without it. |
| `watermark_position` | integer | no | Where the watermark goes, 0-8 read left to right, top to bottom: 0 top-left, 1 top-centre, 2 top-right, 3 middle-left, 4 centre, 5 middle-right, 6 bottom-left, 7 bottom-centre, 8 bottom-right (default: 8). Range: 0 to 8. |
| `watermark_opacity` | integer | no | Watermark opacity 0-100, 100 = fully visible (default: 70). Range: 0 to 100. |
| `copyright` | string | no | Copyright metadata to embed. |
| `creator` | string | no | Creator metadata to embed. |
| `preserve_metadata` | boolean | no | Preserve original metadata (default: true). |
| `white_balance` | string | no | RAW input only. RAW white balance: camera, auto, manual (default: camera). One of: `camera`, `auto`, `manual`. |
| `color_temperature` | integer | no | RAW input, white_balance manual. Manual WB: temperature of the light the picture was taken in, in Kelvin (2000-10000, default 5500 = daylight; a lower value gives a cooler picture, a higher one a warmer picture). Range: 2000 to 10000. |
| `denoising` | boolean | no | RAW input only. Apply wavelet denoising during RAW decode (off by default). |
| `denoising_level` | integer | no | RAW input only. Denoising threshold (0-1000, default 300); giving a level turns denoising on. Range: 0 to 1000. |
| `sharpness_boost` | boolean | no | RAW input only. Sharpen after RAW decode (on by default for RAW input, as in the app; false turns it off; off by default with raw_16bit). |
| `color_boost` | boolean | no | RAW input only. Apply saturation boost after RAW decode. |
| `contrast_boost` | boolean | no | RAW input only. Apply contrast boost after RAW decode. |
| `exposure` | number | no | RAW input only. RAW exposure compensation in stops (EV), -3.0 to 3.0, default 0: +1 doubles the light, -1 halves it. Range: -3 to 3. |
| `fast_raw_decode` | boolean | no | RAW input only. Faster RAW decode at half resolution when the output is at most half the size of the sensor image (slightly softer; for web and preview exports). |
| `raw_16bit` | boolean | no | RAW input only. Decode RAW at 16 bits per channel and write a 16-bit TIFF. Needs format tiff and no effects, RAW boosts, watermark or embedded preview (they work on 8 bits per channel); otherwise the file is written with 8 bits and the result carries a warning. Turns the default RAW sharpening off; a file written with 8 bits instead is named in the result's warnings. |
| `embedded_preview` | boolean | no | RAW input only. Convert the JPEG preview the camera stored in the RAW file instead of developing the RAW data: the fastest way, but the preview can be smaller than the full picture, already has the camera's look, and is 8-bit; white balance, denoising and exposure do not apply to it. A file without a preview is developed as usual (default: false). |
| `lens_corrections` | boolean | no | Apply the lens corrections a DNG file carries: distortion, vignetting, shading (default: true). No effect on other inputs. |
| `pdf_page` | integer | no | Multi-page PDF or TIFF: convert only this page (0-based: 0 is the first page). Omit it to convert EVERY page: one output file each (name_page001.jpg, name_page002.jpg, ...), or one multi-page file when the output format is pdf or tiff; the result lists every file in outputs. Minimum: 0. |

### create_pdf_album

**Create PDF album.** Create a PDF photo album from multiple images with configurable layout. During the trial the PDF always carries the trial watermark.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_paths` | array of string | yes | Required. Absolute paths to images. |
| `output_path` | string | yes | Output PDF file path. |
| `photos_per_page` | integer | no | Photos per page: 1, 2, 4, 6, 8, 10, 12, 16, 24, 48 (default: 4). |
| `page_size` | string | no | Page size: A4, Letter, Legal (default: A4). |
| `orientation` | string | no | Orientation: Auto, Portrait, Landscape (default: Auto). |
| `embed_quality` | integer | no | JPEG quality for non-JPEG inputs; JPEG inputs are embedded unchanged (default: 85). Range: 1 to 100. |

### create_slideshow

**Create video slideshow.** Create a video slideshow from multiple images with Ken Burns motion, transitions, and optional audio. During the trial the video always ends with the branding end card.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_paths` | array of string | yes | Required. Array of image file paths (minimum 2). |
| `output_path` | string | yes | Output video file path. |
| `template` | string | no | Video template: tiktok (9:16), shorts (9:16), facebook-reels (9:16), snapchat (9:16), youtube (16:9), linkedin (16:9), twitter (16:9), instagram (1:1), pinterest (2:3). Default: youtube. |
| `duration_ms` | integer | no | Duration per slide in milliseconds (default: 3000). |
| `transition_ms` | integer | no | Transition duration in milliseconds (default: 800). |
| `ken_burns` | string | no | Ken Burns mode: zoom-in, zoom-out, pan-left, pan-right, alternating, off. Default: zoom-in. |
| `format` | string | no | Output format: mp4, webm (default: mp4). |
| `quality` | integer | no | CRF quality (default: 23, lower = better). Range: 0 to 51. |
| `audio_path` | string | no | Background audio file path. |
| `add_branding` | boolean | no | Add branding end card (default: true). Registered copies only: during the trial the end card is always added and false is ignored. |

### detect_format

**Detect image format.** Detect the image format of a file (RAW, HEIC, JXL, PSD, PDF, etc.)

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_path` | string | yes | Required. Absolute path to the image file. |

### get_file_info

**Get image file info.** Inspect an image file (the CLI's `contenta info`): real format, pixel size, DPI, file size, copyright and creator.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_path` | string | yes | Absolute path to the image file. |

### get_license_status

**Get license and trial status.** Get the license and trial state before converting: registered or trial, clean outputs left (freeFilesRemaining), and what outputs become after the clean ones (afterFreeFiles: watermark). nextOutput says what the next output will be: clean or watermark.

No parameters.

### list_effects

**List effects.** List all available image effects with their parameters.

No parameters.

### list_formats

**List supported formats.** List the image formats Contenta Converter reads (extension, category, decoder) and writes (the CLI's `contenta formats`).

| Parameter | Type | Required | Description |
|---|---|---|---|
| `kind` | string | no | Which list: input (formats read), output (formats written) or all (default). One of: `all`, `input`, `output`. |

### merge_pdfs

**Merge PDFs.** Append multiple PDF files into a single PDF, in list order (page-level merge, no rasterization). During the trial the PDF always carries the trial watermark.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_paths` | array of string | yes | Required. Absolute paths to PDF files, appended in this order. |
| `output_path` | string | yes | Output PDF file path. |

### read_metadata

**Read image metadata.** Read EXIF, IPTC, and XMP metadata from an image file.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_path` | string | yes | Required. Absolute path to the image file. |

### write_metadata

**Write image metadata.** Write EXIF/IPTC/XMP metadata to an existing image file without conversion. Every field goes to XMP too, so WebP, HEIC and AVIF (no IPTC block) keep keywords, city and country. The file is read back: written lists the fields now in it, dropped any that are not, with the reason.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_path` | string | yes | Required. Absolute path to the image file. |
| `copyright` | string | no | Copyright notice. |
| `creator` | string | no | Creator/photographer name. |
| `title` | string | no | Image title. |
| `description` | string | no | Image description/caption. |
| `keywords` | string | no | Keywords (comma-separated). |
| `rights` | string | no | Rights/usage terms. |
| `creator_city` | string | no | Creator city. |
| `creator_country` | string | no | Creator country. |
| `creator_email` | string | no | Creator email. |
| `creator_url` | string | no | Creator URL/website. |
| `latitude` | string | no | GPS latitude (decimal degrees). |
| `longitude` | string | no | GPS longitude (decimal degrees). |
| `altitude` | string | no | GPS altitude in meters. |
| `datetime` | string | no | Date/time override (ISO 8601). |
