---
name: contenta-video
description: Compress and re-encode video files with the VideoRecompress Studio CLI (videorecompress). Use when the user asks to shrink videos, convert to H.265/HEVC, AV1 or VP9, prepare videos for WhatsApp, Discord or email, archive phone, wedding, dashcam or screen recordings, batch-compress a folder, or watch a folder for new videos.
allowed-tools: Bash
---

# VideoRecompress Studio (video compression)

Use the `videorecompress` CLI (VideoRecompress Studio 2026.2.24+, Windows). Default per-user install: `%LOCALAPPDATA%\Programs\VideoRecompressStudio\videorecompress.exe`, on the user PATH. Check with `videorecompress --version` (it prints the build id after a `+`).

Relative and full paths both work for inputs, `--output`, `--profile` and watermark images.

Every command accepts `--json` (JSON on stdout, logs on stderr), `--quiet`, `-v`.

## Commands

```bash
videorecompress analyze <file> [--json]
videorecompress recompress <file> --output <dir> [--preset-id ID | --codec C --crf N] [options]
videorecompress batch <dir> --output <dir> [--preset-id ID | --codec C --crf N] [--workers 2] [--include "*.mp4"]
videorecompress presets [--json] [--preset-id ID]
videorecompress profile save|load|validate <file.json>
videorecompress watch --folder <dir> --preset <ID> [--output <dir>]     # runs until Ctrl+C, scans every 30 s
videorecompress register <email> <key>
videorecompress status                       # license, clean files left, tools, GPU encoders
```

`--output` is a folder; outputs keep the source name.

## Presets: two different flags

- `--preset-id <id>` = named preset (recompress, batch, profile save). Explicit flags override single settings.
- `--encoder-preset` = encoder speed (`ultrafast`..`veryslow`, or 0-13 for SVT-AV1; default `medium`). There is no `--preset` on recompress/batch: it exits 2 with a hint (2026.2.22 silently took it as the encoder speed).
- `watch` uses `-p/--preset <id>` for the named preset.

Pick the preset by what the footage is:

| Footage | `--preset-id` | Codec / CRF |
|------|---------------|-------------|
| Drone and action-cam footage (high-bitrate DJI, GoPro) | `drone_footage` | H.265 / 21 |
| Phone/home video archive | `phone_archive` | H.265 / 23 |
| iPhone `.mov` that must play everywhere | `iphone_to_mp4` | H.264 / 20 |
| Raw footage you will edit later | `youtube_raw` | H.265 / 20 |
| Old DivX, WMV, MPEG-2 or AVI files | `old_video_rescue` | H.265 / 20 |
| Gaming recordings | `gaming_clips` | H.265 / 22 |
| Wedding/event archive (high quality) | `wedding_archive` | H.265 / 18 |
| Maximum savings | `max_savings` | AV1 / 35 |
| Surveillance | `security_archive` | H.265 / 28 |
| Dashcam / body cam | `dashcam_archive` | H.265 / 27 |
| Screen recordings | `screen_recording` | H.265 / 26 |
| Fast, plays everywhere | `quick_h264` | H.264 / 23 |
| Web (WebM) | `web_optimized` | VP9 / 30 |
| 4K down to 1080p | `4k_to_1080p` | H.265 / 22 |
| WhatsApp / email / Discord | `whatsapp`, `email_attachment`, `discord_free` | H.264 / 28-32 |

Full list: `videorecompress presets --json` (24 presets).

Before a large batch, recompress one typical file with `--measure-quality` and report its size saving (`reductionPercent`) and VMAF score (`quality.vmaf` in `--json`). Do this first whenever the user asks to keep the quality ("without it looking worse", "no visible loss"): a VMAF of about 93 or more usually means no visible difference. If the score is lower, try a lower `--crf` on the same file before the batch. On the trial, this test uses one of the free files. When the user wants the archive as close to the original as possible, choose `wedding_archive` (CRF 18) over `phone_archive` (CRF 23).

## Other options (recompress and batch)

`--quality-mode crf|cbr|twopass` + `--bitrate kbps` · `--hw-accel auto|nvenc|qsv|amf|software` · `--audio-mode copy|reencode|remove` · `--audio-codec aac|opus|mp3|flac|ac3|eac3` · `--audio-bitrate` · `--container mp4|mkv|webm` · `--width/--height` (a maximum; smaller videos are not enlarged) · `--trim-mode skip-start|skip-end|keep-first|custom` + `--trim-duration`/`--trim-start`/`--trim-end` (seconds of the source) · `--denoise off|light|medium|strong` · `--deinterlace off|yadif|yadif-bob` · `--watermark-text/--watermark-image` + `--watermark-position TopLeft..BottomRight` · `--fps/--fps-mode` · `--overwrite` · `--profile file.json` · `--measure-quality` (SSIM/VMAF). batch: `-r/--recursive` (default on), `-w/--workers` (default 2), `--include/--exclude`.

Option values are checked: an unknown `--codec`, `--hw-accel`, `--container`, `--quality-mode`, `--audio-*`, `--denoise`, `--deinterlace`, `--trim-mode`, `--fps-mode` or `--watermark-position` exits 2 and names the valid values; `--crf` must be 0-63; `--quality-mode cbr|twopass` needs `--bitrate`; H.264/H.265 cannot go into WebM (use `--codec vp9` or `av1`). The presets `gif_creator`, `youtube_thumbnails` and `video_contact_sheet` write several files and work only with `batch`.

Trim examples: `--trim-mode keep-first --trim-duration 10` keeps the first 10 s; `--trim-mode custom --trim-start 5 --trim-end 12` keeps 0:05-0:12 of the source.

## Examples (verified on 2026.2.24)

```bash
videorecompress analyze video.mp4 --json
videorecompress recompress video.mp4 --output ./compressed
videorecompress recompress video.mp4 --codec h265 --crf 18 --encoder-preset slow --output ./archive
videorecompress recompress video.mp4 --trim-mode keep-first --trim-duration 10 --output ./first10
videorecompress recompress small.3gp --preset-id whatsapp --output ./whatsapp
videorecompress batch ./videos --preset-id phone_archive --workers 2 --output ./compressed
videorecompress batch ./videos --codec av1 --crf 35 --output ./av1 --json
videorecompress watch --folder ./incoming --preset phone_archive --output ./compressed
```

`batch --json` always prints, even for an empty folder: one JSON object per line, `{"type":"progress",...}` objects, then a summary `{"success":true,"cancelled":false,"totalFiles":N,"processed":N,"failed":0,"skipped":0,"durationMs":...,"errors":[]}` (with a `message` when nothing was encoded, for example `All files skipped (same codec as target).` or `No video files found matching criteria.`, both exit 0). When any file fails or cannot be read, `success` is `false`, the exit code is 4 and the file is in `errors`; the other files are still written.

## Exit codes

0 success · 1 general error · 2 invalid arguments (every bad option value, an unknown option such as `--preset`, an unexpected argument; one line plus a `--help` hint, or one `{"success":false,"error":"..."}` object with `--json`) · 3 not used (older versions: trial ended) · 4 recompression failed (also `analyze` or `recompress` on a file that is not media, and a `batch` with a failed file) · 5 file or folder not found · 6 ffmpeg/ffprobe missing.

## Guidelines

- Run `analyze` first; report size before and after.
- The source file is never replaced: an output that would land on the source, or on a file that is in use, is saved under the next free name, and the result says where. Report the path from the result, not the one you asked for.
- If re-encoding would make a file bigger, the source video stream is copied into the output instead (so the file keeps the source codec), and the result says so: `--json` and the MCP results carry `outputCodec` (what is in the file, e.g. `mpeg4`), `streamCopied`, `keptOriginal` and a `warning` such as "H.265 at CRF 18 would be larger than the source (471 KB vs 443 KB); the source video stream (mpeg4) was copied instead". The human output prints the same sentence and `..., mpeg4 stream copy` on the Output line. An explicit `--codec` does not override this guard; a profile with `Safety.KeepOriginalWhenLarger=false` does. Presets with a fixed size (e.g. `whatsapp`, `4k_to_1080p`) never enlarge a smaller video.
- Hardware encoding is automatic; `--hw-accel software` forces the CPU.
- Trial: no end date. This computer's first 10 files (lifetime, plus 10 after the newsletter confirmation in the app) are unrestricted; every later file is watermarked and cut at 10 minutes. `videorecompress status --json` shows how many are left (`freeFilesRemaining`). **A batch spends one free file per video.** Before a batch, run `videorecompress status --json`. When `license.state` is `Trial`, tell the user before you start how many of the videos will come out clean (`freeFilesRemaining`), that the rest will be watermarked and cut at 10 minutes, and that a licence removes both: <https://www.contenta-software.com/videorecompress/buy.php>. Every result says what it did: `recompress --json` and MCP `recompress_video` carry `trialWatermarked`, `trialNotice` (with the buy link) and `freeFilesRemaining`; `batch --json` and `batch_recompress` carry `trialWatermarked` (a count), `trialWatermarkedOf`, `trialNotice` and per-file `files[]` rows with `outputCodec`, `streamCopied`, `keptOriginal`, `trialWatermarked` and `warning`. The human output prints the notice as a yellow line. Registered installs get none of these. Nothing stops working; `videorecompress register <email> <key>` removes the watermark and the cap.
- Numbers in the CLI output and in the `*Formatted` JSON fields use a `.` decimal point whatever the machine locale.
