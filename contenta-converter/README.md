# Contenta Converter PREMIUM

Professional batch image conversion, resizing, and processing for Windows.

[Download Free Trial](https://contenta-converter.com) | [MCP Server Docs](mcp-server.md)

## Features

- **50+ image formats** — JPG, PNG, WebP, HEIC, AVIF, JXL, TIFF, PSD, RAW (600+ cameras), PDF, SVG, BMP, GIF, and more
- **30+ effects** — brightness, contrast, blur, sharpen, sepia, color balance, and more (`contenta effects` for the full list)
- **AI upscaling** — Real-ESRGAN 2x/4x with GPU acceleration
- **Batch processing** — thousands of files with parallel workers
- **Multi-size export** — favicon/app-icon/srcset sets in one pass (`--sizes`, `--size-preset`)
- **PDF albums** — create photo albums with configurable layouts
- **Video slideshows** — Ken Burns motion for TikTok, YouTube, Instagram, etc.
- **Metadata control** — read/write EXIF, IPTC, XMP, GPS coordinates
- **AI transform** — Gemini-powered editing (remove background, enhance, etc.)

## Install & PATH

The CLI (`contenta.exe`) is installed together with the desktop app — same installer, no separate download.

- Default install location: `C:\Program Files\ContentaSoft\Contenta Converter PREMIUM\`
- Add that directory to your `PATH`, or call the exe with its full path.
- Verify it works:

```bash
contenta status      # license state, bundled tools health, version
contenta --version
```

## CLI Reference

**Executable**: `contenta`

**Global options** (all commands): `--json` (structured JSON output), `--quiet` (suppress non-error output), `-v`/`--verbose` (verbose logging)

| Command | Description |
|---------|-------------|
| `convert <input>` | Convert a single image |
| `batch <input-dir>` | Batch convert a directory of images |
| `info <file>` | Show format detection and metadata for an image |
| `effects` | List all available effects with parameters |
| `formats` | List supported formats (`--input` / `--output` to filter) |
| `pdf-album <files>...` | Create a PDF photo album |
| `upscale <input>` | AI upscale with Real-ESRGAN (2x/4x) |
| `slideshow <files>...` | Create a video slideshow |
| `ai-transform <input>` | AI image transformation via Google Gemini |
| `watch <dir>` | Watch a folder and auto-convert new/changed images |
| `profile save\|load\|validate <file>` | Save, load, or validate conversion profiles (JSON) |
| `register <email> <key>` | Register a license key |
| `status` | Show license status, bundled tools health, and version |
| `serve` | Start the MCP server (stdio) |

`convert`, `batch`, `pdf-album`, `slideshow`, and `watch` take their input as a **positional argument** — there is no `--input` flag.

## Quick Start

```bash
# Convert a single image
contenta convert photo.jpg --format webp --quality 85

# Convert a RAW photo (600+ cameras supported out of the box)
contenta convert photo.cr2 --format jpg --quality 92

# Batch convert an entire folder to PNG
contenta batch ./photos --output ./converted --format png --quality 90

# Resize product photos for Amazon (2000x2000)
contenta batch ./products --output ./amazon-ready --format jpg --resize 2000x2000 --resize-mode fit

# AI upscale 4x
contenta upscale photo.jpg --scale 4

# Create a PDF album (6 photos per A4 page)
contenta pdf-album ./vacation/*.jpg --output album.pdf --photos-per-page 6 --page-size A4

# Read image metadata as JSON
contenta info photo.jpg --json

# Create a TikTok slideshow with music
contenta slideshow ./photos/*.jpg --output reel.mp4 --template tiktok --audio music.mp3

# Watch a folder for auto-conversion
contenta watch ./incoming --output ./processed --format webp --quality 80
```

## Platform Size Cheat Sheet

Use `--resize WxH --resize-mode fit` with these dimensions:

| Platform | Dimensions | Platform | Dimensions |
|----------|-----------|----------|-----------|
| Amazon | 2000x2000 | Instagram | 1080x1080 |
| Etsy | 2700x2025 | TikTok | 1080x1920 |
| Shopify | 2048x2048 | YouTube thumbnail | 1280x720 |
| eBay | 1600x1600 | Pinterest | 1000x1500 |
| Walmart | 2000x2000 | Facebook | 1200x630 |
| MLS (real estate) | 1024x768 | Twitter/X | 1600x900 |
| Zillow | 3000x2000 | Website hero | 1920x1080 |

## Supported Formats

### Input (50+)

**Standard**: JPG/JPEG, PNG, WebP, TIFF, BMP, GIF, ICO, TGA, PBM/PGM/PPM
**Modern**: HEIC/HEIF, AVIF, JXL (JPEG XL)
**Professional**: PSD, SVG, PDF, EPS
**RAW** (600+ cameras): CR2, CR3, NEF, ARW, RAF, DNG, ORF, RW2, PEF, SRW, X3F, MRW, ERF, MEF, NRW, 3FR, ARI, IIQ, KDC, MDC, and more

### Output

JPG, PNG, WebP, TIFF, BMP, GIF, HEIC, AVIF, JXL, SVG, PDF, ICO

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Invalid arguments |
| 3 | License required (trial expired) — register with `contenta register <email> <key>` |
| 4 | Conversion failed |
| 5 | File not found |
| 6 | Bundled tools missing — run `contenta status` to see which tool is unhealthy |

## Trial & Licensing

- 30-day free trial starts on first launch — no credit card, no account.
- During the trial, the first **5 images** convert clean; after that, output gets a small watermark until you register.
- `contenta register <email> <key>` removes all trial limits permanently (one-time purchase, no subscription).
- If a command exits with code 3, the trial has expired.

## Troubleshooting

- **`contenta` is not recognized** — the install directory is not on `PATH`. Use the full path (`"C:\Program Files\ContentaSoft\Contenta Converter PREMIUM\contenta.exe"`) or add it to `PATH`.
- **Exit code 6 / tool errors** — run `contenta status`; it lists every bundled tool (fastc, exiftool, ffmpeg, Real-ESRGAN, …) with an OK/missing indicator. Reinstalling restores missing tools.
- **Scripting** — add `--json` to any command for machine-readable output and check the exit code.

## MCP Server

Contenta Converter includes a built-in MCP server with 10 tools for AI agent integration. It works during the free trial. See the [full MCP documentation](mcp-server.md).

```bash
contenta serve
```
