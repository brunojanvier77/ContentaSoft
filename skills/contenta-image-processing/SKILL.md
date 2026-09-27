---
name: contenta-image-processing
description: Convert, resize, watermark and tag images with the Contenta Converter CLI (contenta). Use when the user asks to convert image formats (including camera RAW, HEIC, AVIF, JPEG XL, PSD, PDF), resize photos for marketplaces or social media, apply effects, add a text watermark, write metadata, export icon sizes, build a PDF album, merge PDFs, make a photo slideshow video, or watch a folder.
allowed-tools: Bash
---

# Contenta Converter (image processing)

Use the `contenta` CLI (Contenta Converter 9.0.33+, Windows). Default per-user install: `%LOCALAPPDATA%\Programs\ContentaConverter\contenta.exe`, on the user PATH. Check with `contenta --version`.

Every command accepts `--json` (machine-readable stdout), `--quiet` and `-v`. Inputs are positional; there is no `--input` flag. Check the exit code after each call.

## Commands

```bash
contenta convert <input> [options]              # one image
contenta batch <input-dir> [options]            # a folder (not recursive unless --recursive)
contenta info <file> --json                     # format + metadata
contenta effects --json                         # 51 effects with parameters
contenta formats --json                         # 102 input / 28 output extensions
contenta pdf-album <files>... --output album.pdf [--photos-per-page N] [--page-size A4|Letter|Legal] [--embed-quality 1-100]
contenta pdf-merge <files>... --output merged.pdf
contenta slideshow <files>... --output <FULL path.mp4> [--template T] [--audio <FULL path>] [--duration ms] [--ken-burns M]
contenta ai-transform <input> --prompt "..." [--output path]   # needs GEMINI_API_KEY
contenta watch <dir> --output <dir> --format <fmt>             # runs until Ctrl+C
contenta profile save|load|validate <file.json>
contenta register <email> <key>
contenta status
```

## Main options (convert and batch)

- `-f/--format`: jpg, png, webp, tiff, bmp, gif, jxl, heic, avif, svg, pdf, ico (and the rest of `formats --output`)
- `-q/--quality` 1-100 (default 90); `--png-compression` 0-9 for PNG
- `--resize WxH --resize-mode fit|fill|stretch|longest-edge|shortest-edge`, `--resize-percent N`, `--dont-enlarge`
- `-o/--output <dir>` (default: next to the input); `--overwrite` (default writes a renamed copy); `--filename-pattern`
- `--effects name1 name2` (space-separated); parameters: `--effects "colorbrightnesscontrast:brightness=20,contrast=10"`. A comma between effect names does not work, and unknown names are ignored without an error, so check names with `contenta effects`.
- `--watermark "text" --watermark-position 0-8 (8 = bottom-right) --watermark-opacity 0-100`
- Metadata: `--copyright --creator --title --description --keywords "a,b" --rights --gps "lat,lon" --datetime ISO8601`
- Multi-size: `--sizes 16,32,48` or `--sizes 1x,2x,3x --base-size 512`, `--size-preset favicon|appicon|srcset`. Writes one file per size (`logo-16.ico`, `logo-32.ico`, ...), not one multi-resolution ICO.
- RAW: `--white-balance camera|auto|manual`, `--color-temperature K`, `--denoising`, `--sharpness-boost`, `--color-boost`, `--contrast-boost`
- batch only: `--include "*.jpg"`, `--exclude`, `--recursive`, `-w/--workers N` (default automatic)
- `--profile file.json` loads settings saved by `profile save`

## Windows specifics

- Windows does not expand `*.jpg`, and `contenta` does not either. In PowerShell pass `(Get-ChildItem .\photos\*.jpg).FullName`; in Git Bash `./photos/*.jpg` works because bash expands it.
- `slideshow` needs full paths for `--output` and `--audio` in 9.0.33; a bare file name fails with exit 4.

## Platform sizes (use with `--resize-mode fit`)

Amazon 2000x2000 · Etsy 2700x2025 · Shopify 2048x2048 · eBay 1600x1600 · Instagram 1080x1080 · TikTok/Reels 1080x1920 · YouTube thumbnail 1280x720 · Pinterest 1000x1500 · Facebook link 1200x630 · Twitter/X 1600x900 · MLS 1024x768 · website hero 1920x1080. These are common sizes; the platform's current rules win.

## Examples (verified on 9.0.33)

```powershell
contenta batch .\products --output .\amazon-ready --format jpg --resize 2000x2000 --resize-mode fit
contenta convert photo.cr2 --format jpg --quality 92 --output .\out
contenta convert photo.jpg --output .\proofs --resize 1200x800 --watermark "(c) Studio" --watermark-position 8 --watermark-opacity 60
contenta convert logo.png --sizes 16,32,48,64 --format ico --output .\icons
contenta pdf-album (Get-ChildItem .\photos\*.jpg).FullName --output album.pdf --photos-per-page 6
contenta pdf-merge a.pdf b.pdf --output merged.pdf
contenta slideshow (Get-ChildItem .\photos\*.jpg).FullName --output "$PWD\reel.mp4" --template tiktok --audio "$PWD\music.mp3"
contenta profile save etsy.json --format jpg --quality 88 --resize 2700x2025 --resize-mode fit
contenta batch .\products --profile etsy.json --output .\etsy-ready
```

## Exit codes

0 success · 1 general error (also a missing required option) · 2 invalid arguments (also no Gemini key) · 3 trial ended · 4 conversion failed · 5 file not found · 6 bundled tools missing (`contenta status`).

## Guidelines

- Ask for or confirm the output folder; never overwrite originals unless the user asks (`--overwrite`).
- For marketplace or social sizes, use the table above with `--resize-mode fit`.
- `ai-transform` sends the image to Google and bills the user's own Gemini key; confirm before using it. `--prompt` is required even with `--remove-background`.
- Trial: 30 days; the first 10 images (lifetime) are clean, then output is watermarked; PDF albums, merged PDFs and slideshows are always marked during the trial. After 30 days commands exit with code 3 until `contenta register <email> <key>`.
