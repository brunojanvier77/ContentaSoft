# ContentaSoft

Professional desktop image and video processing tools with CLI and MCP server support for AI agents.

All tools run locally on Windows — no cloud uploads, no subscriptions, no per-image fees.

## Products

| Product | Website | CLI | MCP Server | Key Capability |
|---------|---------|-----|------------|----------------|
| [Contenta Converter PREMIUM](https://contenta-converter.com) | [contenta-converter.com](https://contenta-converter.com) | `contenta` (15 commands) | 11 tools | 50+ formats, 35+ effects, AI upscaling, professional workflows |
| [RAW Express](https://contenta-converter.com) | [contenta-converter.com](https://contenta-converter.com) | `rawexpress` (9 commands) | -- | RAW processing for 600+ camera models |
| [VideoRecompress Studio](https://contenta-videorecompress.com) | [contenta-videorecompress.com](https://contenta-videorecompress.com) | `videorecompress` (7 commands) | -- | H.265/AV1 compression, GPU acceleration |
| [3D CAD Batch Converter](https://contenta-software.com) | [contenta-software.com](https://contenta-software.com) | `cadconvert` (6 commands) | -- | STEP/IGES to STL/OBJ/FBX/glTF |
| [AI Video Enhancer](https://contenta-software.com) | [contenta-software.com](https://contenta-software.com) | `aivideoenhancer` (4 commands) | -- | Real-ESRGAN upscaling, RIFE frame interpolation |

## Quick Start: MCP Server

Add Contenta Converter as an MCP server in your AI client (Claude Desktop, Cursor, etc.):

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

This gives your AI agent access to 11 image processing tools. See [MCP Server Documentation](contenta-converter/mcp-server.md) for the full tool reference.

See [MCP Config Guide](mcp-config/) for setup instructions for different clients.

## Quick Start: CLI

```bash
# Convert a photo to WebP
contenta convert photo.jpg --format webp --quality 85

# Batch resize for Amazon product listings
contenta workflows --id builtin.ecommerce --platforms amazon --input ./photos --output ./ready

# AI upscale 4x
contenta upscale photo.jpg --scale 4
```

## Claude Code Skills

Drop-in skill files for [Claude Code](https://claude.com/claude-code) that teach the agent to use ContentaSoft CLI tools:

- [Image Processing Skill](skills/contenta-image-processing.md) — convert, resize, effects, workflows, upscale, PDF albums, slideshows, metadata
- [Video Processing Skill](skills/contenta-video.md) — recompress, batch encode, analyze, GPU-accelerated compression

## Why ContentaSoft for AI Agents

- **Local processing** — images never leave your machine
- **Batch capable** — process thousands of files in parallel
- **50+ formats** — JPG, PNG, WebP, HEIC, AVIF, JXL, TIFF, PSD, RAW (600+ cameras), PDF, SVG, and more
- **Professional workflows** — one command to resize for Amazon, Etsy, Instagram, MLS, and more
- **AI upscaling** — Real-ESRGAN 2x/4x with GPU acceleration
- **Metadata control** — read/write EXIF, IPTC, XMP, GPS
- **No API keys needed** — everything runs locally (except optional AI transform which uses Gemini)
- **Structured output** — all CLI commands support `--json` for machine-readable output

## Platform

- **OS**: Windows 10/11 (x64)
- **Runtime**: .NET 9
- **GPU**: Optional — NVIDIA/Intel/AMD for accelerated upscaling and video encoding
- **Trial**: 30-day free trial, no credit card required

## Links

- [contenta-converter.com](https://contenta-converter.com) — Contenta Converter PREMIUM
- [contenta-videorecompress.com](https://contenta-videorecompress.com) — VideoRecompress Studio
- [contenta-software.com](https://contenta-software.com) — Full product suite
