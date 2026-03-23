---
name: Contenta Video Processing
description: Recompress and optimize video files using the VideoRecompress CLI — reduce file sizes by 30-70% with modern codecs (H.265, AV1) and GPU acceleration.
tools:
  - Bash
---

# Contenta Video Processing Skill

You have access to the `videorecompress` CLI for video compression and optimization. All commands support `--json` for structured output.

## Commands

### Analyze a video
```bash
videorecompress analyze <input> --json
```
Returns codec, resolution, bitrate, duration, file size, and estimated savings.

### Recompress a single video
```bash
videorecompress recompress <input> --output <file> --preset <preset_id>
```

### Batch recompress a directory
```bash
videorecompress batch --input <dir> --output <dir> --preset <preset_id>
```

### List presets
```bash
videorecompress presets --json
```

### Watch folder for auto-compression
```bash
videorecompress watch --input <dir> --output <dir> --preset <preset_id>
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

## Supported Codecs

- **H.264** — maximum compatibility
- **H.265 (HEVC)** — best quality/size balance
- **AV1** — maximum compression (slower encoding)
- **VP9** — web-optimized

## GPU Acceleration

Hardware encoding is used automatically when available:
- **NVIDIA NVENC** — GeForce/Quadro GPUs
- **Intel QSV** — Intel integrated/Arc GPUs
- **AMD AMF** — Radeon GPUs

Falls back to software encoding if no compatible GPU is detected.

## Guidelines

- Always run `analyze` first to understand the input before choosing a preset.
- For archival, recommend `phone_archive` (H.265) as the best balance of quality and savings.
- For maximum savings where encoding time doesn't matter, use `max_savings` (AV1).
- For quick results with wide compatibility, use `quick_h264`.
- Report estimated and actual savings to the user after compression.
