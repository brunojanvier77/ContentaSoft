---
name: contenta-image-processing
description: Convert, resize, upscale, and process images using the Contenta Converter CLI. Use when the user asks to convert image formats, resize photos for e-commerce/social media, apply effects, AI upscale, create PDF albums, video slideshows, or read/write image metadata.
allowed-tools: Bash
---

# Contenta Image Processing

You have access to the `contenta` CLI for professional image processing. All commands support `--json` for structured output. Inputs are **positional arguments** (there is no `--input` flag). If `contenta` is not on PATH, use the full path: `"C:\Program Files\ContentaSoft\Contenta Converter PREMIUM\contenta.exe"`.

## Commands

### Convert a single image
```bash
contenta convert <input> --format <fmt> --quality <1-100> --resize <WxH> --resize-mode <fit|fill|stretch|longest-edge|shortest-edge>
```
Formats: jpg, png, webp, tiff, bmp, gif, jxl, heic, avif, svg, pdf, ico

### Batch convert
```bash
contenta batch <input-dir> --output <dir> --format <fmt> --quality <q> --workers <n> [--recursive] [--include "*.jpg"]
```

### Resize for a specific platform
Use `--resize` with the platform's dimensions and `--resize-mode fit`:

- E-Commerce: amazon 2000x2000, etsy 2700x2025, shopify 2048x2048, ebay 1600x1600, walmart 2000x2000
- Social: instagram 1080x1080, tiktok 1080x1920, youtube 1280x720, pinterest 1000x1500, facebook 1200x630, twitter 1600x900
- Real Estate: mls 1024x768, zillow 3000x2000, website-hero 1920x1080, print-brochure 2400x3000, email 800x600
- Wedding: lab-prints 3600x2400, gallery 2048x1365, watermarked-proofs 1200x800

```bash
contenta batch ./products --output ./amazon-ready --format jpg --resize 2000x2000 --resize-mode fit
```

### AI upscale
```bash
contenta upscale <input> --scale <2|4> --model <realesrgan-x4plus|realesrgan-x4plus-anime> [--force-cpu]
```

### Multi-size export (icons, srcset)
```bash
contenta convert logo.png --sizes 16,32,48,64 --format ico
contenta convert logo.png --size-preset favicon
```

### Create PDF album
```bash
contenta pdf-album <files>... --output <file.pdf> --photos-per-page <1|2|4|6|8|12|16|24|48> --page-size <A4|Letter|Legal>
```

### Create video slideshow
```bash
contenta slideshow <files>... --output <file.mp4> --template <template> --audio <file> --ken-burns <mode>
```
Templates: tiktok, shorts, facebook-reels, snapchat, youtube, linkedin, twitter, instagram, pinterest
Ken Burns modes: zoom-in, zoom-out, pan-left, pan-right, pan-up, pan-down, alternating, off

### Read metadata
```bash
contenta info <file> --json
```

### Write metadata (during conversion)
```bash
contenta convert <file> --copyright "text" --creator "name" --keywords "a,b,c" --gps "lat,lon"
```
Metadata is written as part of `convert`/`batch` — there is no standalone metadata-write command.

### List effects and formats
```bash
contenta effects --json
contenta formats --json
```

### Watch folder for auto-conversion
```bash
contenta watch <dir> --output <dir> --format <fmt> [--recursive] [--include "*.jpg"]
```

## Guidelines

- Always use `--json` when you need to parse the output programmatically; check the exit code (0 success, 3 trial expired, 4 conversion failed, 5 file not found, 6 bundled tools missing).
- For batch operations, `--workers` defaults to CPU core count; lower it on weaker machines.
- Batch is non-recursive by default — add `--recursive` to include subfolders.
- `ai-transform` requires a Google Gemini API key (`GEMINI_API_KEY` env var or `--api-key`); use MCP `ai_transform` tool when connected via `contenta serve`.
- When the user asks to "resize for Amazon/Etsy/Instagram/MLS", use the platform dimensions table above with `--resize` and `--resize-mode fit`.
- For RAW files (CR2, NEF, ARW, DNG, RAF, etc.), `contenta` handles them automatically; RAW-specific flags: `--white-balance camera|auto|manual`, `--denoising`, `--sharpness-boost`.
- AI upscale uses GPU by default. Add `--force-cpu` if no compatible GPU is available.
- The `--quality` flag applies to lossy formats (JPG, WebP, AVIF). For PNG, use `--png-compression 0-9`.
- Trial users get a watermark on output after the first 5 images. Register with `contenta register <email> <key>` to remove all limits.
