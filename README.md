# ContentaSoft

Professional desktop image and video processing tools with CLI and MCP server support for AI agents.

All tools run locally on Windows — no cloud uploads, no subscriptions, no per-image fees. **30-day free trial, MCP included.**

## Why not ImageMagick / FFmpeg / free tools?

Those tools are great for developers who know exactly what flags to pass. ContentaSoft is for when you need **domain-specific intelligence** that free tools don't have:

- **"Resize my product photos for Amazon"** — ContentaSoft knows Amazon requires 2000x2000 JPEG with white background padding. ImageMagick requires you to figure that out yourself. Same for Etsy (2700x2025), Shopify (2048x2048), MLS (1024x768), Zillow (3000x2000), Instagram (1080x1080), TikTok (1080x1920), and 20+ other platforms.
- **"Process this RAW file from my Canon R5"** — ContentaSoft handles CR3, NEF, ARW, RAF, DNG, and 600+ other camera RAW formats out of the box. No installing dcraw, no LibRaw compilation, no guessing at white balance flags.
- **"Upscale this photo 4x"** — Real-ESRGAN is bundled and GPU-accelerated. No Python environment, no downloading models, no CUDA setup.
- **"Compress these wedding videos for archival"** — One command: `videorecompress batch --preset wedding_archive`. It picks H.265, CRF 18, hardware-accelerates if your GPU supports it, and verifies output integrity. The FFmpeg equivalent is 15+ flags.

**The pattern**: free tools give you primitives, ContentaSoft gives you workflows. An AI agent using ContentaSoft's MCP server can do in one tool call what would take a chain of FFmpeg/ImageMagick commands with manual dimension lookups.

## Products

| Product | CLI | MCP Server | What it does |
|---------|-----|------------|-------------|
| [Contenta Converter PREMIUM](https://contenta-converter.com) | `contenta` | [11 tools](contenta-converter/mcp-server.md) | Image conversion, resize, effects, workflows, AI upscale, PDF albums, slideshows, metadata |
| [VideoRecompress Studio](https://contenta-videorecompress.com) | `videorecompress` | [5 tools](videorecompress/mcp-server.md) | Video compression with H.265/AV1/VP9, GPU acceleration, 7 presets |
| [3D CAD Batch Converter](https://contenta-software.com/cadconverter/) | `cadconvert` | [4 tools](cad-converter/mcp-server.md) | STEP/IGES to STL/OBJ/FBX/glTF with tessellation control |
| [AI Video Enhancer](https://contenta-software.com/aivideoenhancer/) | `aivideoenhancer` | [4 tools](ai-video-enhancer/mcp-server.md) | AI upscaling (Real-ESRGAN), frame interpolation (RIFE), stabilization |
| [RAW Express](https://contenta-software.com/rawexpress/) | `rawexpress` | -- | Dedicated RAW batch converter with white balance and exposure control |

## Getting Started

### Step 1: Install the software

Download and install the product you need from its website (see [Products](#products) table above). All products are Windows desktop applications with a standard `.exe` installer.

- **No account required** — just download and run the installer
- **30-day free trial** starts automatically — no credit card needed
- **CLI and MCP server are included** in the same installer, no separate download
- **.NET 9 runtime is bundled** — no prerequisites to install

### Step 2: Add the CLI to your PATH

After installation, add the install directory to your system PATH so you can use the CLI from any terminal:

```
# Default install locations:
C:\Program Files\ContentaSoft\Contenta Converter PREMIUM\
C:\Program Files\ContentaSoft\VideoRecompress Studio\
C:\Program Files\ContentaSoft\3D CAD Batch Converter\
C:\Program Files\ContentaSoft\AI Video Enhancer Studio\
```

Verify it works:

```bash
contenta status          # should show "Trial: 30 days remaining"
videorecompress status   # same for VideoRecompress
```

### Step 3: Use the CLI

```bash
# Convert a RAW photo to JPEG
contenta convert photo.cr2 --format jpg --quality 92

# Resize product photos for Amazon — knows the exact 2000x2000 spec
contenta workflows --id builtin.ecommerce --platforms amazon --input ./photos --output ./ready

# AI upscale 4x with GPU
contenta upscale photo.jpg --scale 4

# Compress wedding videos — picks H.265, CRF 18, hardware-accelerated
videorecompress batch ./raw-footage --output ./archived --preset wedding_archive

# Convert STEP to STL for 3D printing
cadconvert convert model.step --format stl --quality fine

# AI upscale old video to 4K
aivideoenhancer enhance old_tape.mp4 --upscale 4x --denoise strong
```

### Step 4: Connect the MCP server to your AI client

Add the product to your AI client's MCP configuration. The MCP server works during the free trial.

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

Add only the products you've installed. See [MCP Config Guide](mcp-config/) for Cursor, Windsurf, and other clients.

| Server | Command | Tools | Full reference |
|--------|---------|-------|----------------|
| Contenta Converter | `contenta serve` | 11 image tools | [MCP Docs](contenta-converter/mcp-server.md) |
| VideoRecompress | `videorecompress serve` | 5 video tools | [MCP Docs](videorecompress/mcp-server.md) |
| CAD Converter | `cadconvert serve` | 4 3D tools | [MCP Docs](cad-converter/mcp-server.md) |
| AI Video Enhancer | `aivideoenhancer serve` | 4 AI tools | [MCP Docs](ai-video-enhancer/mcp-server.md) |

### Step 5 (optional): Install Claude Code skills

Skills teach Claude Code about every command, format, preset, and platform dimension — so you can describe what you want in plain English.

```bash
# Install all 4 skills at once
git clone https://github.com/brunojanvier77/ContentaSoft.git /tmp/ContentaSoft
cp -r /tmp/ContentaSoft/skills/* ~/.claude/skills/
```

After installing, try: *"Resize all photos in D:\Products for Amazon and Etsy"* — Claude will pick the right commands, dimensions, and output formats.

| Skill | Slash command | What it teaches Claude Code |
|-------|-------------|---------------------------|
| [Image Processing](skills/contenta-image-processing/SKILL.md) | `/contenta-image-processing` | Convert formats, resize for Amazon/Etsy/MLS/Instagram, AI upscale, PDF albums, slideshows, metadata |
| [Video Compression](skills/contenta-video/SKILL.md) | `/contenta-video` | H.265/AV1 compression, 7 presets, GPU acceleration, batch processing |
| [CAD Conversion](skills/contenta-cad/SKILL.md) | `/contenta-cad` | STEP/IGES to STL/OBJ/glTF, tessellation control, unit conversion, mesh repair |
| [Video Enhancement](skills/contenta-video-enhancer/SKILL.md) | `/contenta-video-enhancer` | AI upscale 2x/4x, frame interpolation 60fps, stabilization, denoising, 6 presets |

### Trial & Licensing

| | Trial (30 days) | Licensed |
|---|---|---|
| **Desktop GUI** | Full access | Full access |
| **CLI** | Full access, watermark after 10 files/session | Full access |
| **MCP Server** | Full access, watermark after 10 files/session | Full access |
| **Agent Skills** | Work with trial CLI | Work with licensed CLI |
| **Duration** | 30 days from install | Permanent |
| **Credit card** | Not required | One-time purchase |

All licenses are **one-time purchases** — no subscriptions, no renewals. Buy once, use forever including all minor updates.

## Platform & Licensing

- **OS**: Windows 10/11 (x64)
- **Runtime**: .NET 9 (bundled in installer)
- **GPU**: Optional — NVIDIA/Intel/AMD for accelerated upscaling and video encoding
- **Trial**: 30-day free trial, no credit card required. All features including CLI and MCP server work during trial. Output files get a small watermark after 10 files per session.
- **License**: One-time purchase per product. No subscription. Removes all watermarks and trial limits permanently.

## Links

- [contenta-converter.com](https://contenta-converter.com) — Contenta Converter PREMIUM
- [contenta-videorecompress.com](https://contenta-videorecompress.com) — VideoRecompress Studio
- [contenta-software.com](https://contenta-software.com) — Full product suite
- [contenta-software.com/rawexpress](https://contenta-software.com/rawexpress/) — RAW Express
- [contenta-software.com/cadconverter](https://contenta-software.com/cadconverter/) — 3D CAD Batch Converter
- [contenta-software.com/aivideoenhancer](https://contenta-software.com/aivideoenhancer/) — AI Video Enhancer Studio
