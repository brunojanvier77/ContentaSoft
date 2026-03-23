# Contenta Converter — MCP Server

Contenta Converter exposes 11 image processing tools via the [Model Context Protocol](https://modelcontextprotocol.io/) (MCP). This enables AI agents (Claude Desktop, Cursor, Windsurf, Claude Code, etc.) to convert, resize, upscale, and manipulate images on your local machine.

## Getting Started

### 1. Install Contenta Converter

Download from [contenta-converter.com](https://contenta-converter.com). The `contenta` CLI is added to your PATH during installation.

### 2. Register (required for MCP)

The MCP server requires a registered license. A 30-day free trial is available.

```bash
contenta register --key YOUR-LICENSE-KEY
```

### 3. Configure Your AI Client

Add to your MCP client configuration:

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

For AI transform (optional), add your Gemini API key:

```json
{
  "mcpServers": {
    "contenta-converter": {
      "command": "contenta",
      "args": ["serve"],
      "env": {
        "GEMINI_API_KEY": "your-gemini-api-key"
      }
    }
  }
}
```

## Protocol Details

| Property | Value |
|----------|-------|
| Transport | stdio (stdin/stdout) |
| Protocol | JSON-RPC 2.0 |
| Protocol Version | `2024-11-05` |
| Server Name | `contenta-converter` |
| Server Version | `2026.1.0` |

## Tools

### convert_image

Convert a single image to a different format with optional resize, effects, and metadata.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `input_path` | string | Yes | Absolute path to the source image |
| `output_dir` | string | No | Output directory (default: same as input) |
| `format` | string | No | Output format: `jpg`, `png`, `webp`, `tiff`, `bmp`, `gif`, `jxl`, `heic`, `avif`, `svg`, `pdf` |
| `quality` | integer | No | Output quality 1-100 (default: 90) |
| `resize_width` | integer | No | Target width in pixels |
| `resize_height` | integer | No | Target height in pixels |
| `resize_mode` | string | No | `fit`, `fill`, `stretch`, `longest-edge`, `shortest-edge` |
| `copyright` | string | No | Copyright metadata to embed |
| `creator` | string | No | Creator metadata to embed |
| `preserve_metadata` | boolean | No | Preserve original metadata (default: true) |

---

### batch_convert

Convert multiple images at once. Provide file paths or a directory.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `input_paths` | string[] | No | Array of absolute file paths to convert |
| `input_dir` | string | No | Directory to scan for images (alternative to `input_paths`) |
| `output_dir` | string | No | Output directory |
| `format` | string | No | Output format (`jpg`, `png`, `webp`, etc.) |
| `quality` | integer | No | Output quality 1-100 |
| `resize_width` | integer | No | Target width |
| `resize_height` | integer | No | Target height |
| `workers` | integer | No | Parallel workers (default: CPU count) |

---

### detect_format

Detect the image format of a file (RAW, HEIC, JXL, PSD, PDF, etc.).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Absolute path to the image file |

**Returns**: format name, extension, file size, and boolean flags: `isRaw`, `isHeic`, `isJxl`, `isPsd`, `isPdf`, `isSvg`.

---

### read_metadata

Read EXIF, IPTC, and XMP metadata from an image file.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Absolute path to the image file |

**Returns**: file name, size, dimensions (width/height), last modified date, and full metadata object.

---

### list_effects

List all available image effects with their parameters. No parameters required.

**Returns**: 23 effects across 4 categories:

| Category | Effects |
|----------|---------|
| Color | brightness, contrast, lighten, darken, saturation, huesaturation, colortemp, colorbalance, autocolorlevel, histogramequalization, invert |
| Artistic | sepia, blackwhite, splash |
| Spatial | blur, sharpen, emboss, despeckle, deskew |
| Transform | crop, flip, rotate, watermark |

---

### upscale_image

AI upscale an image using Real-ESRGAN (2x or 4x).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `input_path` | string | Yes | Absolute path to the source image |
| `output_path` | string | No | Output file path (auto-generated if omitted) |
| `scale_factor` | integer | No | Scale factor: `2` or `4` (default: 2) |
| `model` | string | No | `realesrgan-x4plus` or `realesrgan-x4plus-anime` (default: realesrgan-x4plus) |
| `force_cpu` | boolean | No | Force CPU mode, no GPU (default: false) |

---

### apply_workflow

Apply a professional workflow (E-Commerce, Social, Real Estate, Wedding) to images. Each workflow generates platform-specific output variants.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `workflow_id` | string | Yes | `builtin.ecommerce`, `builtin.social`, `builtin.realestate`, `builtin.wedding` |
| `platforms` | string[] | No | Platform IDs to enable (e.g., `amazon`, `etsy`, `instagram`). Default: all platforms in workflow |
| `input_paths` | string[] | Yes | Absolute paths to source images |
| `output_dir` | string | Yes | Output directory |
| `workers` | integer | No | Parallel workers (default: CPU count) |

**Workflow platforms**:

- **E-Commerce**: `amazon` (2000x2000), `etsy`, `shopify`, `ebay`, `walmart`
- **Social Media**: `instagram`, `tiktok`, `youtube`, `pinterest`, `facebook`, `twitter`
- **Real Estate**: `mls` (1024x768), `zillow` (3000x2000), `website-hero` (1920x1080), `print-brochure`, `email`
- **Wedding**: `lab-prints`, `gallery`, `watermarked-proofs`, `social-media`

---

### create_pdf_album

Create a PDF photo album from multiple images with configurable layout.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `paths` | string[] | Yes | Absolute paths to images |
| `output_path` | string | Yes | Output PDF file path |
| `photos_per_page` | integer | No | 1, 2, 4, 6, 8, 12, 16, 24, or 48 (default: 4) |
| `page_size` | string | No | `A4`, `Letter`, `Legal` (default: A4) |
| `orientation` | string | No | `Auto`, `Portrait`, `Landscape` (default: Auto) |
| `embed_quality` | integer | No | JPEG embed quality 1-100 (default: 85) |

---

### write_metadata

Write EXIF/IPTC metadata to an existing image file without conversion.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Absolute path to the image file |
| `copyright` | string | No | Copyright notice |
| `creator` | string | No | Creator/photographer name |
| `title` | string | No | Image title |
| `description` | string | No | Image description/caption |
| `keywords` | string | No | Keywords (comma-separated) |
| `rights` | string | No | Rights/usage terms |
| `creator_city` | string | No | Creator city |
| `creator_country` | string | No | Creator country |
| `creator_email` | string | No | Creator email |
| `creator_url` | string | No | Creator URL/website |
| `latitude` | string | No | GPS latitude (decimal degrees) |
| `longitude` | string | No | GPS longitude (decimal degrees) |
| `altitude` | string | No | GPS altitude in meters |
| `datetime` | string | No | Date/time override (ISO 8601) |

---

### create_slideshow

Create a video slideshow from multiple images with Ken Burns motion, transitions, and optional audio.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `paths` | string[] | Yes | Array of image file paths (minimum 2) |
| `output_path` | string | Yes | Output video file path |
| `template` | string | No | `tiktok` (9:16), `shorts` (9:16), `facebook-reels` (9:16), `snapchat` (9:16), `youtube` (16:9), `linkedin` (16:9), `twitter` (16:9), `instagram` (1:1), `pinterest` (2:3). Default: `youtube` |
| `duration_ms` | integer | No | Duration per slide in milliseconds (default: 3000) |
| `transition_ms` | integer | No | Transition duration in milliseconds (default: 800) |
| `ken_burns` | string | No | `zoom-in`, `zoom-out`, `pan-left`, `pan-right`, `alternating`, `off`. Default: `zoom-in` |
| `format` | string | No | `mp4` or `webm` (default: mp4) |
| `quality` | integer | No | CRF quality 0-51 (default: 23, lower = better) |
| `audio_path` | string | No | Background audio file path |
| `add_branding` | boolean | No | Add branding end card (default: true) |

---

### ai_transform

AI-powered image transformation via Google Gemini (edit, remove background, enhance).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `input_path` | string | Yes | Absolute path to the source image |
| `prompt` | string | Yes | Transformation prompt (e.g., "Remove the background", "Make it brighter") |
| `api_key` | string | No | Google Gemini API key (or set `GEMINI_API_KEY` env var) |
| `output_path` | string | No | Output file path (default: `{name}_ai.{ext}` next to input) |
| `model` | string | No | Gemini model ID (default: `gemini-2.5-flash-image`) |
| `transparent_background` | boolean | No | Request transparent background (PNG output) |

## Example Conversations

### Resize product photos for Amazon

> **User**: I have 50 product photos in C:\Photos\products. Resize them all to 2000x2000 for Amazon.
>
> **Agent** calls `apply_workflow` with:
> - `workflow_id`: `builtin.ecommerce`
> - `platforms`: `["amazon"]`
> - `input_paths`: [list of 50 files]
> - `output_dir`: `C:\Photos\amazon-ready`

### Read and update photo metadata

> **User**: Check the metadata on sunset.jpg and add my copyright.
>
> **Agent** calls `read_metadata` with `path: "C:\Photos\sunset.jpg"`, reviews the output, then calls `write_metadata` with:
> - `path`: `C:\Photos\sunset.jpg`
> - `copyright`: `(c) 2026 John Smith`
> - `creator`: `John Smith`

### Create a social media slideshow

> **User**: Make an Instagram reel from these vacation photos with background music.
>
> **Agent** calls `create_slideshow` with:
> - `paths`: [list of image paths]
> - `output_path`: `C:\Videos\vacation-reel.mp4`
> - `template`: `instagram`
> - `audio_path`: `C:\Music\summer-vibes.mp3`
> - `ken_burns`: `alternating`
