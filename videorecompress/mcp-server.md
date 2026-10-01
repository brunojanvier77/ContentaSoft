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

In Claude Code on Windows one command adds it, and nothing else needs installing because the client calls the installed `videorecompress.exe` directly:

```powershell
claude mcp add videorecompress -- videorecompress serve
```

If your AI client cannot find `videorecompress`, use the full path, for example `"C:\\Users\\<you>\\AppData\\Local\\Programs\\VideoRecompressStudio\\videorecompress.exe"`. See the [MCP config guide](../mcp-config/) for each client.

Paths can be absolute or relative. A relative path is resolved against the server's working folder, which is the folder your AI client started it in; use absolute paths when you do not know that folder.

## Protocol

| Property | Value |
|----------|-------|
| Transport | stdio, newline-delimited JSON-RPC 2.0 |
| Protocol versions | `2025-11-25`, `2025-06-18`, `2025-03-26`, `2024-11-05`. The server answers with the version the client asks for; for any other version it answers `2025-11-25` |
| Capabilities | `tools` only (no resources or prompts) |
| Server name | `videorecompress-studio` |
| Server version | `2026.2.22` |

All four ContentaSoft servers run on the same host, which behaves like this:

- **stdout carries JSON-RPC only.** Logs and anything else go to stderr, so a client can parse every line it reads.
- **`ping` is answered while a tool runs**, so a long encode does not look like a hung server.
- **`notifications/cancelled` cancels a running call.** The encode stops, its ffmpeg process ends, and, as the specification says, the call gets no reply.
- **Tool calls run one at a time**, in the order they arrive. `ping`, `tools/list` and cancellations are handled in between.
- **Errors**: an unknown tool or method is a JSON-RPC error (`-32602`, `-32601`); a tool that cannot do its job returns a normal result with `isError: true` and a sentence that names the argument and what it accepts. A call missing a required argument says which one.
- **Every tool has a `title` and `annotations`** (`readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint`); see the next section.

**Trial**: the server never refuses to start, and the trial has no end date. This computer's first 10 files are unrestricted (lifetime, plus 10 after the newsletter confirmation in the app). After that, `recompress_video` and `batch_recompress` output carries a watermark and is cut at 10 minutes. Register with `videorecompress register <email> <key>`.

## Tool annotations

| Tool | Title | Read-only | Destructive | Network |
|------|-------|:---------:|:-----------:|:-------:|
| `analyze_video` | Analyze video | yes | no | no |
| `recompress_video` | Recompress video | no | no | no |
| `list_presets` | List presets | yes | no | no |
| `estimate_savings` | Estimate space savings | yes | no | no |
| `batch_recompress` | Batch recompress videos | no | no | no |

The two writing tools create new files, and a source video is never replaced: an `output_path` that points at the source is diverted to a new name. No tool uses the network. The three read-only tools are also marked idempotent.

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
| `path` | string | Yes | Path to the video |

**Returns**: path, fileName, fileSize, fileSizeFormatted, duration, durationFormatted, videoCodec, videoCodecLong, width, height, resolution, frameRate, frameCount, videoBitrate, pixelFormat, audioCodec, audioChannels, audioSampleRate, audioBitrate, totalBitrate, containerFormat, creationDate (when the file has one).

A file with no audio and no video stream, such as a text file renamed `.mp4`, returns an error (`isError: true`) instead of a result full of zeros.

---

### recompress_video

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `input_path` | string | Yes | Path to the source video |
| `output_path` | string | No | Output file path (default: `<name>_recompressed.<ext>` next to the source). A `.mp4`, `.mkv` or `.webm` extension picks the container. A folder is refused |
| `preset` | string | No | Preset ID |
| `codec` | string | No | `h264`, `h265`, `av1`, `vp9` (overrides the preset) |
| `crf` | integer | No | 0-63, lower = better (overrides the preset) |
| `hw_accel` | string | No | `auto`, `nvenc`, `qsv`, `amf`, `software` |

Arguments are checked before anything is encoded. An unknown `codec` or `hw_accel`, a `crf` outside 0-63 and a non-integer number are errors that say which argument was wrong; none of them falls back to a default. H.264 or H.265 into a `.webm` path is refused (WebM takes VP9 or AV1). The presets `gif_creator`, `youtube_thumbnails` and `video_contact_sheet` write several files, so this tool refuses them; use `batch_recompress`.
**Returns**: success, inputPath, outputPath, inputSize, outputSize, reductionPercent, codec, crf, durationMs, integrityOk, audioOnly, keptOriginal; integrityError when the output check fails. `keptOriginal` is true when re-encoding would have made the file bigger and the original video was kept. Presets with a fixed output size never enlarge a smaller video.

---

### list_presets

No parameters.

**Returns**: an array of presets with id, name, description, targetSegment, estimatedSavingsRange, codec, crf, encoderPreset, isCustom. Includes custom presets saved in the desktop app.

---

### estimate_savings

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Path to the video |
| `preset` | string | No | Preset ID |
| `codec` | string | No | Target codec when no preset is given |
| `crf` | integer | No | Target CRF 0-63 when no preset is given (default 23) |

**Returns**: path, currentSize, currentSizeFormatted, estimatedOutputSize, estimatedOutputSizeFormatted, estimatedReductionPercent, codec, crf.

Example (a 6.8 MB 720p H.264 clip with `phone_archive`): `estimatedOutputSize` 4,922,636 bytes, `estimatedReductionPercent` 30.7, `codec` "H265", `crf` 23. The actual `recompress_video` result on the same clip was a 20% reduction. The estimate is a guide; the result depends on how the source was encoded.

---

### batch_recompress

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `input_dir` | string | Yes | Folder with the videos |
| `output_dir` | string | Yes | Output folder |
| `preset` | string | No | Preset ID |
| `codec` | string | No | Target codec when no preset is given |
| `crf` | integer | No | 0-63, when no preset is given |
| `workers` | integer | No | Parallel files, 1 or more (default 1 for software encoding, 2 for hardware) |

**Returns**: total, success, failed, skipped, unreadable, inputDir, outputDir, outputs, errors.

- `skipped` counts videos that were left alone because they already use the target codec. A folder of already-encoded videos answers `total: 0, skipped: N`, so you can tell it from an empty folder.
- `unreadable` counts files that could not be analyzed at all, so they were never tried.
- Each entry in `errors` has `source` and `message`.

## Example requests

> "How much would phone_archive save on C:\Videos\party.mp4?"
>
> The agent calls `estimate_savings` with `path` and `preset: "phone_archive"`, then `recompress_video` if you agree.

> "Compress everything in C:\Dashcam for archiving."
>
> The agent calls `batch_recompress` with `input_dir: "C:\\Dashcam"`, `output_dir: "C:\\Dashcam\\archived"`, `preset: "dashcam_archive"`.
