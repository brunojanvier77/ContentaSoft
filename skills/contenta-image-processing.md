---
name: Contenta Image Processing
description: Process images using the Contenta Converter CLI — convert formats, resize, apply effects, run professional workflows, AI upscale, create PDF albums, video slideshows, and manage metadata.
tools:
  - Bash
---

# Contenta Image Processing Skill

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

E-Commerce platforms: amazon, etsy, shopify, ebay, walmart
Social platforms: instagram, tiktok, youtube, pinterest, facebook, twitter
Real Estate platforms: mls, zillow, website-hero, print-brochure, email
Wedding platforms: lab-prints, gallery, watermarked-proofs, social-media

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

### List effects
```bash
contenta effects --json
```

### List supported formats
```bash
contenta formats --json
```

### Watch folder for auto-conversion
```bash
contenta watch --input <dir> --output <dir> --format <fmt>
```

## Guidelines

- Always use `--json` when you need to parse the output programmatically.
- For batch operations, set `--workers` to match available CPU cores for best performance.
- When the user asks to "resize for Amazon/Etsy/Instagram", use the `workflows` command — it knows the exact dimensions.
- For RAW files (CR2, NEF, ARW, etc.), `contenta` handles them automatically — no special flags needed.
- AI upscale uses GPU by default. Add `--force-cpu` if GPU is not available.
- The `--quality` flag applies to lossy formats (JPG, WebP, AVIF). For PNG, it controls compression level.
