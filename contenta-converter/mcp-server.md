# Contenta Converter: MCP server

`contenta serve` exposes 10 image tools over the [Model Context Protocol](https://modelcontextprotocol.io/), so an AI agent (Claude Desktop, Claude Code, Cursor, Windsurf and others) can convert, resize, tag and package images on your PC.

## Setup

1. Install Contenta Converter from [contenta-converter.com](https://www.contenta-converter.com). The `contenta` CLI and the MCP server come with it; the default per-user install is `%LOCALAPPDATA%\Programs\ContentaConverter\`, added to your user `PATH`.
2. Add the server to your AI client:

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

If the client cannot find `contenta`, use the full path, for example `"C:\\Users\\<you>\\AppData\\Local\\Programs\\ContentaConverter\\contenta.exe"`. See the [MCP config guide](../mcp-config/) for each client.

For `ai_transform`, give the server your Google Gemini API key:

```json
{
  "mcpServers": {
    "contenta-converter": {
      "command": "contenta",
      "args": ["serve"],
      "env": { "GEMINI_API_KEY": "<your key>" }
    }
  }
}
```

Paths can be absolute or relative. A relative path is resolved against the server's working folder, which is the folder your AI client started it in; use absolute paths when you do not know that folder.

## Protocol

| Property | Value |
|----------|-------|
| Transport | stdio |
| Protocol | JSON-RPC 2.0, MCP `2024-11-05` |
| Server name | `contenta-converter` |
| Server version | `9.0.34` |

**License**: the server runs during the 30-day trial and for registered copies; after the trial it refuses to start (exit code 3). During the trial the first 10 images are clean and later output carries a watermark. PDF albums, merged PDFs and slideshows are always marked during the trial. Register with `contenta register <email> <key>`.

## Tools

### convert_image

Convert one image, with optional resize and metadata.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `input_path` | string | Yes | Path to the source image |
| `output_dir` | string | No | Output folder (default: next to the input) |
| `format` | string | No | `jpg`, `png`, `webp`, `tiff`, `bmp`, `gif`, `jxl`, `heic`, `avif`, `svg`, `pdf` |
| `quality` | integer | No | 1-100 (default 90) |
| `resize_width` | integer | No | Target width in pixels |
| `resize_height` | integer | No | Target height in pixels |
| `resize_mode` | string | No | `fit`, `fill`, `stretch`, `longest-edge`, `shortest-edge` |
| `effects` | string[] | No | Effects applied in order, each `name` or `name:key=value,...`, e.g. `["sepia", "sharpen:window=5"]` |
| `copyright` | string | No | Copyright metadata |
| `creator` | string | No | Creator metadata |
| `preserve_metadata` | boolean | No | Keep the original metadata (default true) |

**Returns**: success, input, outputPath, outputSize, durationMs.

---

### batch_convert

Convert a list of files or a whole folder.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `input_paths` | string[] | No | File paths |
| `input_dir` | string | No | Folder to scan (instead of `input_paths`) |
| `output_dir` | string | No | Output folder |
| `format` | string | No | Output format |
| `quality` | integer | No | 1-100 |
| `resize_width` | integer | No | Target width |
| `resize_height` | integer | No | Target height |
| `effects` | string[] | No | Effects applied to every file, same syntax as `convert_image` |
| `workers` | integer | No | Parallel workers (default: automatic, based on your CPU and the job) |

**Returns**: total, success, skipped, failed, outputs, errors.

**Effects**: names and parameters come from `list_effects`. A comma list in one string (`"sepia,sharpen"`) also works. An unknown effect name or parameter fails the call before anything is converted, and the error lists the valid names. Parameters you leave out take the desktop app's defaults, so `"blackwhite"` alone works. `flip` takes `direction=horizontal|vertical`; `crop` takes `x`, `y`, `width` and `height` in pixels.

---

### detect_format

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Path to the file |

**Returns**: path, format (the decoder that will read it, e.g. `RawLibRaw`), extension, size, and the flags isRaw, isHeic, isJxl, isPsd, isPdf, isSvg.

---

### read_metadata

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Path to the image |

**Returns**: path, fileName, format (e.g. `JPEG`, `WebP`, `HEIC`), decoder, extension, size, width, height, dpiX, dpiY, copyright, creator, lastModified, and a metadata object with the dates and the EXIF (camera, exposure, GPS, orientation), IPTC and XMP fields, including keywords and the creator's email and URL. Fields the file does not have are left out.

In 9.0.34, `read_metadata` does not read AVIF files yet: it reports format `QuickTime` with no size or fields. Metadata written to AVIF is in the file, and other readers such as ExifTool show it.

---

### write_metadata

Write metadata into an existing image without converting it.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Path to the image |
| `copyright`, `creator`, `title`, `description`, `keywords`, `rights` | string | No | Text fields (`keywords` comma-separated) |
| `creator_city`, `creator_country`, `creator_email`, `creator_url` | string | No | Creator contact fields |
| `latitude`, `longitude`, `altitude` | string | No | GPS, decimal degrees and meters |
| `datetime` | string | No | Date/time override (ISO 8601) |

The file is read back after writing. Every field also goes to XMP, so WebP, HEIC and AVIF files, which have no IPTC block, keep keywords, city, country, email and URL.

**Returns**: success, path, fieldsWritten, `written` (the fields now in the file), `dropped` (fields that did not stick, with the reason), `notes` (e.g. that a WebP file got XMP instead of IPTC) and `verified`.

---

### list_effects

No parameters. **Returns** an array of 51 effects, each with name, category, description and parameters (ranges and defaults). Categories: Color (19), Enhance (4), Blur / Sharpen (7), Artistic (8), Distortion (8), Correction (1), Transforms (4).

---

### create_pdf_album

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `paths` | string[] | Yes | Image paths |
| `output_path` | string | Yes | Output PDF path |
| `photos_per_page` | integer | No | 1, 2, 4, 6, 8, 10, 12, 16, 24 or 48 (default 4) |
| `page_size` | string | No | `A4`, `Letter`, `Legal` (default A4) |
| `orientation` | string | No | `Auto`, `Portrait`, `Landscape` (default Auto) |
| `embed_quality` | integer | No | JPEG quality for non-JPEG inputs (default 85); JPEG inputs are embedded unchanged |

**Returns**: success, outputPath, images, pages.

---

### merge_pdfs

Append PDF files into one PDF, in list order. Pages are copied, not re-rendered.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `paths` | string[] | Yes | PDF paths, in the order to append |
| `output_path` | string | Yes | Output PDF path |

**Returns**: success, outputPath, files.

---

### create_slideshow

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `paths` | string[] | Yes | Image paths (at least 2) |
| `output_path` | string | Yes | Output video path |
| `template` | string | No | `tiktok`, `shorts`, `facebook-reels`, `snapchat` (9:16), `youtube`, `linkedin`, `twitter` (16:9), `instagram` (1:1), `pinterest` (2:3). Default `youtube` |
| `duration_ms` | integer | No | Time per slide (default 3000) |
| `transition_ms` | integer | No | Transition length (default 800) |
| `ken_burns` | string | No | `zoom-in`, `zoom-out`, `pan-left`, `pan-right`, `alternating`, `off` (default `zoom-in`) |
| `format` | string | No | `mp4` or `webm` (default mp4) |
| `quality` | integer | No | CRF 0-51, lower = better (default 23) |
| `audio_path` | string | No | Background audio file |
| `add_branding` | boolean | No | End card (default true). Only a registered copy can turn it off |

**Returns**: success, outputPath, images, template, format.

---

### ai_transform

Edit an image with a Google Gemini prompt. Needs a Gemini API key (`api_key`, or `GEMINI_API_KEY` in the server's environment); each call is billed to that key by Google.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `input_path` | string | Yes | Path to the source image |
| `prompt` | string | Yes | What to do, e.g. "Remove the background" |
| `api_key` | string | No | Gemini API key |
| `output_path` | string | No | Output path (default `{name}_ai.{ext}` next to the input) |
| `model` | string | No | Gemini model ID (default `gemini-3-pro-image-preview`) |
| `transparent_background` | boolean | No | Ask for a transparent background (PNG output) |

If the model answers without an image, the tool returns an error saying so.

## Example requests

> "Resize the product photos in C:\Photos\products to 2000x2000 JPEGs for Amazon."
>
> The agent calls `batch_convert` with `input_dir: "C:\\Photos\\products"`, `output_dir: "C:\\Photos\\amazon-ready"`, `format: "jpg"`, `resize_width: 2000`, `resize_height: 2000`.

> "Add my copyright to sunset.jpg."
>
> The agent calls `write_metadata` with `path`, `copyright: "(c) 2026 Your Name"`, `creator: "Your Name"`. The reply lists `written: ["copyright", "creator"]` and `verified: true`.

> "Make black-and-white 1000 px versions of the photos in C:\Photos\shoot."
>
> The agent calls `batch_convert` with `input_dir`, `output_dir`, `resize_width: 1000`, `resize_height: 1000`, `effects: ["blackwhite"]`.

> "Put the three scanned invoices into one PDF."
>
> The agent calls `merge_pdfs` with the three paths in order and an `output_path`.
