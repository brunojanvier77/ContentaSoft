---
name: contenta-video
description: Recompress and optimize video files using the VideoRecompress CLI. Use when the user asks to compress videos, reduce file sizes, convert video codecs (H.265, AV1), or batch process video folders.
allowed-tools: Bash
---

# Contenta Video Processing

You have access to the `videorecompress` CLI for video compression and optimization. All commands support `--json` for structured output.

## Commands

### Analyze a video
```bash
videorecompress analyze <input> --json
```
Returns codec, resolution, bitrate, duration, file size, and audio info.

### Recompress a single video
```bash
videorecompress recompress <input> --output <dir> --preset <preset_id>
```
Or with manual settings:
```bash
videorecompress recompress <input> --output <dir> --codec h265 --crf 23 --hw-accel auto
```

### Batch recompress
```bash
videorecompress batch <input-dir> --output <dir> --preset <preset_id> --workers 2
```

### List presets
```bash
videorecompress presets --json
```

### Watch folder
```bash
videorecompress watch --folder <dir> --preset <preset_id>
```

## Built-in Presets

| Preset ID | Codec | CRF | Best For | Est. Savings |
|-----------|-------|-----|----------|-------------|
| `phone_archive` | H.265 | 23 | Home video archival | 40-50% |
| `youtube_raw` | H.265 | 20 | Content creators | 30-40% |
| `security_archive` | H.265 | 28 | Surveillance footage | 50-60% |
| `wedding_archive` | H.265 | 18 | Videographer masters | 25-35% |
| `max_savings` | AV1 | 35 | Maximum compression | 55-70% |
| `quick_h264` | H.264 | 23 | Fast, compatible | 20-30% |
| `web_optimized` | VP9 | 30 | Web delivery | 45-55% |

## Guidelines

- Always run `analyze` first to understand the input before choosing a preset.
- For archival, recommend `phone_archive` (H.265) as the best balance of quality and savings.
- For maximum savings where encoding time doesn't matter, use `max_savings` (AV1).
- For quick results with wide compatibility, use `quick_h264`.
- GPU acceleration is used automatically when available (NVIDIA NVENC, Intel QSV, AMD AMF). Use `--hw-accel software` to force CPU encoding.
- Report estimated and actual savings to the user after compression.
- Trial users get a watermark on output after 10 files. No other restrictions.
