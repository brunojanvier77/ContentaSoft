# VideoRecompress Studio: MCP server

`videorecompress serve` exposes 7 tools over the [Model Context Protocol](https://modelcontextprotocol.io/), so an AI agent (Claude Code, Claude Desktop, Cursor, Windsurf, Codex and others) can work on files on your PC.

Generated from `videorecompress catalog`, version 2026.2.27. The same reference is on the website: [www.contenta-software.com/videorecompress/docs/cli.html](https://www.contenta-software.com/videorecompress/docs/cli.html). The command line is documented in [cli.md](cli.md).

## Setup

```json
{
  "mcpServers": {
    "videorecompress-studio": {
      "command": "videorecompress",
      "args": ["serve"]
    }
  }
}
```

If your AI client cannot find `videorecompress`, give the full path, for example `"C:\\Users\\<you>\\AppData\\Local\\Programs\\VideoRecompressStudio\\videorecompress.exe"`. The [MCP config guide](../mcp-config/) covers each client. In Claude Code one command adds it:

```powershell
claude mcp add videorecompress-studio -- videorecompress serve
```

Pass absolute paths to every tool.

## Protocol

| Property | Value |
|---|---|
| Transport | stdio, newline-delimited JSON-RPC 2.0 |
| Protocol versions | `2025-11-25`, `2025-06-18`, `2025-03-26`, `2024-11-05`. The server answers with the version the client asks for; for any other version it answers `2025-11-25` |
| Capabilities | `tools` only (no resources or prompts) |
| Server name | `videorecompress-studio` |
| Server version | `2026.2.27` |

All four ContentaSoft servers run on the same host, which behaves like this:

- **stdout carries JSON-RPC only.** Logs go to stderr, so a client can parse every line it reads.
- **`ping` is answered while a tool runs**, so a long job does not look like a hung server.
- **`notifications/cancelled` cancels a running call.** The call stops and, as the specification says, gets no reply.
- **Tool calls run one at a time**, in the order they arrive.
- **Errors**: an unknown tool or method is a JSON-RPC error; a tool that cannot do its job returns a normal result with `isError: true` and a sentence that names the problem.
- **Licence first**: `get_license_status` returns the licence and trial state, so an agent can tell the user what a run will produce before it starts one.

## Tool annotations

| Tool | Title | Read-only | Destructive | Idempotent | Network |
|---|---|---|---|---|---|
| [`batch_recompress`](#batch_recompress) | Batch recompress videos | no | no | no | no |
| [`estimate_savings`](#estimate_savings) | Estimate space savings | yes | no | yes | no |
| [`get_file_info`](#get_file_info) | Get video file info | yes | no | yes | no |
| [`get_license_status`](#get_license_status) | Get license and trial status | yes | no | yes | no |
| [`list_formats`](#list_formats) | List formats | yes | no | yes | no |
| [`list_presets`](#list_presets) | List presets | yes | no | yes | no |
| [`recompress_video`](#recompress_video) | Recompress video | no | no | no | no |

Destructive: the tool can replace a file that already exists. Network: the tool uses the internet.

## Tools

### batch_recompress

**Batch recompress videos.** Batch recompress all videos in a directory.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_dir` | string | yes | Directory containing video files to recompress. |
| `output_dir` | string | yes | Output directory for recompressed files. |
| `preset` | string | no | Preset ID, one of: phone_archive, youtube_raw, security_archive, wedding_archive, max_savings, av1_max_savings, quick_h264, web_optimized, drone_footage, screen_recording, 4k_to_1080p, dashcam_archive, gaming_clips, old_video_rescue, gif_creator, youtube_thumbnails, video_contact_sheet, whatsapp, email_attachment, discord_free, twitter_x, instagram_reels, tiktok, iphone_to_mp4 (custom presets saved in the app work too; list_presets shows them all). |
| `codec` | string | no | Video codec: h264, h265, av1, vp9 (used when no preset specified). |
| `crf` | integer | no | Constant Rate Factor (used when no preset specified). Range: 0 to 63. |
| `workers` | integer | no | Number of parallel workers (default: 1 for software, 2 for hardware). Minimum: 1. |
| `hdr` | string | no | HDR sources (HDR10, HLG): keep (default; stays 10-bit HDR with H.265/AV1/VP9, H.264 is always tone-mapped) or sdr (tone-map to SDR BT.709). Single-layer Dolby Vision is kept only by the H.265 software encoder (hw_accel software) into MP4/MKV without a resize; HDR10+ only by VP9. Otherwise the base HDR10/HLG picture is kept and the result's hdrNote says what was not. |
| `overwrite` | boolean | no | Replace the output when it already exists (default: an existing output is kept and the new file gets a free name). |
| `skip_existing` | boolean | no | Leave an input alone when its output already exists, so a re-run only does what is missing. |

### estimate_savings

**Estimate space savings.** Estimate how much smaller ONE video gets with a preset (or codec/crf) by actually encoding it: three 4-second samples spread over the file (the whole file when it is 20 s or shorter) with the same settings and encoder recompress_video uses, scaled to the full file. Takes a few seconds per file and writes only temporary files. Use it to compare presets on a representative file. Returns estimatedReductionPercent, a low-high range across the samples, the method (full_encode = exact size, sample_encode = typically within a few points, heuristic = a per-codec guess when no sample could be encoded) and the encoder. It does not measure quality: run recompress_video on one file (CLI: recompress --measure-quality for VMAF/SSIM) to check the picture before converting a whole folder.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_path` | string | yes | Absolute path to the video file. |
| `preset` | string | no | Preset ID, one of: phone_archive, youtube_raw, security_archive, wedding_archive, max_savings, av1_max_savings, quick_h264, web_optimized, drone_footage, screen_recording, 4k_to_1080p, dashcam_archive, gaming_clips, old_video_rescue, gif_creator, youtube_thumbnails, video_contact_sheet, whatsapp, email_attachment, discord_free, twitter_x, instagram_reels, tiktok, iphone_to_mp4 (custom presets saved in the app work too; list_presets shows them all). |
| `codec` | string | no | Video codec: h264, h265, av1, vp9 (used when no preset specified). |
| `crf` | integer | no | Constant Rate Factor (used when no preset specified, default: 23). Range: 0 to 63. |

### get_file_info

**Get video file info.** Analyze a video file and return codec, resolution, duration, bitrate, HDR, audio and subtitle information.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_path` | string | yes | Absolute path to the video file. |

### get_license_status

**Get license and trial status.** Get the license and trial state before compressing: registered or trial, clean files left (freeFilesRemaining), and what files become after the free ones (afterFreeFiles: watermark = a watermark and a 10-minute length cap). nextOutput says what the next file will be: clean or watermark.

No parameters.

### list_formats

**List formats.** List the containers and codecs VideoRecompress reads (input) and writes (output), with the video codecs each output container holds.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `direction` | string | no | input = only what it reads, output = only what it writes, all (default) = both (the CLI's formats --input / --output). One of: `input`, `output`, `all`. |

### list_presets

**List presets.** List available video recompression presets with codec, quality, and estimated savings.

No parameters.

### recompress_video

**Recompress video.** Recompress a video file with modern codecs (H.265, AV1, VP9) for smaller file size.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_path` | string | yes | Absolute path to the source video file. |
| `output_path` | string | no | Output file path (default: auto-generated in same directory). |
| `preset` | string | no | Preset ID, one of: phone_archive, youtube_raw, security_archive, wedding_archive, max_savings, av1_max_savings, quick_h264, web_optimized, drone_footage, screen_recording, 4k_to_1080p, dashcam_archive, gaming_clips, old_video_rescue, gif_creator, youtube_thumbnails, video_contact_sheet, whatsapp, email_attachment, discord_free, twitter_x, instagram_reels, tiktok, iphone_to_mp4 (custom presets saved in the app work too; list_presets shows them all). |
| `codec` | string | no | Video codec: h264, h265, av1, vp9 (overrides preset if both specified). |
| `crf` | integer | no | Constant Rate Factor (overrides preset if both specified). Range: 0 to 63. |
| `hw_accel` | string | no | Hardware acceleration: auto, nvenc, qsv, amf, software. |
| `hdr` | string | no | HDR sources (HDR10, HLG): keep (default; stays 10-bit HDR with H.265/AV1/VP9, H.264 is always tone-mapped) or sdr (tone-map to SDR BT.709). Single-layer Dolby Vision is kept only by the H.265 software encoder (hw_accel software) into MP4/MKV without a resize; HDR10+ only by VP9. Otherwise the base HDR10/HLG picture is kept and the result's hdrNote says what was not. |
| `overwrite` | boolean | no | true: replace the output when it exists. false: give the new file a free name instead. Default: an explicit output_path is replaced, the generated name (\<name\>\_recompressed) is never replaced. |
| `skip_existing` | boolean | no | true: when the output already exists, write nothing and answer skipped: true (safe re-runs). |
