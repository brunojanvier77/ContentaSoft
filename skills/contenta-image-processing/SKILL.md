---
name: contenta-image-processing
description: Convert, resize, watermark and tag images with the Contenta Converter CLI (contenta). Use when the user asks to convert image formats (including camera RAW, HEIC, AVIF, JPEG XL, PSD, PDF), resize photos for marketplaces or social media, apply effects, add a text watermark, write metadata, export icon sizes, build a PDF album, merge PDFs, split a multi-page PDF or TIFF into page images, make a photo slideshow video, or watch a folder.
allowed-tools: Bash
---

# Contenta Converter (image processing)

Use the `contenta` CLI (Contenta Converter 9.0.36+, Windows). Default per-user install: `%LOCALAPPDATA%\Programs\ContentaConverter\contenta.exe`, on the user PATH. Check with `contenta --version` (it prints the build id after a `+`).

Every command accepts `--json` (machine-readable stdout), `--quiet` and `-v`. Inputs are positional; there is no `--input` flag. Relative and absolute paths both work. Check the exit code after each call.

## Commands

```bash
contenta convert <input> [options]              # one image
contenta batch <input-dir> [options]            # a folder (not recursive unless --recursive); without -f each file keeps its format
contenta info <file> --json                     # format, width/height, dpi, copyright, creator + full metadata
contenta effects --json                         # 51 effects with parameters
contenta formats --json                         # 102 input / 28 output extensions
contenta pdf-album <files>... --output album.pdf [--photos-per-page N] [--page-size A4|Letter|Legal] [--embed-quality 1-100] [--background-image img --background-mode stretch|center|tile]
contenta pdf-merge <files>... --output merged.pdf
contenta slideshow <files>... --output reel.mp4 [--template T] [--audio music.mp3] [--duration ms] [--ken-burns M]
contenta ai-transform <input> --prompt "..." [--output path]   # needs GEMINI_API_KEY
contenta ai-transform <input> --remove-background [--bg-mode white|transparent|custom --bg-color #RRGGBB]
contenta watch <dir> --output <dir> --format <fmt>             # runs until Ctrl+C
contenta profile save|load|validate <file.json>
contenta register <email> <key>
contenta status                                 # license, clean outputs left, bundled tools
```

## Main options (convert and batch)

- `-f/--format`: jpg, png, webp, tiff, bmp, gif, jxl, heic, avif, svg, pdf, ico (and the rest of `formats --output`)
- `-q/--quality` 1-100 (default 90); `--png-compression` 0-9 for PNG
- `--resize WxH --resize-mode fit|fill|stretch|longest-edge|shortest-edge|fit-with-background|crop-to-aspect`, `--resize-percent N`, `--dont-enlarge`. `fill`, `stretch`, `fit-with-background` and `crop-to-aspect` need both sides; `1920x` or `x1080` works with `fit`, `longest-edge` and `shortest-edge`.
- `-o/--output <dir>` (default: next to the input); `--overwrite` (default writes a renamed copy); `--filename-pattern`
- `--effects sepia sharpen` or `--effects "sepia,sharpen:window=5"` (spaces or commas); parameters as `name:key=value,key=value`, e.g. `"colorbrightnesscontrast:brightness=20,contrast=10"`. Unknown names or parameters exit 2 with the valid list; missing parameters take the app defaults. `flip:direction=horizontal|vertical`, `crop:x=0,y=0,width=800,height=600`. Names and ranges: `contenta effects`.
- `--watermark "text" --watermark-position 0-8 (8 = bottom-right) --watermark-opacity 0-100`
- Metadata: `--copyright --creator --title --description --keywords "a,b" --rights --creator-city --creator-country --creator-email --creator-url --gps "lat,lon" --datetime ISO8601`. Kept in WebP, HEIC and AVIF too (as XMP, since they have no IPTC block).
- Multi-size: `--sizes 16,32,48` or `--sizes 1x,2x,3x --base-size 512`, `--size-preset favicon|appicon|srcset`. Writes one file per size (`logo-16.ico`, `logo-32.ico`, ...), not one multi-resolution ICO.
- RAW: `--white-balance camera|auto|manual`, `--color-temperature K`, `--denoising`, `--sharpness-boost`, `--color-boost`, `--contrast-boost`
- batch only: `--include "*.jpg"`, `--exclude`, `--recursive`, `-w/--workers N` (default automatic)
- batch only: `--rename-pattern "shop_{seq:3}_{w}x{h}"` renames the outputs with Batch Rename tokens: `{name} {ext} {date} {date:FORMAT} {time} {year} {month} {camera} {make} {lens} {iso} {aperture} {focal} {w} {h} {gps} {seq} {seq:N} {folder}`
- batch only: `--zip-output [--zip-split-size MB]` also packs the outputs into `contenta-converter-images.zip` (`_part2.zip`... when split; default 25 MB)
- `--pdf-page N` (0-based) converts only that page of a multi-page PDF or TIFF; without it every page converts
- `--profile file.json` loads settings saved by `profile save`

## Multi-page PDF and TIFF

A multi-page PDF or TIFF converts every page. To JPG, PNG, WebP and the like you get one file per page (`scan_page001.jpg`, `scan_page002.jpg`, ...); to PDF or TIFF you get one multi-page file. `--pdf-page N` picks one page (0-based). One conversion writes at most 2000 pages (exit 4 and nothing written beyond that; use `--pdf-page`). With `--json` the result lists `outputs` and `pages`; in a `batch` summary `inputs` counts the files given and `total` the outputs written.

```bash
contenta convert scan.tif -f jpg -o ./pages          # scan_page001.jpg, scan_page002.jpg, ...
contenta convert scan.tif -f pdf -o ./pdf            # one PDF with every page
contenta convert contract.pdf -f png --pdf-page 2     # the third page only
```

## Windows specifics

- The examples are for Git Bash, which is what Claude Code's Bash tool runs on Windows: `./photos/*.jpg` expands there. `contenta` does not expand `*.jpg` itself, so in PowerShell pass `(Get-ChildItem .\photos\*.jpg).FullName` and write paths as `.\out`.

## Platform sizes (use with `--resize-mode fit`)

Amazon 2000x2000 · Etsy 2700x2025 · Shopify 2048x2048 · eBay 1600x1600 · Instagram 1080x1080 · TikTok/Reels 1080x1920 · YouTube thumbnail 1280x720 · Pinterest 1000x1500 · Facebook link 1200x630 · Twitter/X 1600x900 · MLS 1024x768 · website hero 1920x1080. These are common sizes; the platform's current rules win.

## Examples (verified on 9.0.36)

```bash
contenta batch ./products --output ./amazon-ready --format jpg --resize 2000x2000 --resize-mode fit
contenta convert photo.cr2 --format jpg --quality 92 --output ./out
contenta convert photo.jpg --output ./proofs --resize 1200x800 --watermark "(c) Studio" --watermark-position 8 --watermark-opacity 60
contenta convert logo.png --sizes 16,32,48,64 --format ico --output ./icons
contenta pdf-album ./photos/*.jpg --output album.pdf --photos-per-page 6
contenta pdf-merge a.pdf b.pdf --output merged.pdf
contenta slideshow ./photos/*.jpg --output reel.mp4 --template tiktok --audio music.mp3
contenta profile save etsy.json --format jpg --quality 88 --resize 2700x2025 --resize-mode fit
contenta batch ./products --profile etsy.json --output ./etsy-ready
contenta convert photo.jpg --effects blackwhite "crop:x=100,y=100,width=800,height=600" --output ./fx
contenta batch ./products --output ./upload --format jpg --resize 2000x2000 --resize-mode fit --rename-pattern "shop_{seq:3}" --zip-output
contenta info photo.jpg --json
```

## Exit codes

0 success · 1 general error (also a missing required option) · 2 invalid arguments (a bad value for `--format`, `-q`, `--resize`, `--resize-mode`, `--color-space`, `--gps`, `--datetime`, `--pdf-page`, an unknown effect or effect parameter, or no Gemini key) · 3 not used (older versions: trial ended) · 4 conversion failed (also a page the document does not have) · 5 file or folder not found (also an unexpanded `*.jpg`) · 6 bundled tools missing (`contenta status`) · 130 cancelled. With `--json` an error prints `{"success":false,"error":"...","exitCode":N}`.

## Guidelines

- Ask for or confirm the output folder; never overwrite originals unless the user asks (`--overwrite`).
- For marketplace or social sizes, use the table above with `--resize-mode fit`.
- `ai-transform` sends the image to Google and bills the user's own Gemini key; confirm before using it. `--prompt` is required unless `--remove-background` is given.
- Trial: no end date. Every file converts; this computer's first 10 clean outputs (lifetime, plus 10 after the newsletter confirmation in the app) come out without a watermark, then output carries one, including `ai-transform` results. PDF albums, merged PDFs and slideshows are always marked during the trial. `contenta status --json` shows how many are left (`cleanOutputsLeft`). **A batch spends one clean output per file** (`--sizes` one per size), so before a batch on the trial, read `cleanOutputsLeft` and tell the user how many of the files will come out clean; the CLI and MCP results of 9.0.36 do not say which outputs were marked. Nothing stops working; `contenta register <email> <key>` removes the watermark.
