# Contenta Converter

Convert, resize, watermark and tag thousands of photos from the command line on Windows, including camera RAW files. The same install gives you the desktop app, the `contenta` CLI and an MCP server for AI agents.

[Download the free trial](https://www.contenta-converter.com) | [MCP server reference](mcp-server.md)

Documented version: **9.0.36**.

## What it does

- **Reads 102 file extensions, writes 28.** Input includes JPG, PNG, WebP, HEIC/HEIF, AVIF, JPEG XL, TIFF, PSD/PSB, PDF, SVG, EPS, DjVu, JPEG 2000 and 30 camera RAW extensions (CR2, CR3, NEF, ARW, RAF, DNG, ORF, RW2 and more). Run `contenta formats` for the full list.
- **Batch conversion** of whole folders with parallel workers, resize by size or percentage, output filename patterns, batch rename and ZIP packaging.
- **Multi-page PDF and TIFF**: every page converts, as one numbered file per page or as one multi-page PDF or TIFF (see [Multi-page PDF and TIFF](#multi-page-pdf-and-tiff)).
- **51 effects in 7 categories** (color, enhance, blur/sharpen, artistic, distortion, correction, transforms). Run `contenta effects`.
- **Text watermark** with position and opacity.
- **Metadata**: copyright, creator, keywords, creator email and URL, GPS, date and more, written during conversion. WebP, HEIC and AVIF outputs keep it too (as XMP).
- **Multi-size export**: icon and srcset sets in one command (`--sizes`, `--size-preset`).
- **PDF photo albums** and **PDF merge** (page-level, no re-rendering).
- **Video slideshows** from photos, sized for TikTok, Shorts, Reels, YouTube, Instagram and others.
- **AI transform** with Google Gemini (needs your own Gemini API key).
- **Watch folders**: convert new images as they arrive.

## Install and PATH

The CLI is installed with the desktop app. A default install is per-user, in `%LOCALAPPDATA%\Programs\ContentaConverter\`, and the installer adds that folder to your user `PATH`. Open a new terminal after installing, then check:

```powershell
contenta --version    # 9.0.36 or later (prints the build id after a +)
contenta status       # version, license state, clean outputs left and bundled tools
```

If `contenta` is not found, or `--version` prints an older number, call the exe by its full path (`& "$env:LOCALAPPDATA\Programs\ContentaConverter\contenta.exe"`) or remove the older copy that comes first on `PATH`.

## Commands

Options available on every command: `--json` (machine-readable output), `--quiet`, `-v`/`--verbose`.

| Command | What it does |
|---------|--------------|
| `convert <input>` | Convert one image |
| `batch <input-dir>` | Convert every image in a folder |
| `info <file>` | Format, pixel size, DPI, copyright, creator and the full metadata |
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

Inputs are positional arguments; there is no `--input` flag. Relative paths work everywhere (inputs, `--output`, `--audio`, `--profile`, background images). Run `contenta <command> --help` for every option.

## Examples

The examples match the 9.0.36 CLI. All of them except the camera RAW and `ai-transform` ones were run against 9.0.36 on 2026-10-01, with the file names adapted.

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

# Effects: names separated by spaces or commas, parameters after a colon
contenta convert photo.jpg --effects sepia sharpen --output .\fx
contenta convert photo.jpg --effects "sepia,sharpen:window=5" --output .\fx
contenta convert photo.jpg --effects "colorbrightnesscontrast:brightness=20,contrast=10" --output .\fx
contenta convert photo.jpg --effects blackwhite --output .\bw
contenta convert photo.jpg --effects "flip:direction=vertical" "crop:x=100,y=100,width=800,height=600" --output .\cropped

# Metadata written while converting (kept in AVIF, WebP and HEIC too)
contenta convert photo.jpg --format avif --quality 70 --keywords "beach,summer" --gps "43.2965,5.3698" --copyright "(c) 2026 Studio Name" --creator "Studio Name" --creator-email "studio@example.com" --creator-url "https://example.com" --output .\tagged

# Icon sizes: one file per size (logo-16.ico, logo-32.ico, ...)
contenta convert logo.png --sizes 16,32,48,64 --format ico --output .\icons
contenta convert logo.png --size-preset favicon --output .\favicon

# Format, pixel size, DPI and metadata as JSON
contenta info photo.jpg --json

# Keep each file's format: batch without --format
contenta batch .\products --output .\web --resize 1600x1600 --resize-mode fit --quality 85

# Rename the outputs with Batch Rename tokens: shop_001_1338x2000.jpg, ...
contenta batch .\products --output .\renamed --format jpg --resize 2000x2000 --resize-mode fit --rename-pattern "shop_{seq:3}_{w}x{h}"
contenta batch .\photos --output .\by-date --format jpg --rename-pattern "{date:yyyy-MM-dd}_{camera}_{seq}"

# Also pack the outputs into ZIP files of at most 25 MB each
contenta batch .\products --output .\upload --format jpg --resize 2000x2000 --resize-mode fit --zip-output --zip-split-size 25

# PDF album, 6 photos per A4 page
contenta pdf-album .\photos\img1.jpg .\photos\img2.jpg .\photos\img3.jpg --output album.pdf --photos-per-page 6 --page-size A4

# PDF album on a tiled background image
contenta pdf-album .\photos\img1.jpg .\photos\img2.jpg --output album-bg.pdf --photos-per-page 2 --background-image paper.png --background-mode tile

# Merge PDFs in this order
contenta pdf-merge album.pdf invoice.pdf --output merged.pdf

# Save settings once, reuse them on any folder
contenta profile save etsy.json --format jpg --quality 88 --resize 2700x2025 --resize-mode fit --copyright "(c) My Shop"
contenta batch .\products --profile etsy.json --output .\etsy-ready

# Convert whatever lands in a folder (Ctrl+C to stop)
contenta watch .\incoming --output .\processed --format webp --quality 80
```

An unknown effect name or parameter stops the command with exit code 2 and lists the valid ones, so a typo never converts anything. Parameters you leave out take the desktop app's defaults (`blackwhite` alone uses r=30, g=59, b=11). `flip` takes `direction=horizontal|vertical`; `crop` takes `x`, `y`, `width` and `height` in pixels from the top-left corner.

`--rename-pattern` runs after the batch and uses the desktop app's Batch Rename tokens: `{name}` `{ext}` `{date}` `{date:FORMAT}` `{time}` `{year}` `{month}` `{camera}` `{make}` `{lens}` `{iso}` `{aperture}` `{focal}` `{w}` `{h}` `{gps}` `{seq}` `{seq:N}` `{folder}`. The extension is kept, and a name that is already taken gets `_1`, `_2` and so on.

`--zip-output` writes `contenta-converter-images.zip` in the output folder and leaves the converted files in place. With `--zip-split-size N` the files are spread over several archives (`contenta-converter-images_part2.zip`, `_part3`...) of at most N MB each (default 25); each archive opens on its own.


### Multi-page PDF and TIFF

Every page of a multi-page PDF or TIFF is converted, in `convert`, `batch` and `watch` and in the MCP tools `convert_image` and `batch_convert`.

```powershell
# Every page of a scanned TIFF as a numbered JPEG: scan_page001.jpg, scan_page002.jpg, ...
contenta convert scan.tif -f jpg -o .\pages

# The same TIFF as one PDF holding every page
contenta convert scan.tif -f pdf -o .\pdf

# Only the third page (0-based), as PNG
contenta convert contract.pdf -f png --pdf-page 2 -o .\pages
```

- **One file per page** for formats such as JPG, PNG, WebP or AVIF, numbered in reading order and padded to the same width (`_page001`; four digits past 999 pages). A document with one page keeps its plain name.
- **PDF or TIFF out stays one file.** A multi-page PDF or TIFF converted to PDF or TIFF is written as one multi-page file under the plain name. If one page fails, no document is written.
- **`--pdf-page N`** converts only that page. It is 0-based (0 is the first page) and works for PDF and TIFF input. A page the document does not have is an error that names the page count.
- **Existing files are not overwritten** unless you pass `--overwrite`; a taken name gets ` (1)` appended, like any other output.
- **Limit: 2000 pages per conversion.** A longer document is refused with a message (exit code 4) and nothing is written; convert it a page at a time with `--pdf-page`, or split it first.
- A camera TIFF's thumbnail directory and a pyramid TIFF's reduced-resolution levels are not pages.
- With `--json`, the result lists every file in `outputs` and the input's page count in `pages`. In the `batch` summary, `inputs` is the number of files you gave and `total` the number of outputs, so `total` can be larger.
- The trial counts one output page as one output.

### Passing many files in PowerShell

Windows shells do not expand `*.jpg` for you, and `contenta` does not expand it either, so `contenta pdf-album .\photos\*.jpg` fails with exit code 5. In PowerShell, pass the file list explicitly:

```powershell
contenta pdf-album (Get-ChildItem .\photos\*.jpg).FullName --output album.pdf --photos-per-page 10
```

Git Bash expands `./photos/*.jpg` itself, so the glob form works there.

### Slideshows

```powershell
contenta slideshow (Get-ChildItem .\photos\*.jpg).FullName --output reel.mp4 --template tiktok --audio music.mp3 --duration 2000
```

Templates: `tiktok`, `shorts`, `facebook-reels`, `snapchat` (9:16), `youtube`, `linkedin`, `twitter` (16:9), `instagram` (1:1), `pinterest` (2:3). Ken Burns modes: `zoom-in`, `zoom-out`, `pan-left`, `pan-right`, `pan-up`, `pan-down`, `alternating`, `off`.

### AI transform (needs a Gemini API key)

`ai-transform` sends the image to Google Gemini with your own API key (`--api-key` or the `GEMINI_API_KEY` environment variable). Google bills that key per image, and its image models have no free tier. Without a key the command stops with exit code 2. The examples are illustrative; they were not run for this page:

```powershell
$env:GEMINI_API_KEY = "<your key>"
contenta ai-transform photo.jpg --prompt "Add warm evening light" --output photo_warm.png

# Background removal needs no prompt: white (default), transparent or a #RRGGBB colour
contenta ai-transform product.jpg --remove-background --bg-mode transparent --output product-nobg.png
```

`--prompt` is required unless you pass `--remove-background`. `--model` takes the tiers `fast` (default), `pro` or `lite`, or an explicit Gemini model ID; `--size` takes `1K` (default), `2K` or `4K` (`lite` makes 1K only). During the trial, the result follows the same watermark rule as every other output (see [Trial and license](#trial-and-license)): once the clean outputs are used up the image carries the trial watermark, and `--json` reports `trialWatermarked: true`.

## Useful options

| Option | Notes |
|--------|-------|
| `-f, --format` | `jpg`, `png`, `webp`, `tiff`, `bmp`, `gif`, `jxl`, `heic`, `avif`, `svg`, `pdf`, `ico` (plus the other extensions `contenta formats --output` lists) |
| `-q, --quality` | 1-100, default 90 |
| `--resize WxH` + `--resize-mode` | `fit`, `fill`, `stretch`, `longest-edge`, `shortest-edge`, `fit-with-background`, `crop-to-aspect`. `fill`, `stretch`, `fit-with-background` and `crop-to-aspect` need both sides; `1920x` or `x1080` works with `fit`, `longest-edge` and `shortest-edge` |
| `--resize-percent` | 1-100 |
| `--dont-enlarge` | Leave smaller images at their size |
| `--effects` | Effect names separated by spaces or commas; parameters as `name:key=value,key=value`. Unknown names or parameters exit 2 |
| `--watermark`, `--watermark-position`, `--watermark-opacity` | White text watermark; position 0-8 (8 = bottom-right, the default); opacity 0-100 (default 70) |
| `--copyright`, `--creator`, `--title`, `--description`, `--keywords`, `--rights`, `--creator-city`, `--creator-country`, `--creator-email`, `--creator-url`, `--gps`, `--datetime` | Metadata written into the output (EXIF, IPTC and XMP; XMP only where the format has no IPTC block) |
| `--sizes`, `--size-preset`, `--base-size` | Multi-size export: pixel sizes (`16,32,48`), scale factors (`1x,2x,3x`), or `favicon` / `appicon` / `srcset` |
| `--filename-pattern` | Default `{originalname}` |
| `--overwrite` | Replace existing outputs (default: write a renamed copy) |
| `--png-compression` | zlib level 0-9, default 2 |
| `--white-balance`, `--color-temperature`, `--denoising`, `--sharpness-boost`, `--color-boost`, `--contrast-boost` | RAW decoding |
| `--pdf-page` | Convert only this page (0-based) of a multi-page PDF or TIFF; without it every page converts |
| `--profile` | Load settings saved with `profile save` |
| batch: `--include`, `--exclude`, `--recursive`, `-w/--workers` | Filters; subfolders are not included unless you add `--recursive`; workers default to automatic |
| batch: `--rename-pattern` | Rename the outputs with Batch Rename tokens after the batch |
| batch: `--zip-output`, `--zip-split-size` | Also pack the outputs into ZIP archives, split at N MB (default 25) |
| pdf-album: `--photos-per-page` | 1, 2, 4, 6, 8, 10, 12, 16, 24 or 48 (default 4) |
| pdf-album: `--embed-quality` | JPEG quality for non-JPEG inputs (default 85); JPEG inputs are embedded unchanged |
| pdf-album: `--background`, `--background-image`, `--background-mode` | Page color (hex) or an image drawn behind the photos: `stretch`, `center` or `tile` |

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
| 3 | Reserved. Versions before 9.0.36 used it for an ended trial; nothing returns it now |
| 4 | Conversion failed (also a page number the document does not have, and a document over 2000 pages) |
| 5 | File or folder not found (also a wildcard such as `*.jpg`, which `contenta` does not expand) |
| 6 | Bundled tools missing: run `contenta status` |
| 130 | Cancelled (Ctrl+C) |

Since 9.0.36 a bad option value is a usage error with exit code 2 and one sentence that names the option and what it accepts, instead of a silent default. That covers `--format`, `-q`, `--resize` (not `WxH`), `--resize-mode`, `--color-space`, `--gps`, `--datetime`, `--aspect-ratio`, `--pdf-page`, effect names and effect parameters outside their range, slideshow templates and `--photos-per-page`. A missing `--profile` file exits with 5, a malformed one or a bad `--sizes` list with 2, each with a message instead of a stack trace; add `-v` to see the stack trace. With `--json`, an error prints `{"success":false,"error":"...","exitCode":N}`.

## Trial and license

- **No end date, no account, no card.** The trial converts every file you give it, in full. Each computer gets **10 clean (watermark-free) outputs for life**, plus 10 more after you confirm the newsletter in the desktop app. The count is per computer, never per batch.
- **Which outputs are clean.** The desktop app marks every result of a batch and lets you pick which ones to keep clean. The CLI, the MCP server and watch folders have no picker: the first outputs they write take the clean outputs that are left, and every later output carries the watermark. All of them spend one shared allowance. A failed or skipped file never uses one, and one output page of a multi-page PDF or TIFF counts as one output.
- `contenta status` shows how many clean outputs are left (`cleanOutputsLeft` in `--json`).
- During the trial, PDF albums, merged PDFs and slideshows are always marked (watermark or end card), whatever is left. `ai-transform` results follow the clean-output rule above.
- Registering removes the watermark from every output. Nothing in the CLI or the MCP server stops working when the trial runs out; output is watermarked.
- License: **$129 one-time** for a lifetime license; the [buy page](https://www.contenta-converter.com/buy.php) also lists a quarterly plan.
- Register from the CLI with `contenta register <email> <key>` (the key is 25 characters, dashes optional), or in the desktop app.

## MCP server

`contenta serve` starts an MCP server with 10 tools, each with a title and read-only, destructive and network hints. See the [MCP reference](mcp-server.md).
