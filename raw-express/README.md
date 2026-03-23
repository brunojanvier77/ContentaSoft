# RAW Express

Batch RAW photo converter for Windows. Convert RAW files from 600+ camera models to JPG, PNG, WebP, TIFF, or AVIF with full control over white balance, exposure, and denoising.

[Download Free Trial](https://contenta-software.com/rawexpress/)

## Features

- **600+ camera models** — Canon (CR2, CR3), Nikon (NEF), Sony (ARW), Fujifilm (RAF), DNG, and many more
- **White balance control** — Camera WB, Auto WB, or Manual color temperature (2000-10000K)
- **Exposure compensation** — -3 to +3 stops
- **Denoising** — wavelet-based noise reduction with adjustable strength
- **Sharpening** — post-conversion sharpening
- **Batch processing** — convert entire shoots with parallel workers
- **AI upscaling** — Real-ESRGAN 2x/4x
- **Output formats** — JPEG, TIFF, PNG, WebP, AVIF

## CLI Reference

**Executable**: `rawexpress`

**Global options**: `--json`, `--quiet`, `-v`/`--verbose`

| Command | Description |
|---------|-------------|
| `convert` | Convert a single RAW file |
| `batch` | Batch convert a directory of RAW files |
| `info` | Show RAW file info and metadata |
| `formats` | List supported RAW formats |
| `upscale` | AI upscale output with Real-ESRGAN |
| `watch` | Watch folder for auto-conversion |
| `profile` | Show user/license info |
| `register` | Register a license key |
| `serve` | Start the MCP server |

## CLI Examples

### Convert a single RAW file

```bash
rawexpress convert photo.cr2 --format webp --quality 90 --white-balance camera
```

### Batch convert a wedding shoot

```bash
rawexpress batch --input ./raw-photos --output ./converted --format jpg --quality 95
```

### Convert with manual white balance and denoising

```bash
rawexpress convert photo.nef --format tiff --white-balance manual --color-temperature 5500 --denoise --denoise-level 50
```

## Supported RAW Formats

Canon (CR2, CR3, CRW), Nikon (NEF, NRW), Sony (ARW, SRF, SR2), Fujifilm (RAF), Olympus (ORF), Panasonic (RW2), Pentax (PEF, DNG), Samsung (SRW), Sigma (X3F), Leica (RWL, DNG), Hasselblad (3FR, FFF), Phase One (IIQ), Adobe DNG, and many more.
