# MCP configuration guide

Connect the ContentaSoft MCP servers to your AI client. Each product's CLI has a `serve` command that runs an MCP server over stdio.

| Server | Command | Tools | Reference |
|--------|---------|-------|-----------|
| Contenta Converter | `contenta serve` | 10 image tools | [Docs](../contenta-converter/mcp-server.md) |
| VideoRecompress Studio | `videorecompress serve` | 5 video compression tools | [Docs](../videorecompress/mcp-server.md) |
| AI Video Enhancer Studio | `aivideoenhancer serve` | 4 video enhancement tools | [Docs](../ai-video-enhancer/mcp-server.md) |
| 3D CAD Converter | `cadconvert serve` | 4 3D tools | [Docs](../cad-converter/mcp-server.md) |

## Before you start

1. Install the products you want. The MCP servers work during the free trial, which has no registration step; each product's README describes what the trial includes.
2. Open a new terminal and check that the CLI answers, e.g. `contenta --version`.
3. Add only the servers for products you have installed.

**Command or full path?** The installers put each CLI in a per-user folder and add it to your user `PATH`:

| Product | Default exe |
|---------|-------------|
| Contenta Converter | `%LOCALAPPDATA%\Programs\ContentaConverter\contenta.exe` |
| VideoRecompress Studio | `%LOCALAPPDATA%\Programs\VideoRecompressStudio\videorecompress.exe` |
| AI Video Enhancer Studio | `%LOCALAPPDATA%\Programs\AIVideoEnhancerStudio\aivideoenhancer.exe` |
| 3D CAD Converter | `%LOCALAPPDATA%\Programs\CadConverter\cadconvert.exe` |

The short command (`"command": "contenta"`) works when the client was started after the install. If the client cannot find it, or an older copy elsewhere on `PATH` answers, use the full path. JSON does not expand `%LOCALAPPDATA%`, so write it out and double the backslashes: `"C:\\Users\\<you>\\AppData\\Local\\Programs\\ContentaConverter\\contenta.exe"`.

## Claude Desktop

Edit `%APPDATA%\Claude\claude_desktop_config.json` (Settings > Developer > Edit Config), then restart Claude Desktop:

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

The same file is in this folder: [claude-desktop-config.json](claude-desktop-config.json).

## What every server does

The four servers run on one shared host, so they behave the same way in any client:

- **Protocol versions:** 2025-11-25, 2025-06-18, 2025-03-26 and 2024-11-05. The server answers with the version your client asks for.
- **stdout is JSON-RPC only;** logs go to stderr.
- **`ping` is answered while a tool runs,** and **`notifications/cancelled` stops a running call** (the call then gets no reply, as the specification says).
- **Tool calls run one at a time,** in the order they arrive.
- **Every tool has a title and annotations,** so a client can ask before it runs a tool that writes files or calls the network. Only Contenta Converter's `ai_transform` uses the network (Google Gemini).
- **Tools only:** no resources and no prompts.

Per-tool titles and hints are in each server's reference page ([Contenta Converter](../contenta-converter/mcp-server.md#tool-annotations), [VideoRecompress Studio](../videorecompress/mcp-server.md#tool-annotations), [AI Video Enhancer Studio](../ai-video-enhancer/mcp-server.md#tool-annotations), [3D CAD Converter](../cad-converter/mcp-server.md#tool-annotations)).

## Claude Code

On Windows, add a server from the command line. Nothing else needs installing: Claude Code calls the installed exe directly, so no `npx` or wrapper is involved:

```powershell
claude mcp add contenta-converter -- contenta serve
claude mcp add videorecompress -- videorecompress serve
claude mcp add ai-video-enhancer -- aivideoenhancer serve
claude mcp add cad-converter -- cadconvert serve
```

By default a server is added for the current project on your machine only. `claude mcp add --scope user contenta-converter -- contenta serve` makes it available in every project; `--scope project` writes it to the project's `.mcp.json` so it is shared with the repository. A project `.mcp.json` uses the same `mcpServers` format as the Claude Desktop file above. Run `claude mcp list` to check. To give `ai_transform` a Gemini key: `claude mcp add -e GEMINI_API_KEY=<your key> contenta-converter -- contenta serve`.

You can also skip MCP and let Claude Code call the CLIs directly with the [skills](../skills/).

## Cursor

Add to `.cursor\mcp.json` in your project, or to `%USERPROFILE%\.cursor\mcp.json` for all projects:

```json
{
  "mcpServers": {
    "contenta-converter": { "command": "contenta", "args": ["serve"] }
  }
}
```

## Windsurf

Add to `%USERPROFILE%\.codeium\windsurf\mcp_config.json`:

```json
{
  "mcpServers": {
    "contenta-converter": { "command": "contenta", "args": ["serve"] }
  }
}
```

## Gemini key for ai_transform

Contenta Converter's `ai_transform` tool uses your own Google Gemini API key. Put it in the server's environment:

```json
{
  "mcpServers": {
    "contenta-converter": {
      "command": "contenta",
      "args": ["serve"],
      "env": { "GEMINI_API_KEY": "<your key>" }
    }
  }
}
```

Or pass it per call with the tool's `api_key` parameter. Google bills the calls to that key.

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Server not found / fails to start | Use the full exe path in `command`; restart the client after editing the config |
| Output has a watermark, or a CAD file is draft quality | The free allowance is used up. Nothing stops working; register to remove the limits: `contenta register <email> <key>`, `videorecompress register <email> <key>`, `aivideoenhancer register <email> <key>`, `cadconvert register -k <key> -e <email>`, or in the desktop app |
| Tools do not appear | Restart the client; check the JSON is valid |
| `ai_transform` fails with "Gemini API key required" | Set `GEMINI_API_KEY` in `env` or pass `api_key` |
| A tool cannot find a file | Pass absolute paths. A relative path is resolved against the folder the client started the server in, which is often not your project folder. `convert_cad` refuses relative paths outright |
