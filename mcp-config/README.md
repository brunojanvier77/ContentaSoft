# MCP Configuration Guide

Set up ContentaSoft tools as MCP servers in your AI client.

## Available MCP Servers

| Server | Command | Tools | Documentation |
|--------|---------|-------|---------------|
| Contenta Converter | `contenta serve` | 10 image tools | [Docs](../contenta-converter/mcp-server.md) |
| VideoRecompress | `videorecompress serve` | 5 video tools | [Docs](../videorecompress/mcp-server.md) |
| CAD Converter | `cadconvert serve` | 4 3D tools | [Docs](../cad-converter/mcp-server.md) |
| AI Video Enhancer | `aivideoenhancer serve` | 4 AI tools | [Docs](../ai-video-enhancer/mcp-server.md) |

## Prerequisites

1. **Install** the product(s) you want to use — the 30-day free trial works with MCP, no registration needed
2. Verify the CLI works (e.g., `contenta status`, `videorecompress status`)
3. If a CLI is not on `PATH`, use the full exe path as the `command` in your MCP config (default install dirs are under `C:\Program Files\ContentaSoft\`)

## Claude Desktop

Edit `%APPDATA%\Claude\claude_desktop_config.json`:

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
    },
    "cad-converter": {
      "command": "cadconvert",
      "args": ["serve"]
    },
    "ai-video-enhancer": {
      "command": "aivideoenhancer",
      "args": ["serve"]
    }
  }
}
```

Add only the servers for products you have installed. Restart Claude Desktop after saving.

## Cursor

Add to your Cursor MCP settings (`.cursor/mcp.json` in your project or global config):

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

## Windsurf

Add to your Windsurf MCP configuration:

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

## Claude Code

Claude Code discovers MCP servers from `~/.claude/mcp.json` or project-level `.claude/mcp.json`:

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

Alternatively, use the [Claude Code skill files](../skills/) for direct CLI integration without MCP.

## With AI Transform (Gemini)

To enable the `ai_transform` tool, add your Google Gemini API key:

```json
{
  "mcpServers": {
    "contenta-converter": {
      "command": "contenta",
      "args": ["serve"],
      "env": {
        "GEMINI_API_KEY": "your-gemini-api-key"
      }
    }
  }
}
```

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `contenta` not found | Provide the full path in the MCP config: `C:\Program Files\ContentaSoft\Contenta Converter PREMIUM\contenta.exe`, or add the install dir to PATH |
| "License required" (trial expired) | Register: `contenta register your@email.com XXXXX-XXXXX-XXXXX-XXXXX-XXXXX` or `videorecompress register your@email.com XXXXX-XXXXX-XXXXX-XXXXX-XXXXX` |
| Tools not appearing | Restart your AI client after saving the config |
| ai_transform fails | Set `GEMINI_API_KEY` env var or pass `api_key` parameter |

## Available Tools

The Contenta Converter MCP server exposes 10 tools. See the [full tool reference](../contenta-converter/mcp-server.md) for parameters and examples.

| Tool | What it does |
|------|-------------|
| `convert_image` | Convert format, resize, add metadata |
| `batch_convert` | Process multiple images at once |
| `detect_format` | Identify image format (RAW, HEIC, PSD, etc.) |
| `read_metadata` | Extract EXIF/IPTC/XMP metadata |
| `list_effects` | Get available effects (32 effects) |
| `upscale_image` | AI upscale 2x or 4x with Real-ESRGAN |
| `create_pdf_album` | Build PDF photo albums |
| `write_metadata` | Embed EXIF/IPTC metadata and GPS |
| `create_slideshow` | Create video slideshows for social platforms |
| `ai_transform` | AI image editing via Google Gemini |
