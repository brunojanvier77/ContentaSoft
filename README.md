# ContentaSoft

Command-line tools and MCP servers for four Windows desktop apps that process images, video and 3D files on your own PC. Conversion, compression and CAD work run locally; your files are not uploaded.

| Product | CLI | MCP tools | What you get |
|---------|-----|-----------|--------------|
| [Contenta Converter](https://www.contenta-converter.com) 9.0.36 | `contenta` | [10](contenta-converter/mcp-server.md) | Batch image conversion (102 input extensions including 30 camera RAW, 28 output), resize, watermark, metadata, effects, batch rename and ZIP, icon sets, multi-page PDF and TIFF to one file per page, PDF albums and merges, photo slideshows |
| [VideoRecompress Studio](https://www.contenta-software.com/videorecompress/) 2026.2.22 | `videorecompress` | [5](videorecompress/mcp-server.md) | Smaller videos with H.265, AV1 or VP9, hardware encoding, 24 presets, batch and watch folders |
| [AI Video Enhancer Studio](https://www.contenta-software.com/aivideoenhancer/) 2026.7.16 | `aivideoenhancer` | [4](ai-video-enhancer/mcp-server.md) | NVIDIA Video Super Resolution upscaling, RIFE AI interpolation to 30/60 fps, stabilization, denoise, still frames, highlight reels |
| [3D CAD Converter](https://www.contenta-software.com/3dcadconverter/) 1.0.27 | `cadconvert` | [4](cad-converter/mcp-server.md) | STEP/IGES/BREP to STL, OBJ, 3MF, glTF/GLB, FBX and more; VRML in and out; 19 formats read, 14 written |

Each product folder has a README with every command, option and exit code, and examples checked against the version shown. Each MCP page lists every tool with its title and its read-only, destructive and network hints.

## Why a CLI instead of ImageMagick or FFmpeg?

Those tools are excellent if you already know the flags. These CLIs package the decisions for common jobs:

- `contenta convert photo.cr2 --format jpg` decodes a Canon RAW file with the bundled decoder; there is nothing else to install.
- `videorecompress batch .\videos --preset-id phone_archive --output .\compressed` picks the codec, quality and GPU encoder, then checks every output before reporting success.
- `cadconvert convert -i part.step -o part.stl --tessellation 0.01 --angular 0.1` meshes a STEP file with OpenCascade at a quality you choose.
- `aivideoenhancer enhance tape.avi -o .\restored --preset old_video_restoration` stabilizes, denoises and upscales in one pass.

An AI agent connected to the MCP servers can do each of these in one tool call.

All four MCP servers run on the same host: they speak MCP protocol versions 2025-11-25, 2025-06-18, 2025-03-26 and 2024-11-05, answer `ping` while a tool is running, cancel a running call on `notifications/cancelled`, and keep stdout for JSON-RPC only.

## Getting started

### 1. Install

Download the product from its website (links above). Each installer contains the desktop app, the CLI and the MCP server. Requirements: Windows 10 or 11, 64-bit. The .NET runtime is included.

A default install is per-user, in `%LOCALAPPDATA%\Programs\<Product>\`, and the installer adds that folder to your user `PATH`:

| Product | Default folder |
|---------|----------------|
| Contenta Converter | `%LOCALAPPDATA%\Programs\ContentaConverter\` |
| VideoRecompress Studio | `%LOCALAPPDATA%\Programs\VideoRecompressStudio\` |
| AI Video Enhancer Studio | `%LOCALAPPDATA%\Programs\AIVideoEnhancerStudio\` |
| 3D CAD Converter | `%LOCALAPPDATA%\Programs\CadConverter\` |

### 2. Check the CLI

Open a new terminal:

```powershell
contenta --version
videorecompress --version
aivideoenhancer --version
cadconvert --version
```

`--version` prints the version followed by a build id (`9.0.36+<id>`). If a command is not found, or prints an older version than the table above, an older copy is earlier on `PATH`: call the exe by its full path, or uninstall the old copy.

### 3. Try it

```powershell
contenta batch .\products --output .\amazon-ready --format jpg --resize 2000x2000 --resize-mode fit
videorecompress recompress video.mp4 --codec h265 --crf 18 --output .\archive
aivideoenhancer enhance .\clip.mp4 -o .\enhanced --upscale x2 --denoise light
cadconvert convert -i model.step -o model.stl
```

Relative and full paths both work in every CLI.

### 4. Connect an AI client (optional)

Every CLI has a `serve` command that runs an MCP server over stdio. For Claude Code on Windows, one command is enough; nothing else needs installing, because the client calls the installed exe directly:

```powershell
claude mcp add contenta-converter -- contenta serve
```

For Claude Desktop, add this to `%APPDATA%\Claude\claude_desktop_config.json` and restart it:

```json
{
  "mcpServers": {
    "contenta-converter": { "command": "contenta", "args": ["serve"] },
    "videorecompress": { "command": "videorecompress", "args": ["serve"] },
    "ai-video-enhancer": { "command": "aivideoenhancer", "args": ["serve"] },
    "cad-converter": { "command": "cadconvert", "args": ["serve"] }
  }
}
```

Keep only the products you installed. The [MCP config guide](mcp-config/) covers Claude Code scopes, Cursor, Windsurf, full exe paths and the Gemini key.

### 5. Install the Claude Code skills (optional)

The skills tell Claude Code which commands, options, presets and sizes to use, so you can ask in plain language ("resize the photos in D:\Products for Amazon and Etsy").

```powershell
git clone https://github.com/brunojanvier77/ContentaSoft.git $env:TEMP\ContentaSoft
New-Item -ItemType Directory -Force $env:USERPROFILE\.claude\skills | Out-Null
Copy-Item -Recurse $env:TEMP\ContentaSoft\skills\* $env:USERPROFILE\.claude\skills\
```

| Skill | Covers |
|-------|--------|
| [contenta-image-processing](skills/contenta-image-processing/SKILL.md) | Conversion, marketplace and social sizes, watermark, metadata, icon sets, PDF albums and merges, slideshows |
| [contenta-video](skills/contenta-video/SKILL.md) | H.265/AV1/VP9 compression, presets, batch, watch folders |
| [contenta-video-enhancer](skills/contenta-video-enhancer/SKILL.md) | Upscaling, interpolation, stabilization, denoise, still frames, highlight reels |
| [contenta-cad](skills/contenta-cad/SKILL.md) | STEP/IGES to mesh, mesh quality, units, repair, batch |

## Trial and licensing

Every product has a free trial with no account and no credit card, and the CLI and MCP server are included. Nothing in a CLI or an MCP server stops working when the free allowance runs out: conversion continues, with the limits in the table. Each allowance is counted per computer, never per batch, and confirming the newsletter in the desktop app adds 10 more.

| Product | Free in the trial | After that | Lifetime license |
|---------|-------------------|------------|------------------|
| Contenta Converter | 10 clean outputs per computer, for life; no end date. PDF albums, merged PDFs and slideshows are always marked | Every later output carries a watermark | $129 |
| VideoRecompress Studio | 10 files without restrictions, for life; no end date | Watermark and a 10-minute cut | $79 |
| AI Video Enhancer Studio | 5 full exports at full resolution, for life; no end date. Free clean 10-second clips with `enhance --clip` | Watermark and a 1280x720 cap | $129 |
| 3D CAD Converter | 10 conversions at full quality within 30 days of the first launch | Conversion continues as a trial export: draft-quality meshes for STEP, IGES and BREP, and a note in the file. No watermark | $179 |

A lifetime license is a one-time payment and removes the trial limits. Each product's buy page also lists a quarterly plan.

## Links

- [contenta-converter.com](https://www.contenta-converter.com): Contenta Converter
- [contenta-software.com/videorecompress](https://www.contenta-software.com/videorecompress/): VideoRecompress Studio
- [contenta-software.com/aivideoenhancer](https://www.contenta-software.com/aivideoenhancer/): AI Video Enhancer Studio
- [contenta-software.com/3dcadconverter](https://www.contenta-software.com/3dcadconverter/): 3D CAD Converter
- [contenta-software.com](https://www.contenta-software.com): all products
