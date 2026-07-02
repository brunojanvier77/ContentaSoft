# VideoRecompress Studio — MCP Server

VideoRecompress Studio exposes 5 video compression tools via the [Model Context Protocol](https://modelcontextprotocol.io/) (MCP).

## Getting Started

```json
{
  "mcpServers": {
    "videorecompress": {
      "command": "C:\\Program Files\\ContentaSoft\\VideoRecompress Studio\\videorecompress.exe",
      "args": ["serve"]
    }
  }
}
```

If the install directory is on your `PATH`, `"command": "videorecompress"` also works.

## Protocol Details

| Property | Value |
|----------|-------|
| Transport | stdio (stdin/stdout) |
| Protocol | JSON-RPC 2.0 |
| Protocol Version | `2024-11-05` |
| Server Name | `videorecompress-studio` |
| Server Version | `2026.2.4` |

**Licensing**: `serve` requires an active trial or a registered license (trial expired → the server refuses to start, exit code 3). Register with `videorecompress register <email> <key>` or in the desktop app. During the trial, the first 3 files (lifetime) are processed without restrictions; after that, `recompress_video`/`batch_recompress` output gets a watermark and a 600-second duration cap.

## Presets

`recompress_video`, `batch_recompress`, and `estimate_savings` accept a `preset` parameter: the **ID** of any of the 24 built-in presets (or a custom preset saved from the GUI). Explicit `codec`/`crf` values override the preset when both are given.

Built-in preset IDs by category:

- **Compression**: `phone_archive`, `youtube_raw`, `security_archive`, `wedding_archive`, `max_savings`, `av1_max_savings`, `quick_h264`, `web_optimized`, `drone_footage`, `screen_recording`, `4k_to_1080p`, `dashcam_archive`, `gaming_clips`, `old_video_rescue`
- **Social media**: `whatsapp`, `email_attachment`, `discord_free`, `twitter_x`, `instagram_reels`, `tiktok`
- **Tools** (no re-encoding): `gif_creator`, `youtube_thumbnails`, `video_contact_sheet`
- **Conversion**: `iphone_to_mp4`

## Tools

### analyze_video

Analyze a video file and return codec, resolution, duration, bitrate, and audio information.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Absolute path to the video file |

**Returns**: fileName, fileSize, videoCodec, resolution, width, height, frameRate, frameCount, duration, videoBitrate, pixelFormat, totalBitrate, containerFormat, audioCodec, audioChannels, audioBitrate, subtitleStreams.

---

### recompress_video

Recompress a video file with modern codecs (H.265, AV1, VP9) for smaller file size.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `input_path` | string | Yes | Absolute path to the source video |
| `output_path` | string | No | Output file path (default: auto-generated next to the source) |
| `preset` | string | No | Built-in preset ID (see [Presets](#presets)), e.g. `phone_archive`, `wedding_archive`, `tiktok` |
| `codec` | string | No | `h264`, `h265`, `av1`, `vp9` (overrides preset) |
| `crf` | integer | No | Quality 0-63 (lower = better, overrides preset) |
| `hw_accel` | string | No | `auto`, `nvenc`, `qsv`, `amf`, `software` (default: auto) |

**Returns**: success, inputPath, outputPath, inputSize, outputSize, reductionPercent, codec, crf, durationMs, integrityOk, integrityError.

---

### list_presets

List available video recompression presets with codec, quality, and estimated savings.

No parameters required.

**Returns**: all 24 built-in presets plus any custom presets saved from the GUI. Highlights:

| Preset ID | Codec | CRF | Est. Savings |
|-----------|-------|-----|-------------|
| `phone_archive` | H.265 | 23 | 40-50% |
| `youtube_raw` | H.265 | 20 | 30-40% |
| `security_archive` | H.265 | 28 | 50-60% |
| `wedding_archive` | H.265 | 18 | 25-35% |
| `max_savings` | AV1 | 35 | 55-70% |
| `av1_max_savings` | AV1 | 33 | 60-75% |
| `quick_h264` | H.264 | 23 | 20-30% |
| `web_optimized` | VP9 | 30 | 45-55% |
| `drone_footage` | H.265 | 21 | 50-65% |
| `screen_recording` | H.265 | 26 | 55-70% |
| `4k_to_1080p` | H.265 | 22 | 65-80% |
| `dashcam_archive` | H.265 | 27 | 50-60% |
| `gaming_clips` | H.265 | 22 | 45-60% |
| `old_video_rescue` | H.265 | 20 | 40-60% |
| `whatsapp` | H.264 | 28 | 60-75% |
| `email_attachment` | H.264 | 28 | 55-70% |
| `discord_free` | H.264 | 32 | 70-85% |
| `twitter_x` | H.264 | 23 | 30-45% |
| `instagram_reels` | H.264 | 23 | 30-45% |
| `tiktok` | H.264 | 23 | 30-45% |
| `gif_creator` | Copy | — | N/A |
| `youtube_thumbnails` | Copy | — | N/A |
| `video_contact_sheet` | Copy | — | N/A |
| `iphone_to_mp4` | H.264 | 20 | 10-25% |

---

### estimate_savings

Estimate file size savings for a video with a given recompression profile.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Absolute path to the video file |
| `preset` | string | No | Built-in preset ID to estimate with |
| `codec` | string | No | Target codec (used when no preset specified) |
| `crf` | integer | No | Target CRF 0-63 (used when no preset specified, default: 23) |

**Returns**: currentSize, estimatedOutputSize, estimatedReductionPercent, sourceCodec, targetCodec.

---

### batch_recompress

Batch recompress all videos in a directory.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `input_dir` | string | Yes | Directory to scan for videos |
| `output_dir` | string | Yes | Output directory |
| `preset` | string | No | Built-in preset ID (see [Presets](#presets)) |
| `codec` | string | No | Target codec (used when no preset specified) |
| `crf` | integer | No | Quality 0-63 (used when no preset specified) |
| `workers` | integer | No | Parallel workers (default: 1 for software encoding, 2 for hardware) |

**Returns**: total, success, failed, outputs, errors.
