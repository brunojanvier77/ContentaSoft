# MCP Configuration Guide

Set up Contenta Converter as an MCP server in your AI client.

## Prerequisites

1. **Install** Contenta Converter from [contenta-converter.com](https://contenta-converter.com)
2. **Register** your license: `contenta register --key YOUR-KEY`
3. Verify the CLI works: `contenta status`

## Claude Desktop

Edit `%APPDATA%\Claude\claude_desktop_config.json`:

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

Restart Claude Desktop. You should see "contenta-converter" in the MCP tools list.

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
| `contenta` not found | Reinstall Contenta Converter — the installer adds it to PATH. Or provide the full path: `C:\Program Files\ContentaSoft\Contenta Converter PREMIUM 2026\contenta.exe` |
| "Registration required" | Run `contenta register --key YOUR-KEY` or start a free trial |
| Tools not appearing | Restart your AI client after saving the config |
| ai_transform fails | Set `GEMINI_API_KEY` env var or pass `api_key` parameter |

## Available Tools

The MCP server exposes 11 tools. See the [full tool reference](../contenta-converter/mcp-server.md) for parameters and examples.

| Tool | What it does |
|------|-------------|
| `convert_image` | Convert format, resize, add metadata |
| `batch_convert` | Process multiple images at once |
| `detect_format` | Identify image format (RAW, HEIC, PSD, etc.) |
| `read_metadata` | Extract EXIF/IPTC/XMP metadata |
| `list_effects` | Get available effects (23 effects) |
| `upscale_image` | AI upscale 2x or 4x with Real-ESRGAN |
| `apply_workflow` | E-Commerce, Social, Real Estate, Wedding workflows |
| `create_pdf_album` | Build PDF photo albums |
| `write_metadata` | Embed EXIF/IPTC metadata and GPS |
| `create_slideshow` | Create video slideshows for social platforms |
| `ai_transform` | AI image editing via Google Gemini |
