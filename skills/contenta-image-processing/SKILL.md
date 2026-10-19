---
name: contenta-image-processing
description: Convert, resize, watermark and tag images with the Contenta Converter CLI (contenta). Use when the user asks to convert image formats (including camera RAW, HEIC, AVIF, JPEG XL, PSD, PDF), resize photos for marketplaces or social media, apply effects, add a text watermark, write metadata, export icon sizes, build a PDF album, merge PDFs, split a multi-page PDF or TIFF into page images, make a photo slideshow video, or watch a folder.
allowed-tools: Bash
---

# Contenta Converter (image processing)

Use the `contenta` CLI (Contenta Converter 9.0.41+, Windows). Default per-user install: `%LOCALAPPDATA%\Programs\ContentaConverter\contenta.exe`, on the user PATH. Check with `contenta --version` (it prints the build id after a `+`). Full reference: https://github.com/brunojanvier77/ContentaSoft/blob/main/contenta-converter/cli.md

Every command accepts `--json`, `--plain`, `--quiet`, `-v/--verbose` and `--progress`. Input files and folders are positional. Relative and absolute paths both work. Check the exit code after each call.

## Running commands

- **MCP first:** when this app's MCP tools are connected (`convert_image`, `batch_convert`, `create_pdf_album` ...), use them for what they cover and the command line for the rest. They take paths and options as named fields, so there is no quoting or wildcard to get wrong. Give them full paths.
- **One command per call:** run one `contenta` command at a time, with no `cd`, `&&`, `;`, pipes, loops, variables or `$(...)`. Anything else on the line can stop the run to ask the user for approval, and the CLI already handles folders and reports its results (below).
- **Let the CLI do the folder work:** `contenta batch <folder>` converts a whole folder (`--include "*.jpg"` picks the files, `--recursive` adds subfolders, `--skip-existing` makes a re-run do only what is missing). `-o` on `convert`, `batch` and `pdf-album` creates missing folders. `pdf-album`, `slideshow` and `pdf-merge` take files, not a folder.
- **Read the results, don't measure them:** every `--json` result lists the written files in `outputs`; `convert` adds `outputSize` (bytes) and `durationMs`, and a `batch` summary adds `durationMs`. No `stat`, `date` or `Get-Item` wrapper is needed. To see what is in a folder, use the agent's own file tools.

## Commands

```bash
contenta convert <input> [options]              # one image
contenta batch <input-dir> [options]            # a folder (not recursive unless --recursive); without -f each file keeps its format
contenta info <file> --json                     # format, width/height, dpi, copyright, creator + full metadata
contenta effects --json                         # 51 effects with parameters
contenta formats --json [--input|--output]      # 102 input / 28 output extensions
contenta pdf-album <files>... --output album.pdf [--photos-per-page N] [--page-size A4|Letter|Legal] [--embed-quality 1-100] [--background-image img --background-mode stretch|center|tile]
contenta pdf-merge <files>... --output merged.pdf
contenta slideshow <files>... --output reel.mp4 [--template T] [--audio music.mp3] [--duration ms] [--ken-burns M]
contenta ai-transform <input> --prompt "..." [--output path]   # needs GEMINI_API_KEY
contenta ai-transform <input> --remove-background [--bg-mode white|transparent|custom --bg-color #RRGGBB]
contenta watch <dir> --output <dir> --format <fmt>             # runs until Ctrl+C
contenta profile save|load|validate <file.json>
contenta status --json                          # version, license block, bundled tools
contenta register <email> <key>
```

## Main options (convert and batch)

- `-f/--format`: jpg, png, webp, tiff, bmp, gif, jxl, heic, avif, svg, pdf, ico (and the rest of `formats --output`)
- `-q/--quality` 1-100 (default 90); `--png-compression` 0-9 for PNG
- `--resize WxH --resize-mode fit|fill|stretch|longest-edge|shortest-edge|fit-with-background|crop-to-aspect`, `--resize-percent N`, `--dont-enlarge`. `fill`, `stretch`, `fit-with-background` and `crop-to-aspect` need both sides; `1920x` or `x1080` works with `fit`, `longest-edge` and `shortest-edge`.
- `-o/--output <dir>` (default: next to the input); `--overwrite` (default writes a renamed copy); `--skip-existing` (a re-run converts only what is missing); `--filename-pattern`
- `--effects sepia sharpen` or `--effects "sepia,sharpen:window=5"` (spaces or commas); parameters as `name:key=value,key=value`, e.g. `"colorbrightnesscontrast:brightness=20,contrast=10"`. Unknown names or parameters exit 2 with the valid list; missing parameters take the app defaults. `flip:direction=horizontal|vertical`, `crop:x=0,y=0,width=800,height=600`. Names and ranges: `contenta effects`.
- `--watermark "text" --watermark-position 0-8 (8 = bottom-right) --watermark-opacity 0-100`
- Metadata: `--copyright --creator --title --description --keywords "a,b" --rights --creator-city --creator-country --creator-email --creator-url --gps "lat,lon" --datetime ISO8601`. Kept in WebP, HEIC and AVIF too (as XMP, since they have no IPTC block).
- Multi-size: `--sizes 16,32,48` or `--sizes 1x,2x,3x --base-size 512`, `--size-preset favicon|appicon|srcset`. Writes one file per size (`logo-16.ico`, `logo-32.ico`, ...), not one multi-resolution ICO.
- RAW: `--white-balance camera|auto|manual`, `--color-temperature K`, `--denoising`, `--sharpness-boost` (on by default for RAW input, as in the app; `--sharpness-boost false` turns it off), `--color-boost`, `--contrast-boost`, `--exposure EV`
- batch only: `--include "*.jpg"`, `--exclude`, `--recursive`, `-w/--workers N` (default automatic), `--report file.csv`
- batch only: `--rename-pattern "shop_{seq:3}_{w}x{h}"` renames the outputs with Batch Rename tokens: `{name} {ext} {date} {date:FORMAT} {time} {year} {month} {camera} {make} {lens} {iso} {aperture} {focal} {w} {h} {gps} {seq} {seq:N} {folder}`
- batch only: `--zip-output [--zip-split-size MB]` also packs the outputs into `contenta-converter-images.zip` (`_part2.zip`... when split; default 25 MB)
- `--pdf-page N` (0-based) converts only that page of a multi-page PDF or TIFF; without it every page converts
- `--profile file.json` loads settings saved by `profile save`

## Multi-page PDF and TIFF

A multi-page PDF or TIFF converts every page. To JPG, PNG, WebP and the like you get one file per page (`scan_page001.jpg`, `scan_page002.jpg`, ...); to PDF or TIFF you get one multi-page file. `--pdf-page N` picks one page (0-based). One conversion writes at most 2000 pages (a longer document is refused and nothing is written; use `--pdf-page`). With `--json` the result lists `outputs` and `pages`; in a `batch` summary `inputs` counts the files given and `total` the outputs written.

```bash
contenta convert scan.tif -f jpg -o ./pages          # scan_page001.jpg, scan_page002.jpg, ...
contenta convert scan.tif -f pdf -o ./pdf            # one PDF with every page
contenta convert contract.pdf -f png --pdf-page 2     # the third page only
```

## Windows specifics

- The examples are for Git Bash, which is what Claude Code's Bash tool runs on Windows: `./photos/*.jpg` expands there. `contenta` does not expand `*.jpg` itself (an unexpanded wildcard exits 3). In PowerShell, convert a folder with `contenta batch .\photos --include "*.jpg"`, and give `pdf-album`, `slideshow` and `pdf-merge` their files by name (`contenta pdf-album a.jpg b.jpg c.jpg --output album.pdf`), since they do not take a folder. `(Get-ChildItem .\photos\*.jpg).FullName` also works, but it puts a second command inside the first, which can stop the run to ask the user. Write paths as `.\out`.

## Platform sizes

Amazon 2000x2000 · Etsy 2700x2025 · Shopify 2048x2048 · eBay 1600x1600 · Instagram 1080x1080 · TikTok/Reels 1080x1920 · YouTube thumbnail 1280x720 · Pinterest 1000x1500 · Facebook link 1200x630 · Twitter/X 1600x900 · MLS 1024x768 · website hero 1920x1080. These are common sizes; the platform's current rules win.

## Recipes

- **Square product images (Amazon, Shopify, eBay):** `--resize-mode fit-with-background` with the platform's size from the table, for Amazon `--resize 2000x2000`. The whole photo is scaled into the square and the rest is padded with white; nothing is cropped. `fit` keeps the photo's shape (a 4:3 photo comes out 2000x1500), and `fill` crops to fill the square.
- **Stay under a file size (for example 500 KB per photo):** write JPG or WebP and lower `-q` step by step (90, 80, 70, 60), reading `outputSize` (bytes) from `convert --json` after each try. If 60 is still too big, resize smaller as well. `-q` does not change PNG, which is lossless; switch to JPG or WebP. For a folder, find the quality on one typical file with `convert`, then run the `batch` with it.

## Examples

```bash
contenta batch ./products --output ./amazon-ready --format jpg --resize 2000x2000 --resize-mode fit-with-background
contenta batch ./products --output ./amazon-ready --format jpg --resize 2000x2000 --resize-mode fit-with-background --skip-existing --json --progress
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

## JSON output

With `--json` stdout holds exactly one JSON object, also for errors and parse errors. Read `success` (boolean), `exitCode`, `error` (one sentence or `null`), `errors[]` (one entry per failed file), `warnings[]` and `outputs[]` (the files written; report these paths, not the ones you asked for). Trial keys appear when they apply: `trialWatermarked`, `trialNotice`, `freeFilesRemaining`, `buyUrl`. `batch --json` prints one result object; with `--progress` it first prints one `{"type":"progress","file":...,"status":"done|skipped|failed","completed":N,"total":T}` line per output, so skip the lines that start with `{"type":"progress"` and parse the last line.

## Exit codes

Same table in all four ContentaSoft CLIs: 0 OK · 1 unexpected error (a bug or a broken install; see the log) · 2 invalid arguments (unknown option, missing argument, a bad value for `--format`, `-q`, `--resize`, `--resize-mode`, `--color-space`, `--gps`, `--datetime`, `--pdf-page`, an unknown effect or effect parameter, a malformed profile, or no Gemini key; nothing was done) · 3 input not found or unreadable (also an unexpanded `*.jpg` and a missing profile) · 4 partial failure (a batch where some files failed) · 5 all failed (a failed `convert`) · 6 license or trial refused (also a key the server rejected) · 130 cancelled.

## Guidelines

- Ask for or confirm the output folder; never overwrite originals unless the user asks (`--overwrite`). To resume an interrupted batch, rerun it with `--skip-existing`.
- For marketplace or social sizes, use the table above: `--resize-mode fit-with-background` when the platform wants the exact size (square product images), `fit` when the photo only has to fit inside it.
- `ai-transform` sends the image to Google and bills the user's own Gemini key; confirm before using it. `--prompt` is required unless `--remove-background` is given.
- Trial: no end date. Every file converts; this computer's first 10 clean outputs (lifetime, plus 10 after the newsletter confirmation in the app) come out without a watermark, then output carries one, including `ai-transform` results. PDF albums, merged PDFs and slideshows are always marked during the trial. **A batch spends one clean output per file** (`--sizes` one per size). Before a batch, run `contenta status --json` (MCP: `get_license_status`) and read its `license` object: `state` (`Trial` or `Registered`), `freeFilesRemaining`, `nextOutput` (`clean` or `watermark`) and `afterFreeFiles`. When `state` is `Trial`, tell the user before you start how many of the files will come out clean (`freeFilesRemaining`; none for a PDF album, merged PDF or slideshow), that the rest will carry the trial watermark, and that a licence removes it: <https://www.contenta-converter.com/buy.php>. Every writing command and MCP tool says what it did: `--json` and the MCP results carry `trialWatermarked` (`true`, or for `batch` the number of marked outputs), `trialNotice` (one sentence with the buy link) and `freeFilesRemaining`; the human output prints the notice as a yellow line. Registered installs get none of these. Nothing stops working; `contenta register <email> <key>` removes the watermark.
- A `batch` in which some files fail exits 4 and names each file in `errors`; the other files are still converted (MCP `batch_convert` returns an error result in that case, so read its text). A folder with nothing to convert exits 0 with `"noMatchingFiles": true`, and `skippedFolders` lists folders that could not be read. `watch --json` prints `{"type":"watcherRestarted"}` when it has to re-attach to the folder.
- MCP: `convert_image` and `batch_convert` also take `watermark_text`, `watermark_position` (0-8) and `watermark_opacity` (0-100), the same options as `--watermark*`. `get_file_info` and `list_formats` match `info` and `formats`.
