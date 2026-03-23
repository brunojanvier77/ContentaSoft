# Contenta Converter PREMIUM

Professional batch image conversion, resizing, and processing for Windows.

[Download Free Trial](https://contenta-converter.com) | [MCP Server Docs](mcp-server.md)

## Features

- **50+ image formats** — JPG, PNG, WebP, HEIC, AVIF, JXL, TIFF, PSD, RAW (600+ cameras), PDF, SVG, BMP, GIF, and more
- **35+ effects** — brightness, contrast, blur, sharpen, sepia, watermark, crop, rotate, and more
- **AI upscaling** — Real-ESRGAN 2x/4x with GPU acceleration
- **Professional workflows** — one command for Amazon, Etsy, Instagram, MLS, and more
- **Batch processing** — thousands of files with parallel workers
- **PDF albums** — create photo albums with configurable layouts
- **Video slideshows** — Ken Burns motion for TikTok, YouTube, Instagram, etc.
- **Metadata control** — read/write EXIF, IPTC, XMP, GPS coordinates
- **AI transform** — Gemini-powered editing (remove background, enhance, etc.)

## CLI Reference

**Executable**: `contenta`

**Global options**: `--json` (JSON output), `--quiet` (suppress output), `-v`/`--verbose` (verbose logging)

| Command | Description |
|---------|-------------|
| `convert` | Convert a single image |
| `batch` | Batch convert a directory of images |
| `info` | Show image info and metadata |
| `effects` | List available effects |
| `formats` | List supported formats |
| `pdf-album` | Create a PDF photo album |
| `upscale` | AI upscale with Real-ESRGAN (2x/4x) |
| `slideshow` | Create a video slideshow |
| `ai-transform` | AI image transformation via Gemini |
| `workflows` | List or apply professional workflows |
| `watch` | Watch a folder for auto-conversion |
| `profile` | Show user/license info |
| `register` | Register a license key |
| `status` | Check trial/registration status |
| `serve` | Start the MCP server |

## CLI Examples

### Convert a single image

```bash
contenta convert photo.jpg --format webp --quality 85
```

### Batch convert an entire folder to PNG

```bash
contenta batch --input ./photos --output ./converted --format png --quality 90
```

### Resize for e-commerce (Amazon)

```bash
contenta workflows --id builtin.ecommerce --platforms amazon --input ./products --output ./amazon-ready
```

### AI upscale 4x

```bash
contenta upscale photo.jpg --scale 4
```

### Create a PDF album

```bash
contenta pdf-album --input ./vacation --output album.pdf --photos-per-page 6 --page-size A4
```

### Read image metadata (JSON)

```bash
contenta info photo.jpg --json
```

### Create a TikTok slideshow

```bash
contenta slideshow --input ./photos --output reel.mp4 --template tiktok --audio music.mp3
```

### Watch a folder for auto-conversion

```bash
contenta watch --input ./incoming --output ./processed --format webp --quality 80
```

## Supported Formats

### Input (50+)

**Standard**: JPG/JPEG, PNG, WebP, TIFF, BMP, GIF, ICO, TGA, PBM/PGM/PPM
**Modern**: HEIC/HEIF, AVIF, JXL (JPEG XL)
**Professional**: PSD, SVG, PDF, EPS
**RAW** (600+ cameras): CR2, CR3, NEF, ARW, RAF, DNG, ORF, RW2, PEF, SRW, X3F, MRW, ERF, MEF, NRW, 3FR, ARI, IIQ, KDC, MDC, and more

### Output

JPG, PNG, WebP, TIFF, BMP, GIF, HEIC, AVIF, JXL, SVG, PDF

## Professional Workflows

| Workflow | Platforms |
|----------|-----------|
| **E-Commerce** | Amazon (2000x2000), Etsy, Shopify, eBay, Walmart |
| **Social Media** | Instagram, TikTok, YouTube, Pinterest, Facebook, Twitter/X |
| **Real Estate** | MLS (1024x768), Zillow (3000x2000), Website Hero (1920x1080), Print Brochure, Email |
| **Wedding** | Lab Prints, Gallery, Watermarked Proofs, Social Media |

## MCP Server

Contenta Converter includes a built-in MCP server with 11 tools for AI agent integration. See the [full MCP documentation](mcp-server.md).

```bash
contenta serve
```
