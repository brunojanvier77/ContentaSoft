# VideoRecompress Studio — MCP Server

VideoRecompress Studio exposes 5 video compression tools via the [Model Context Protocol](https://modelcontextprotocol.io/) (MCP).

## Getting Started

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

## Protocol Details

| Property | Value |
|----------|-------|
| Transport | stdio (stdin/stdout) |
| Protocol | JSON-RPC 2.0 |
| Protocol Version | `2024-11-05` |
| Server Name | `videorecompress-studio` |
| Server Version | `2026.1.0` |

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
| `output_path` | string | No | Output file path (default: auto-generated) |
| `preset` | string | No | Preset ID: `phone_archive`, `youtube_raw`, `security_archive`, `wedding_archive`, `max_savings`, `quick_h264`, `web_optimized` |
| `codec` | string | No | `h264`, `h265`, `av1`, `vp9` (overrides preset) |
| `crf` | integer | No | Quality 0-63 (lower = better, overrides preset) |
| `hw_accel` | string | No | `auto`, `nvenc`, `qsv`, `amf`, `software` (default: auto) |

**Returns**: success, inputPath, outputPath, inputSize, outputSize, reductionPercent, durationMs, integrityOk.

---

### list_presets

List available video recompression presets with codec, quality, and estimated savings.

No parameters required.

**Returns**:

| Preset ID | Codec | CRF | Est. Savings |
|-----------|-------|-----|-------------|
| `phone_archive` | H.265 | 23 | 40-50% |
| `youtube_raw` | H.265 | 20 | 30-40% |
| `security_archive` | H.265 | 28 | 50-60% |
| `wedding_archive` | H.265 | 18 | 25-35% |
| `max_savings` | AV1 | 35 | 55-70% |
| `quick_h264` | H.264 | 23 | 20-30% |
| `web_optimized` | VP9 | 30 | 45-55% |

---

### estimate_savings

Estimate file size savings for a video with a given recompression profile.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Absolute path to the video file |
| `preset` | string | No | Preset ID to estimate with |
| `codec` | string | No | Target codec (if not using preset) |
| `crf` | integer | No | Target CRF (if not using preset) |

**Returns**: currentSize, estimatedOutputSize, estimatedReductionPercent, sourceCodec, targetCodec.

---

### batch_recompress

Batch recompress all videos in a directory.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `input_dir` | string | Yes | Directory to scan for videos |
| `output_dir` | string | Yes | Output directory |
| `preset` | string | No | Preset ID |
| `codec` | string | No | Target codec |
| `crf` | integer | No | Quality value |
| `workers` | integer | No | Parallel workers (default: 2) |

**Returns**: total, success, failed, outputs, errors.
