# Contenta Converter

Convert, resize, watermark and tag thousands of photos from the command line on Windows, including camera RAW files. The same install gives you the desktop app, the `contenta` CLI and an MCP server for AI agents.

[Download the free trial](https://www.contenta-converter.com) | [MCP server reference](mcp-server.md)

Documented version: **9.0.33**.

## What it does

- **Reads 102 file extensions, writes 28.** Input includes JPG, PNG, WebP, HEIC/HEIF, AVIF, JPEG XL, TIFF, PSD/PSB, PDF, SVG, EPS, DjVu, JPEG 2000 and 30 camera RAW extensions (CR2, CR3, NEF, ARW, RAF, DNG, ORF, RW2 and more). Run `contenta formats` for the full list.
- **Batch conversion** of whole folders with parallel workers, resize by size or percentage, and output filename patterns.
- **51 effects in 7 categories** (color, enhance, blur/sharpen, artistic, distortion, correction, transforms). Run `contenta effects`.
- **Text watermark** with position and opacity.
- **Metadata**: copyright, creator, keywords, GPS, date and more, written during conversion.
- **Multi-size export**: icon and srcset sets in one command (`--sizes`, `--size-preset`).
- **PDF photo albums** and **PDF merge** (page-level, no re-rendering).
- **Video slideshows** from photos, sized for TikTok, Shorts, Reels, YouTube, Instagram and others.
- **AI transform** with Google Gemini (needs your own Gemini API key).
- **Watch folders**: convert new images as they arrive.

## Install and PATH

The CLI is installed with the desktop app. A default install is per-user, in `%LOCALAPPDATA%\Programs\ContentaConverter\`, and the installer adds that folder to your user `PATH`. Open a new terminal after installing, then check:

```powershell
contenta --version    # 9.0.33 or later
contenta status       # license state and bundled tools
```

If `contenta` is not found, or `--version` prints an older number, call the exe by its full path (`& "$env:LOCALAPPDATA\Programs\ContentaConverter\contenta.exe"`) or remove the older copy that comes first on `PATH`.

## Commands

Options available on every command: `--json` (machine-readable output), `--quiet`, `-v`/`--verbose`.

| Command | What it does |
|---------|--------------|
| `convert <input>` | Convert one image |
| `batch <input-dir>` | Convert every image in a folder |
| `info <file>` | Format detection and metadata |
| `effects` | List the effects and their parameters |
| `formats` | List input and output formats (`--input` / `--output` to filter) |
| `pdf-album <files>...` | Lay photos out as a PDF album |
| `pdf-merge <files>...` | Append PDF files into one PDF, in argument order |
| `slideshow <files>...` | Make an MP4/WebM slideshow video from photos |
| `ai-transform <input>` | Edit an image with a Google Gemini prompt |
| `watch <dir>` | Convert new or changed images in a folder |
| `profile save\|load\|validate <file>` | Save conversion settings to JSON and reuse them |
| `register <email> <key>` | Register a license key |
| `status` | License state, bundled tools, version |
| `serve` | Start the MCP server (stdio) |

Inputs are positional arguments; there is no `--input` flag. Run `contenta <command> --help` for every option.

## Examples

Each of these was run against 9.0.33 in PowerShell.

```powershell
# One file to WebP
contenta convert photo.jpg --format webp --quality 85 --output .\out

# A Canon RAW file to JPEG
contenta convert photo.cr2 --format jpg --quality 92 --output .\out

# A folder to PNG
contenta batch .\photos --output .\converted --format png

# Product photos for Amazon: fit inside 2000x2000
contenta batch .\products --output .\amazon-ready --format jpg --resize 2000x2000 --resize-mode fit

# Watermarked proofs: 1200x800, text bottom-right at 60% opacity
contenta convert photo.jpg --output .\proofs --resize 1200x800 --watermark "(c) Studio Name" --watermark-position 8 --watermark-opacity 60

# Effects: separate several with spaces, parameters after a colon
contenta convert photo.jpg --effects sepia sharpen --output .\fx
contenta convert photo.jpg --effects "colorbrightnesscontrast:brightness=20,contrast=10" --output .\fx

# Keywords and GPS written while converting
contenta convert photo.jpg --format avif --quality 70 --keywords "beach,summer" --gps "43.2965,5.3698" --output .\tagged

# Icon sizes: one file per size (logo-16.ico, logo-32.ico, ...)
contenta convert logo.png --sizes 16,32,48,64 --format ico --output .\icons
contenta convert logo.png --size-preset favicon --output .\favicon

# Metadata as JSON
contenta info photo.jpg --json

# PDF album, 6 photos per A4 page
contenta pdf-album .\photos\img1.jpg .\photos\img2.jpg .\photos\img3.jpg --output album.pdf --photos-per-page 6 --page-size A4

# Merge PDFs in this order
contenta pdf-merge album.pdf invoice.pdf --output merged.pdf

# Save settings once, reuse them on any folder
contenta profile save etsy.json --format jpg --quality 88 --resize 2700x2025 --resize-mode fit --copyright "(c) My Shop"
contenta batch .\products --profile etsy.json --output .\etsy-ready

# Convert whatever lands in a folder (Ctrl+C to stop)
contenta watch .\incoming --output .\processed --format webp --quality 80
```

### Passing many files in PowerShell

Windows shells do not expand `*.jpg` for you, and `contenta` does not expand it either, so `contenta pdf-album .\photos\*.jpg` fails with exit code 5. In PowerShell, pass the file list explicitly:

```powershell
contenta pdf-album (Get-ChildItem .\photos\*.jpg).FullName --output album.pdf --photos-per-page 10
```

Git Bash expands `./photos/*.jpg` itself, so the glob form works there.

### Slideshows: use full paths

In 9.0.33, `slideshow` needs full paths for `--output` and `--audio`. A bare file name like `--output reel.mp4` fails.

```powershell
contenta slideshow (Get-ChildItem .\photos\*.jpg).FullName --output "$PWD\reel.mp4" --template tiktok --audio "$PWD\music.mp3" --duration 2000
```

Templates: `tiktok`, `shorts`, `facebook-reels`, `snapchat` (9:16), `youtube`, `linkedin`, `twitter` (16:9), `instagram` (1:1), `pinterest` (2:3). Ken Burns modes: `zoom-in`, `zoom-out`, `pan-left`, `pan-right`, `pan-up`, `pan-down`, `alternating`, `off`.

### AI transform (needs a Gemini API key)

`ai-transform` sends the image to Google Gemini with your own API key (`--api-key` or the `GEMINI_API_KEY` environment variable). Without a key it stops with exit code 2. This example is illustrative; it was not run for this page:

```powershell
$env:GEMINI_API_KEY = "<your key>"
contenta ai-transform photo.jpg --prompt "Replace the background with plain white" --output "$PWD\photo_white.png"
```

`--prompt` is required on every call, including with `--remove-background`.

## Useful options

| Option | Notes |
|--------|-------|
| `-f, --format` | `jpg`, `png`, `webp`, `tiff`, `bmp`, `gif`, `jxl`, `heic`, `avif`, `svg`, `pdf`, `ico` (plus the other extensions `contenta formats --output` lists) |
| `-q, --quality` | 1-100, default 90 |
| `--resize WxH` + `--resize-mode` | `fit`, `fill`, `stretch`, `longest-edge`, `shortest-edge` |
| `--resize-percent` | 1-100 |
| `--dont-enlarge` | Leave smaller images at their size |
| `--effects` | Effect names separated by spaces; parameters as `name:key=value,key=value` |
| `--watermark`, `--watermark-position`, `--watermark-opacity` | White text watermark; position 0-8 (8 = bottom-right, the default); opacity 0-100 (default 70) |
| `--copyright`, `--creator`, `--title`, `--description`, `--keywords`, `--rights`, `--gps`, `--datetime` | Metadata written into the output |
| `--sizes`, `--size-preset`, `--base-size` | Multi-size export: pixel sizes (`16,32,48`), scale factors (`1x,2x,3x`), or `favicon` / `appicon` / `srcset` |
| `--filename-pattern` | Default `{originalname}` |
| `--overwrite` | Replace existing outputs (default: write a renamed copy) |
| `--png-compression` | zlib level 0-9, default 2 |
| `--white-balance`, `--color-temperature`, `--denoising`, `--sharpness-boost`, `--color-boost`, `--contrast-boost` | RAW decoding |
| `--pdf-page` | Page index (0-based) for multi-page PDF/TIFF input |
| `--profile` | Load settings saved with `profile save` |
| batch: `--include`, `--exclude`, `--recursive`, `-w/--workers` | Filters; subfolders are not included unless you add `--recursive`; workers default to automatic |
| pdf-album: `--photos-per-page` | 1, 2, 4, 6, 8, 10, 12, 16, 24 or 48 (default 4) |
| pdf-album: `--embed-quality` | JPEG quality for non-JPEG inputs (default 85); JPEG inputs are embedded unchanged |

## Size cheat sheet

Use `--resize WxH --resize-mode fit`. These are common listing sizes, not limits enforced by the tool; check the platform's current guidelines.

| Use | Size | Use | Size |
|-----|------|-----|------|
| Amazon | 2000x2000 | Instagram post | 1080x1080 |
| Etsy | 2700x2025 | TikTok / Reels | 1080x1920 |
| Shopify | 2048x2048 | YouTube thumbnail | 1280x720 |
| eBay | 1600x1600 | Pinterest | 1000x1500 |
| Real estate (MLS) | 1024x768 | Facebook link image | 1200x630 |
| Website hero | 1920x1080 | Twitter/X | 1600x900 |

## Exit codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error (also missing required options) |
| 2 | Invalid arguments |
| 3 | License required: the trial has ended |
| 4 | Conversion failed |
| 5 | File not found |
| 6 | Bundled tools missing: run `contenta status` |

## Trial and license

- The trial lasts 30 days. The first **10 images** (counted over the life of the install, not per batch) come out clean; after that, output carries a watermark.
- During the trial, PDF albums, merged PDFs and slideshows are always marked (watermark or end card).
- After 30 days, conversion commands and the MCP server stop with exit code 3 until you register.
- License: **$129 one-time** for a lifetime license; the [buy page](https://www.contenta-converter.com/buy.php) also lists a quarterly plan.
- Register from the CLI with `contenta register <email> <key>`, or in the desktop app.

## MCP server

`contenta serve` starts an MCP server with 10 tools. See the [MCP reference](mcp-server.md).
