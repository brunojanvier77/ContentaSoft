# VideoRecompress Studio: MCP server

`videorecompress serve` exposes 5 video tools over the [Model Context Protocol](https://modelcontextprotocol.io/), so an AI agent (Claude Desktop, Claude Code, Cursor, Windsurf and others) can analyze and compress videos on your PC.

## Setup

```json
{
  "mcpServers": {
    "videorecompress": {
      "command": "videorecompress",
      "args": ["serve"]
    }
  }
}
```

If your AI client cannot find `videorecompress`, use the full path, for example `"C:\\Users\\<you>\\AppData\\Local\\Programs\\VideoRecompressStudio\\videorecompress.exe"`. See the [MCP config guide](../mcp-config/) for each client.

Pass absolute paths to every tool.

## Protocol

| Property | Value |
|----------|-------|
| Transport | stdio |
| Protocol | JSON-RPC 2.0, MCP `2024-11-05` |
| Server name | `videorecompress-studio` |
| Server version | `2026.2.19` |

**License**: the server runs during the 30-day trial and for registered copies. After the trial ends it refuses to start (exit code 3). During the trial the first 10 files are unrestricted; after that, `recompress_video` and `batch_recompress` output carries a watermark and is cut at 10 minutes.

## Presets

`recompress_video`, `batch_recompress` and `estimate_savings` take a `preset` parameter: the ID of a built-in preset or of a custom preset saved in the desktop app. An explicit `codec` or `crf` overrides the preset's value.

- **Compression**: `phone_archive`, `youtube_raw`, `security_archive`, `wedding_archive`, `max_savings`, `av1_max_savings`, `quick_h264`, `web_optimized`, `drone_footage`, `screen_recording`, `4k_to_1080p`, `dashcam_archive`, `gaming_clips`, `old_video_rescue`
- **Social**: `whatsapp`, `email_attachment`, `discord_free`, `twitter_x`, `instagram_reels`, `tiktok`
- **Conversion**: `iphone_to_mp4`
- **Desktop-app tools** (not re-encodes): `gif_creator`, `youtube_thumbnails`, `video_contact_sheet`

Codec, CRF and estimated savings for each are in the [CLI README](README.md#presets) and in `list_presets`.

## Tools

### analyze_video

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Absolute path to the video |

**Returns**: path, fileName, fileSize, fileSizeFormatted, duration, durationFormatted, videoCodec, videoCodecLong, width, height, resolution, frameRate, frameCount, videoBitrate, pixelFormat, audioCodec, audioChannels, audioSampleRate, audioBitrate, totalBitrate, containerFormat, creationDate (when the file has one).

---

### recompress_video

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `input_path` | string | Yes | Absolute path to the source video |
| `output_path` | string | No | Output file path (default: generated next to the source) |
| `preset` | string | No | Preset ID |
| `codec` | string | No | `h264`, `h265`, `av1`, `vp9` (overrides the preset) |
| `crf` | integer | No | 0-63, lower = better (overrides the preset) |
| `hw_accel` | string | No | `auto`, `nvenc`, `qsv`, `amf`, `software` |

**Returns**: success, inputPath, outputPath, inputSize, outputSize, reductionPercent, codec, crf, durationMs, integrityOk, audioOnly, keptOriginal; integrityError when the output check fails.

---

### list_presets

No parameters.

**Returns**: an array of presets with id, name, codec, crf, target, estimatedSavings, isCustom. Includes custom presets saved in the desktop app.

---

### estimate_savings

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Absolute path to the video |
| `preset` | string | No | Preset ID |
| `codec` | string | No | Target codec when no preset is given |
| `crf` | integer | No | Target CRF 0-63 when no preset is given (default 23) |

**Returns**: path, currentSize, currentSizeFormatted, estimatedOutputSize, estimatedOutputSizeFormatted, estimatedReductionPercent, codec, crf.

Example (a 443 KB clip with `phone_archive`): `estimatedOutputSize` 228,718 bytes, `estimatedReductionPercent` 49.6, `codec` "H265", `crf` 23. The actual `recompress_video` result on the same clip was a 45% reduction.

---

### batch_recompress

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `input_dir` | string | Yes | Folder with the videos |
| `output_dir` | string | Yes | Output folder |
| `preset` | string | No | Preset ID |
| `codec` | string | No | Target codec when no preset is given |
| `crf` | integer | No | 0-63, when no preset is given |
| `workers` | integer | No | Parallel files (default 1 for software encoding, 2 for hardware) |

**Returns**: total, success, failed, inputDir, outputDir, outputs, errors.

## Example requests

> "How much would phone_archive save on C:\Videos\party.mp4?"
>
> The agent calls `estimate_savings` with `path` and `preset: "phone_archive"`, then `recompress_video` if you agree.

> "Compress everything in C:\Dashcam for archiving."
>
> The agent calls `batch_recompress` with `input_dir: "C:\\Dashcam"`, `output_dir: "C:\\Dashcam\\archived"`, `preset: "dashcam_archive"`.
