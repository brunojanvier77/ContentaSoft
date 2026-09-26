# ContentaSoft

Professional desktop image, video, and 3D processing tools with CLI and MCP server support for AI agents.

All tools run locally on Windows — no cloud uploads, no subscriptions, no per-image fees. **30-day free trial, MCP included.**

## Why not ImageMagick / FFmpeg / free tools?

Those tools are great for developers who know exactly what flags to pass. ContentaSoft is for when you need **domain-specific intelligence** that free tools don't have:

- **"Resize my product photos for Amazon"** — Amazon requires 2000x2000 JPEG. One command: `contenta batch ./photos --output ./ready --format jpg --resize 2000x2000 --resize-mode fit`. Same for Etsy (2700x2025), Shopify (2048x2048), MLS (1024x768), Zillow (3000x2000), Instagram (1080x1080), TikTok (1080x1920), and 20+ other platforms — the dimensions are in the docs and the Claude Code skills, so an AI agent picks them for you.
- **"Process this RAW file from my Canon R5"** — ContentaSoft handles CR3, NEF, ARW, RAF, DNG, and 600+ other camera RAW formats out of the box. No installing dcraw, no LibRaw compilation, no guessing at white balance flags.
- **"Upscale this photo 4x"** — Real-ESRGAN is bundled and GPU-accelerated. No Python environment, no downloading models, no CUDA setup.
- **"Compress these wedding videos for archival"** — One command: `videorecompress batch ./raw-footage --output ./archived --codec h265 --crf 18`. It hardware-accelerates if your GPU supports it and verifies output integrity. The FFmpeg equivalent is 15+ flags.

**The pattern**: free tools give you primitives, ContentaSoft gives you workflows. An AI agent using ContentaSoft's MCP servers can do in one tool call what would take a chain of FFmpeg/ImageMagick commands with manual dimension lookups.

## Products

| Product | CLI | MCP Server | What it does |
|---------|-----|------------|-------------|
| [Contenta Converter PREMIUM](https://www.contenta-converter.com) | `contenta` | [10 tools](contenta-converter/mcp-server.md) | Image conversion (50+ formats, 600+ RAW cameras), resize, effects, AI upscale, PDF albums, slideshows, metadata |
| [VideoRecompress Studio](https://www.contenta-software.com/videorecompress/) | `videorecompress` | [5 tools](videorecompress/mcp-server.md) | Video compression with H.265/AV1/VP9, GPU acceleration, 24 presets, watch folders |
| [3D CAD Converter](https://www.contenta-software.com/3dcadconverter/) | `cadconvert` | [4 tools](cad-converter/mcp-server.md) | STEP/IGES/BREP to STL/OBJ/FBX/glTF and 19 formats with tessellation control |
| [AI Video Enhancer Studio](https://www.contenta-software.com/aivideoenhancer/) | `aivideoenhancer` | [4 tools](ai-video-enhancer/mcp-server.md) | AI upscaling (NVIDIA Video Super Resolution), frame interpolation (RIFE), stabilization, denoising |

## Getting Started

### Step 1: Install the software

Download and install the product you need from its website (see [Products](#products) table above). All products are Windows desktop applications with a standard `.exe` installer.

- **No account required** — just download and run the installer
- **30-day free trial** starts automatically — no credit card needed
- **CLI and MCP server are included** in the same installer, no separate download
- **.NET 9 runtime is bundled** — no prerequisites to install

### Step 2: Add the CLI to your PATH

After installation, add the install directory to your system PATH so you can use the CLI from any terminal (or call the exe with its full path):

```
# Default install locations:
C:\Program Files\ContentaSoft\Contenta Converter PREMIUM\
C:\Program Files\ContentaSoft\VideoRecompress Studio\
C:\Program Files\ContentaSoft\3D CAD Converter\
C:\Program Files\ContentaSoft\AI Video Enhancer Studio\
```

Verify it works:

```bash
contenta status           # license state, bundled tools health, version
videorecompress status    # tool availability, license state, GPU info
aivideoenhancer status    # system & GPU info
cadconvert formats        # lists the 19 supported 3D formats
```

### Step 3: Use the CLI

```bash
# Convert a RAW photo to JPEG
contenta convert photo.cr2 --format jpg --quality 92

# Resize product photos for Amazon (2000x2000 spec)
contenta batch ./photos --output ./ready --format jpg --resize 2000x2000 --resize-mode fit

# AI upscale a photo 4x with GPU
contenta upscale photo.jpg --scale 4

# Compress wedding videos for archival (H.265, CRF 18)
videorecompress batch ./raw-footage --output ./archived --codec h265 --crf 18

# Convert STEP to STL for 3D printing (fine tessellation)
cadconvert convert -i model.step -o model.stl --tessellation 0.01

# AI upscale an old video and denoise it
aivideoenhancer enhance old_tape.mp4 --output ./enhanced --upscale x4 --denoise strong
```

Every CLI documents its exit codes (0 = success, non-zero = specific failure class) — see each product's README for the table.

### Step 4: Connect the MCP server to your AI client

Add the product to your AI client's MCP configuration. The MCP servers work during the free trial.

**Claude Desktop** — edit `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "contenta-converter": {
      "command": "contenta",
      "args": ["serve"]
    },
    "videorecompress": {
      "command": "videorecompress",
      "args": ["serve"]
    }
  }
}
```

**Claude Code** — edit `.claude/settings.json`:

```json
{
  "mcpServers": {
    "contenta-converter": {
      "command": "contenta",
      "args": ["serve"]
    }
  }
}
```

If the CLI is not on `PATH`, use the full exe path as `command` (e.g. `"C:\\Program Files\\ContentaSoft\\Contenta Converter PREMIUM\\contenta.exe"`).

Add only the products you've installed. See [MCP Config Guide](mcp-config/) for Cursor, Windsurf, and other clients.

| Server | Command | Tools | Full reference |
|--------|---------|-------|----------------|
| Contenta Converter | `contenta serve` | 10 image tools | [MCP Docs](contenta-converter/mcp-server.md) |
| VideoRecompress | `videorecompress serve` | 5 video tools | [MCP Docs](videorecompress/mcp-server.md) |
| CAD Converter | `cadconvert serve` | 4 3D tools | [MCP Docs](cad-converter/mcp-server.md) |
| AI Video Enhancer | `aivideoenhancer serve` | 4 AI tools | [MCP Docs](ai-video-enhancer/mcp-server.md) |

### Step 5 (optional): Install Claude Code skills

Skills teach Claude Code about every command, format, preset, and platform dimension — so you can describe what you want in plain English.

```bash
# Git Bash / macOS / Linux
git clone https://github.com/brunojanvier77/ContentaSoft.git /tmp/ContentaSoft
cp -r /tmp/ContentaSoft/skills/* ~/.claude/skills/
```

```powershell
# Windows PowerShell
git clone https://github.com/brunojanvier77/ContentaSoft.git $env:TEMP\ContentaSoft
Copy-Item -Recurse $env:TEMP\ContentaSoft\skills\* $env:USERPROFILE\.claude\skills\
```

After installing, try: *"Resize all photos in D:\Products for Amazon and Etsy"* — Claude will pick the right commands, dimensions, and output formats.

| Skill | Slash command | What it teaches Claude Code |
|-------|-------------|---------------------------|
| [Image Processing](skills/contenta-image-processing/SKILL.md) | `/contenta-image-processing` | Convert formats, resize for Amazon/Etsy/MLS/Instagram, AI upscale, PDF albums, slideshows, metadata |
| [Video Compression](skills/contenta-video/SKILL.md) | `/contenta-video` | H.265/AV1 compression, presets, GPU acceleration, batch processing, watch folders |
| [CAD Conversion](skills/contenta-cad/SKILL.md) | `/contenta-cad` | STEP/IGES to STL/OBJ/glTF, tessellation control, unit conversion, mesh repair |
| [Video Enhancement](skills/contenta-video-enhancer/SKILL.md) | `/contenta-video-enhancer` | AI upscale, frame interpolation, stabilization, denoising, 8 presets |

### Trial & Licensing

| | Trial (30 days) | Licensed |
|---|---|---|
| **Desktop GUI** | Full access | Full access |
| **CLI** | Full access, watermark after the free-file allowance* | Full access |
| **MCP Server** | Full access, watermark after the free-file allowance* | Full access |
| **Agent Skills** | Work with trial CLI | Work with licensed CLI |
| **Duration** | 30 days from install | Permanent |
| **Credit card** | Not required | One-time purchase |

*Free-file allowance per product: Contenta Converter — first 5 images; AI Video Enhancer — first 5 videos (then watermark + 720p cap); VideoRecompress — first 3 videos (then watermark + 10-minute duration cap); 3D CAD Converter — 10 lifetime conversions, then blocked until purchase.

All licenses are **one-time purchases** — no subscriptions, no renewals. Buy once, use forever including all minor updates.

## Platform & Requirements

- **OS**: Windows 10/11 (x64)
- **Runtime**: .NET 9 (bundled in installer)
- **GPU**: Optional for most features — NVIDIA/Intel/AMD accelerate video encoding; Real-ESRGAN photo upscaling uses any Vulkan GPU. **Exception**: AI Video Enhancer's video upscaling requires an NVIDIA RTX GPU (NVIDIA Video Super Resolution); its RIFE frame interpolation needs a Vulkan-capable GPU.
- **Trial**: 30-day free trial, no credit card required. All features including CLI and MCP server work during trial (watermark after the per-product free-file allowance above).
- **License**: One-time purchase per product. No subscription. Removes all watermarks and trial limits permanently.

## Links

- [contenta-converter.com](https://www.contenta-converter.com) — Contenta Converter PREMIUM
- [contenta-software.com/videorecompress](https://www.contenta-software.com/videorecompress/) — VideoRecompress Studio
- [contenta-software.com/aivideoenhancer](https://www.contenta-software.com/aivideoenhancer/) — AI Video Enhancer Studio
- [contenta-software.com/3dcadconverter](https://www.contenta-software.com/3dcadconverter/) — 3D CAD Converter
- [contenta-software.com](https://www.contenta-software.com) — Full product suite
