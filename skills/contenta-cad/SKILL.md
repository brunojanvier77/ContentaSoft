---
name: contenta-cad
description: Convert 3D CAD and mesh files with the 3D CAD Converter CLI (cadconvert). Use when the user asks to convert STEP, IGES or BREP to STL, OBJ, 3MF, glTF/GLB, FBX, PLY or other formats, convert between mesh formats (including USD/USDZ and VRML input), change units, control mesh quality for 3D printing, reduce the polygon count of a mesh (including UV-preserving glTF/GLB decimation), set the up axis for game engines, optimize glTF/GLB for the web and games (meshopt compression, KTX2 textures, texture map stripping, LOD files), bake offline game assets (HDR/EXR sky to Babylon .env v2, far-card impostor atlases, seamless PBR normal/ORM/KTX2 textures, per-vertex AO and ground contact-shadow decals), inspect a 3D file, write a glTF/GLB's textures out as images, render PNG thumbnails of a model, or batch-convert or watch a folder of CAD files. Windows 10/11 only (no Mac or Linux version).
allowed-tools: Bash
---

# 3D CAD Converter

Use the `cadconvert` CLI (3D CAD Converter 1.0.35+, Windows). Default per-user install: `%LOCALAPPDATA%\Programs\CadConverter\cadconvert.exe`, on the user PATH. Check with `cadconvert --version` (it prints the build id after a `+`). Full reference: https://github.com/brunojanvier77/ContentaSoft/blob/main/cad-converter/cli.md

Every command accepts `--json` (one JSON object on stdout), `--plain`, `--quiet`, `-v/--verbose` and `--progress`. A conversion that writes several part files lists the extra ones in `additionalOutputs`; side files such as `.bin` or `.mtl` are not listed. When stdout is not a terminal (or with `--plain`) the text output is plain `Key: Value` lines and ASCII tables, and every number uses a `.` decimal point with no thousands separator.

## Running commands

- **MCP first:** when this app's MCP tools are connected (`convert_cad`, `get_file_info`, `list_formats` ...), use them for what they cover and the command line for the rest. They take paths and options as named fields, so there is no quoting or wildcard to get wrong. Give them full paths.
- **One command per call:** run one `cadconvert` command at a time, with no `cd`, `&&`, `;`, pipes, loops, variables or `$(...)`. Anything else on the line can stop the run to ask the user for approval, and the CLI already handles folders and reports its results (below).
- **Let the CLI do the folder work:** `cadconvert batch <folder> -o <output-folder>` converts every 3D file in the folder and its subfolders (`--include "*.step"` picks the files, `--skip-existing` makes a re-run do only what is missing), and both `batch` and `convert -o` create the output folder when it is missing.
- **Read the results, don't measure them:** every `--json` result lists the written files in `outputs`; `convert`, and each `files` row of a `batch`, add `outputBytes` and `durationMs`. No `stat`, `date` or `Get-Item` wrapper is needed. To see what is in a folder, use the agent's own file tools.

## Commands

```bash
cadconvert convert <input> -o <output> [-f <fmt>] [--preset web-game|3d-printing|visualization|engineering|mesh-review] [--quality P|N] [--angular RAD] [--up-axis y|z|unchanged] [--decimate R] [--simplify-mode quality|aggressive] [--max-error E] [--lock-border true|false] [--units mm|cm|in|m|ft] [--repair] [--binary true|false] [--assembly preserve|merge] [--compress] [--max-size MB] [--textures keep|ktx2|webp] [--keep-textures all|base-color|none] [--texture-max-size PX] [--lods 0.5,0.25] [--instancing] [--strict] [--base-color IMG] [--texture-dir DIR]
cadconvert batch <dir> -o <outdir> [-f stl] [-r] [-w N] [--include "*.step"] [--exclude ...] [--report r.csv] [same conversion options]
cadconvert watch <dir> -o <outdir> [-f stl] [-w N] [same conversion options]     # runs until Ctrl+C
cadconvert info <file> --json [--extract-textures <dir>]
cadconvert render <file> --out <png> [--views front,side,top,iso,back,bottom] [--size 512]
cadconvert bake-env <sky.hdr|sky.exr> --out <sky.env> [--size 256] [--samples 64]
cadconvert bake-impostor <model> --out <dir-or-prefix> [--angles 8] [--pitch 8] [--tile-size 256] [--format png|webp] [--emit-glb]
cadconvert bake-pbr <albedo> --out <dir-or-prefix> [--seamless] [--normal-strength 2.0] [--invert-normal-y] [--roughness-min 0.25] [--roughness-max 0.85] [--metallic 0.0] [--ao-strength 1.0] [--invert-roughness] [--ktx2] [--ktx2-quality 8]
cadconvert bake-ao <model> --out <dir-or-prefix> [--samples 32] [--max-distance 0] [--strength 1.0] [--no-vertex-ao] [--no-shadow] [--shadow-size 256] [--shadow-padding 0.25]
cadconvert formats [--input|--output]
cadconvert status --json                       # license block: trial or registered, conversions left
cadconvert register <email> <key>
```

- `-o` can be an existing folder (the file is named after the input).
- No `-o`: `convert` writes next to the input with the `-f` extension (then `-f` is required); `batch` and `watch` write to `<input>_converted` next to the input folder.
- `-f` on `convert` defaults to the output extension; `batch` and `watch` default to `stl`. A `-f` that contradicts the output extension exits 2 (leave `-f` out to take it from the extension).
- `batch` and `watch` are recursive by default (`-r false` for the folder itself only); `-w` defaults to the number of processors. `watch` writes a renamed copy rather than overwriting an existing output. `--overwrite` replaces existing outputs; `--skip-existing` leaves them alone, so a re-run converts only what is missing.
- `--profile file.json` starts from a profile saved by the desktop app; command-line options override it.

## Mesh quality (STEP/IGES/BREP sources only)

`--quality` takes a preset or a linear deflection in mm (e.g. `--quality 0.01`). `--angular` is in **radians** and defaults to the preset's value (0.5 with a numeric `--quality`).

| Preset | Linear | Angular |
|--------|--------|---------|
| draft | 1.0 | 5.0 |
| standard (default) | 0.1 | 0.5 |
| fine (smooth curves, 3D printing) | 0.01 | 0.1 |
| ultrafine | 0.001 | 0.05 |

Finer settings make much larger files: on a small test part, fine produced about 12x the triangles of standard. Mesh-to-mesh conversions ignore these options.

## Units

Sources that declare a unit are read at their true size, converted to mm, then written in `--units` (mm when left out):

- STEP, IGES: the file's unit. 3MF: `unit` attribute. glTF/GLB and VRML (`.wrl`, `.vrml`, `.wrz`): always metres (a 1 m VRML cube is 1000 mm). FBX: `UnitScaleFactor`. Collada: `<unit>` (metres when absent). USD/USDZ: `metersPerUnit` (centimetres when absent).
- BREP, STL, OBJ, PLY, OFF, X3D, AMF, DXF, DWG: no unit; the numbers are taken as mm. The CLI has no source-unit option.
- glTF, FBX and Collada outputs declare their unit and up axis; a glTF output opens at its true size in metres. VRML outputs declare their unit with a root `scale 0.001` (millimetre numbers, scaled to metres as the VRML specification requires).

## Up axis, repair, polygon reduction, size budget

- `--up-axis y` for game engines, glTF and three.js; `z` for CAD and 3D printing. It starts from the up axis the source declares (USD, Collada, FBX, glTF, VRML). A STEP/IGES/BREP re-export is not rotated (the CLI prints a note).
- `--repair` repairs STEP/IGES/BREP surfaces before meshing (broken edges and faces, small gaps). It has no effect on mesh sources (STL, OBJ, PLY ...).
- `--decimate 0.25` (or `25%`) keeps about a quarter of the triangles. Mesh sources only: STL, OBJ, PLY, glTF/GLB, FBX, 3MF, Collada, OFF, VRML, X3D, AMF, DXF. STEP/IGES/BREP are not reduced; lower their `--quality` instead (draft is lightest). DWG and USD are not reduced. Meshes over 2,000,000 triangles are written unreduced. From 1.2.0 a textured mesh keeps its texture coordinates through the reduction (1.1.x dropped them): the seams of the texture layout stay in place, so the result may hold more triangles than asked; compare `trianglesAfter` with `trianglesBefore`. glTF/GLB sources: see `--simplify-mode` below.
- `--max-size 5` (glTF/GLB only, MB): a file over it is written again with fewer triangles until it fits, down to 5% of its triangles. On a glTF/GLB source the textures go first: with `--textures` their size limit is lowered; with textures kept as they are, they become WebP (then smaller) before any triangle goes, and a note says so. One that still does not fit is written at the smallest size and reported as over budget (exit 4).

### Web and games: KTX2 textures, LOD files, instancing

For a browser game or a web viewer, start from the preset (needs 3D CAD Converter 1.1.0 or later):

    cadconvert convert truck.glb -o out\ --preset web-game --lods 50%,25% --strict --json

- `--preset web-game` is GLB with meshopt compression and KTX2 textures up to 2048 px. KTX2 is the one texture format that stays compressed in GPU memory; the viewer needs a KTX2 transcoder and a meshopt decoder (three.js and Babylon.js have both). PNG, JPEG and WebP source textures all become KTX2.
- A glTF or GLB source going to glTF or GLB keeps its animations, skins and material extensions. A unit change, `--up-axis` or `--repair` rebuilds the model and loses them; the result's `notes` say so.
- `--lods 50%,25%` also writes `name.lod1.glb` and `name.lod2.glb`: geometry only, no textures. Each LOD line (and `lods` in `--json`) gives the share of triangles it kept; a mesh that cannot be simplified without visible damage keeps more than asked.
- `--instancing` writes a repeated mesh once; node names are not kept. Not with `--assembly merge`.
- `--strict` fails the file (exit code 5, or 4 in a batch) when something asked for could not be applied. Without it the file is written and `web.applied` is false.
- `cadconvert info model.glb --json` lists each texture (size, format, bytes) and an estimate of GPU texture memory now and as KTX2. It reads compressed files too. `--extract-textures <dir>` (1.1.1+) also writes every texture into that folder (PNG and JPEG as they are, WebP and KTX2 decoded to PNG) and each `gltf.textures[]` entry gets `file`.
- `cadconvert render` (1.1.1+) draws a lit PNG thumbnail per view (default front, side, top, iso; Y up, the model's front facing +Z) of any input format; `views[]` in the JSON lists the files. With several views `--out thumb.png` writes `thumb-front.png` ...; an existing folder gets `<model>-<view>.png`. A render uses no trial file. Use it to look at a model or check lettering on it.
- Asset packs (FBX, OBJ, DAE to glTF/GLB): the textures the source links are put into the GLB (beside a `.gltf`). A link to a file that is not there (often a path on the pack author's PC) is removed and the result's notes name it. `--texture-dir <folder>` looks such a file up by name, or a PNG/JPEG of the same name. `--base-color atlas.png` puts a pack's shared colour atlas on every material that has texture coordinates, with a white base colour. These two options are in builds of 1.1.0 from 17 Nov 2026 on: check `cadconvert convert --help` lists them.
- **Lighter textures and triangle reduction on glTF/GLB (1.2.0+):**
  - `--keep-textures all|base-color|none`: keep every map (default), only the base colour map (`base-color`: the normal, metallic-roughness, occlusion, emissive and extension maps are removed), or no texture at all (`none`). A removed map leaves its average in the material: the emissive, metallic and roughness factors are multiplied by the map's mean, and with `none` the base colour factor takes the average colour of the base colour map. The model keeps its look instead of turning white, glowing or fully metallic. `web.texturesDropped` counts what was removed.
  - `--decimate` on a glTF/GLB source has two modes, `--simplify-mode quality|aggressive`. `quality` (the default since 1.2.0) keeps texture coordinates intact and stops at an error limit (`--max-error 0.001..1.0`, default `0.01`; `--lock-border true|false`, default `true`), so it may keep more triangles than asked: compare `web.achievedRatio` with `web.targetRatio`; `web.stoppedForQuality` is then `true` and `warning` says so ("Kept 80% of the triangles instead of 25% ..."). `aggressive` forces the ratio, as 1.1.x always did. On a textured model the result then carries `web.uvError` (0 to 1); above 0.15 the textures are smeared: `web.applied` is `false`, and with `--strict` nothing is written (exit 5). `--max-size` always forces its size, whatever the mode.
  - A compressed output whose texture coordinates are quantized lists `KHR_texture_transform` in `extensionsRequired`. three.js and Babylon.js load it by default; a loader without that extension refuses the file instead of showing shifted textures. The bytes of such an output differ from a 1.1.x output of the same source.
- **Game asset bakers (1.2.0+, CPU only).** Each writes its files together at the end and lists them in `outputs[]`:
  - `cadconvert bake-env <sky.hdr|sky.exr> --out <sky.env> [--size 64|128|256|512] [--samples 8..512]` (MCP `bake_environment`): an equirectangular HDR or EXR panorama (width twice the height) becomes a Babylon.js `.env` environment: diffuse irradiance plus the prefiltered specular mips Babylon reads. EXR: uncompressed, RLE, ZIPS and ZIP scanline files; other compressions are refused with a message.
  - `cadconvert bake-impostor <model> --out <dir-or-prefix> [--angles 4..16] [--pitch -45..45] [--tile-size 64..1024] [--format png|webp] [--emit-glb]` (MCP `bake_impostor`): views from N angles around the model in one atlas (`.impostor.png` or `.webp`), `.impostor.json` (bounds and grid) and, with `--emit-glb`, a crossed-card `.impostor.glb`, which needs an even `--angles`.
  - `cadconvert bake-pbr <albedo> --out <dir-or-prefix> [--seamless] [--normal-strength 0.1..10.0] [--invert-normal-y] [--roughness-min 0..1] [--roughness-max 0..1] [--metallic 0..1] [--ao-strength 0..2] [--invert-roughness] [--ktx2] [--ktx2-quality 1..10]` (MCP `bake_pbr`): from one colour image, `.albedo.png`, a tangent-space `.normal.png` (OpenGL convention; `--invert-normal-y` for DirectX), `.orm.png` (R occlusion, G roughness, B metallic) and `.pbr.json`. `--ktx2` adds `.ktx2` files: the albedo as sRGB colour, the normal and ORM maps as linear data. The maps are derived from the brightness of the image: a starting point, not a scan.
  - `cadconvert bake-ao <model> --out <dir-or-prefix> [--samples 8..256] [--max-distance D] [--strength 0.1..3.0] [--no-vertex-ao] [--no-shadow] [--shadow-size 64..1024] [--shadow-padding 0..1.5]` (MCP `bake_ao`): `.ao.glb` is a static copy of the model with ambient occlusion in its vertex colours (`COLOR_0`); `.shadow.png` and `.shadow.json` are a soft ground shadow with its footprint. The copy keeps the geometry, the base colour (factor and map), the metallic and roughness factors, the alpha mode and double-sidedness. It has no skins, animations, node names or other texture maps, and `warnings[]` says what a given source loses. Use the copy for static props; for a rigged model take only the shadow (`--no-vertex-ao`).
- Other options: `--texture-mode auto|etc1s|uastc`, `--texture-quality 1..10`, `--texture-pow2`, `--quantize standard|fine|off`, `--compress-fallback`, `--keep-extras`.

## Formats

Write: stl, obj, ply, fbx, dae, 3mf, gltf, glb, wrl (vrml), x3d, off, step (.step/.stp), iges (.iges/.igs), brep.
Read only: amf, dwg, dxf, usd (.usd/.usda/.usdc, text or binary), usdz, and `.wrz` (gzipped VRML; `.wrl` and `.vrml` are read and written).

VRML input: VRML 2.0 and 1.0, read in metres and Y-up. Extrusion, line and point sets, Text, PROTO instances, textures, scripts and animation are not drawn (the message says so). Inline loads only relative local files; web, absolute and network addresses are refused with a warning.

BREP input means OpenCascade `.brep`/`.brp` only (not Parasolid `.x_t` or ACIS `.sat`). It has no unit, so it is read as mm, and it comes in as one merged solid without part names, colours or assembly tree.

Native CAD files do not open directly: SolidWorks (`.sldprt`, `.sldasm`), Fusion (`.f3d`), Inventor (`.ipt`, `.iam`), Creo (`.prt`), CATIA (`.CATPart`), Parasolid (`.x_t`) and ACIS (`.sat`). Ask the user to export the part or assembly as STEP from their CAD program, then convert the STEP file.

## Recipes

- **Previews and deliverables for customers:** GLB with `--up-axis y` for a browser viewer (one file, no side files; add `--max-size` for a web budget); 3MF or STL when the customer prints or machines from a mesh, with `--quality fine` from STEP/IGES so curves stay smooth; STEP when they need exact CAD geometry rather than a mesh, for example from a supplier's IGES. `batch` converts a whole folder: `cadconvert batch ./parts -o ./for-customer -f glb --up-axis y`. Run `cadconvert info` on one part first and tell the user its size in mm.
- **3D printing in a slicer (Bambu Studio, PrusaSlicer, OrcaSlicer, Cura):** convert to 3MF, which keeps the unit and colours, or to STL, which every slicer reads but which has no colour. From STEP/IGES use `--quality fine` so curves print smooth: `cadconvert convert part.step -o part.3mf --quality fine`. For a STEP or IGES source, run `cadconvert info` first and tell the user the size in mm, so a part in the wrong unit is caught before printing. STL, OBJ and PLY carry no unit; ask the user what unit the numbers are in.

## Examples

```bash
cadconvert convert model.step -o model.stl
cadconvert convert model.step -f glb
cadconvert convert model.step -o model_fine.stl --quality fine
cadconvert convert part.step -o part.stl --quality 0.01 --angular 0.1
cadconvert convert model.step -o model_y.glb --up-axis y
cadconvert convert model.step -o model_in.obj --units in --repair
cadconvert convert tower.glb -o tower.stl                  # glTF metres -> true size in mm
cadconvert convert tower_cm.fbx -o tower_fbx.stl           # Blender cm FBX -> mm
cadconvert convert scene.usdz -o scene.glb
cadconvert convert scan.stl -o scan_light.glb --decimate 0.25
cadconvert convert model.stl -o model_ascii.stl --binary false
cadconvert convert part.brep -o part.step
cadconvert convert room.wrl -o room.stl                 # VRML is read in metres: a 1 m cube is 1000 mm
cadconvert batch ./cad-files -o ./meshes -f obj --quality fine --skip-existing --json --progress
cadconvert batch ./cad-files -o ./web -f glb --up-axis y --max-size 5
cadconvert info model.step --json
cadconvert watch ./incoming -o ./converted -f glb --up-axis y
cadconvert convert prop.glb -o prop_web.glb --preset web-game --keep-textures base-color --decimate 0.35 --strict --json   # quality mode: check web.achievedRatio
cadconvert bake-env sky.hdr --out sky.env --size 256 --samples 64 --json
cadconvert bake-impostor spruce.glb --out ./out/spruce --angles 8 --pitch 8 --tile-size 256 --emit-glb --json
cadconvert bake-pbr asphalt.png --out ./out/asphalt --seamless --normal-strength 2.2 --roughness-min 0.35 --roughness-max 0.85 --ktx2 --json
cadconvert bake-ao truck.glb --out ./out/truck --samples 32 --shadow-size 256 --json
```

## JSON output

With `--json` stdout holds exactly one JSON object, also for errors. Read `success` (boolean), `exitCode`, `error` (one sentence or `null`), `errors[]` (one entry per failed file), `warnings[]` and `outputs[]` (the files written; report these paths). Conversions also carry the CLI's `notes` (for example that a STEP source is not reduced by `--decimate`). Trial keys appear when they apply: `trialExport`, `trialNotice`, `freeFilesRemaining`, `buyUrl`. With `--progress`, `batch` and `watch` first print `{"type":"progress","file":...,"completed":N,"total":T}` lines; skip them and parse the last line.

## Exit codes

Same table in all four ContentaSoft CLIs: 0 OK · 1 unexpected error (a bug or a broken install) · 2 invalid arguments (also a badly shaped key, a `-f` that contradicts the output extension, an unknown extension, a file that is not a 3D format; nothing was done) · 3 input not found or unreadable · 4 partial failure (a batch where some files failed, or a glTF/GLB over its `--max-size` budget) · 5 all failed (a failed `convert`) · 6 license or trial refused (also a key the server rejected) · 130 cancelled.

## Guidelines

- Run `cadconvert info` first: it gives the declared unit (STEP, IGES, 3MF, FBX, Collada, USD; glTF and VRML are metres) and, for STEP/IGES/BREP/USD/VRML, the bounding box in mm. A file it cannot read is an error, not a table of zeros. STL, OBJ, PLY and OFF carry no unit and show Unknown.
- Use standard quality unless the user needs smooth curves (fine) or a quick, light preview (draft).
- To shrink a mesh file, use `--decimate`; to shrink a mesh made from STEP/IGES, use a coarser `--quality`.
- STL has no colors or materials; the CLI says so. Use 3MF, OBJ or GLB when color matters.
- glTF (`.gltf`) is JSON plus side files; GLB (`.glb`) is a single binary file and is easier to share.
- MCP: paths must be absolute. `convert_cad` takes `input_path`, `output_path` or `output_dir`, `format`, `quality` (a preset or a linear deflection in mm, e.g. `"0.001"`), `angular` (radians), `up_axis` (`y|z|unchanged`), `decimate` (`0.25` or `"25%"`, mesh sources only), `simplify_mode` (`quality|aggressive`), `max_error`, `lock_border`, `max_size_mb`, `base_color`, `texture_dir`, `units`, `repair`, `binary`, `compress`, `assembly`, `textures`, `keep_textures` (`all|base-color|none`), `texture_mode`, `texture_quality`, `texture_max_size`, `texture_pow2`, `lods`, `instancing`, `quantize`, `compress_fallback`, `keep_extras`, `strict`, `preset`, `overwrite`, `skip_existing`. `get_file_info` matches `info` (`extract_textures_dir` = `--extract-textures`), `render_model` matches `render` (`input_path`, `output_path`, `views`, `size`), `bake_environment` matches `bake-env` (`input_path`, `output_path`, `size`, `samples`), `bake_impostor` matches `bake-impostor` (`input_path`, `output_path`, `angles`, `pitch`, `tile_size`, `format`, `emit_glb`), `bake_pbr` matches `bake-pbr` (`input_path`, `output_path`, `seamless`, `normal_strength`, `invert_normal_y`, `roughness_min`, `roughness_max`, `metallic`, `ao_strength`, `invert_roughness`, `ktx2`, `ktx2_quality`), `bake_ao` matches `bake-ao` (`input_path`, `output_path`, `samples`, `max_distance`, `strength`, `bake_vertex_ao`, `bake_contact_shadow`, `shadow_size`, `shadow_padding`), `list_formats` matches `formats`.
- Trial: 10 conversions at full quality (plus 10 more after the newsletter confirmation in the app), with no end date. After that nothing is blocked and nothing expires: STEP/IGES/BREP to a mesh format is meshed at `draft` whatever `--quality` says, and every export carries a trial note (FBX gets draft only). The CLI prints `Buy:` and a link; registering removes it. Before a batch, run `cadconvert status --json` (MCP: `get_license_status`) and read its `license` object: `state` (`Trial` or `Registered`), `freeFilesRemaining`, `nextOutput` (`clean` or `draft_quality`) and `afterFreeFiles`. When `state` is `Trial` and `nextOutput` is `draft_quality`, or `freeFilesRemaining` is smaller than the number of files, tell the user before you convert that the STEP/IGES/BREP parts past the free conversions will be meshed at Draft quality and every one of those exports will carry a trial note, and that a licence removes both: <https://www.contenta-software.com/3dcadconverter/buy.php>.
