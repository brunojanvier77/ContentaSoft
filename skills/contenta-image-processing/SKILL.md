---
name: contenta-image-processing
description: Convert, resize, upscale, and process images using the Contenta Converter CLI. Use when the user asks to convert image formats, resize photos for e-commerce/social media, apply effects, AI upscale, create PDF albums, video slideshows, or read/write image metadata.
allowed-tools: Bash
---

# Contenta Image Processing

You have access to the `contenta` CLI for professional image processing. All commands support `--json` for structured output.

## Commands

### Convert a single image
```bash
contenta convert <input> --format <fmt> --quality <1-100> --resize-width <px> --resize-height <px>
```
Formats: jpg, png, webp, tiff, bmp, gif, heic, avif, jxl, svg, pdf

### Batch convert
```bash
contenta batch --input <dir> --output <dir> --format <fmt> --quality <q> --workers <n>
```

### Apply professional workflow
```bash
contenta workflows --id <workflow_id> --platforms <p1,p2> --input <dir> --output <dir>
```
Workflow IDs: `builtin.ecommerce`, `builtin.social`, `builtin.realestate`, `builtin.wedding`

E-Commerce platforms: amazon (2000x2000), etsy (2700x2025), shopify (2048x2048), ebay (1600x1600), walmart (2000x2000)
Social platforms: instagram (1080x1080), tiktok (1080x1920), youtube (1280x720), pinterest (1000x1500), facebook (1200x630), twitter (1600x900)
Real Estate platforms: mls (1024x768), zillow (3000x2000), website-hero (1920x1080), print-brochure (2400x3000), email (800x600)
Wedding platforms: lab-prints (3600x2400), gallery (2048x1365), watermarked-proofs (1200x800), social-media (1080x1080)

### AI upscale
```bash
contenta upscale <input> --scale <2|4> --model <realesrgan-x4plus|realesrgan-x4plus-anime>
```

### Create PDF album
```bash
contenta pdf-album --input <dir> --output <file.pdf> --photos-per-page <1|2|4|6|8|12|16|24|48> --page-size <A4|Letter|Legal>
```

### Create video slideshow
```bash
contenta slideshow --input <dir> --output <file.mp4> --template <template> --audio <file> --ken-burns <mode>
```
Templates: tiktok, shorts, facebook-reels, snapchat, youtube, linkedin, twitter, instagram, pinterest
Ken Burns modes: zoom-in, zoom-out, pan-left, pan-right, alternating, off

### Read metadata
```bash
contenta info <file> --json
```

### Write metadata
```bash
contenta convert <file> --copyright "text" --creator "name"
```

### List effects and formats
```bash
contenta effects --json
contenta formats --json
```

### Watch folder for auto-conversion
```bash
contenta watch --input <dir> --output <dir> --format <fmt>
```

## Guidelines

- Always use `--json` when you need to parse the output programmatically.
- For batch operations, set `--workers` to match available CPU cores for best performance.
- When the user asks to "resize for Amazon/Etsy/Instagram/MLS", use the `workflows` command with the right platform ID. It knows the exact dimensions each platform requires.
- For RAW files (CR2, NEF, ARW, DNG, RAF, etc.), `contenta` handles them automatically.
- AI upscale uses GPU by default. Add `--force-cpu` if GPU is not available.
- The `--quality` flag applies to lossy formats (JPG, WebP, AVIF). For PNG, it controls compression level.
- Trial users get a watermark on output after 10 images. No other restrictions.
