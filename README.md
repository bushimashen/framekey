# FrameKey · Video Background Remover & Frame Player

> Split a video into frames, key out the background, and export transparent PNGs, a sprite sheet, or an alpha-channel WebM — entirely in your browser. Nothing is uploaded.

**English** | [简体中文](README.zh-CN.md)

<a href="https://bushimashen.github.io/framekey/"><b>▶ Live demo</b></a>

## Why it exists

If you build UI motion, character animation or sticker assets, you often get a clip shot on a **green screen or a flat colored background** and need it as: a folder of transparent PNGs, one sprite sheet plus a JSON of rects, or a transparent video you can drop straight into a page. Most online tools want an upload, an account and a watermark — and frame-by-frame keying really needs to be inspected frame by frame.

FrameKey puts the whole loop in a single HTML file: drop a video → extract frames → click the background color once → apply to every frame → export.

## Features

| Step | What you get |
|---|---|
| 1 Upload | Pick, drag-and-drop or paste a video; duration, resolution and frame rate are read automatically |
| 2 Extract | Adjustable target FPS (with a built-in "estimate real frame rate" helper), quality preset `480p / 720p / 1080p`; frames land in a thumbnail strip, click any thumb to jump |
| 3 Key out | **Eyedropper** on the background color, or **Auto** sample from the current frame's border; tolerance, edge feather, spill removal and speckle cleanup all preview live while you drag |
| 4 Play / export | `Space` to play, `←` `→` to step, draggable scrubber; export current frame PNG, all frames as ZIP, sprite sheet + JSON, or a transparent-background WebM |

- Every parameter previews in real time — no waiting for a full re-run per tweak
- Extracted frames are cached in memory, so re-visiting an already-keyed frame is instant
- Export filenames carry a timestamp so repeat exports never overwrite each other

## Usage

Download `index.html` and double-click it. No install, no build step, no network access.

> Some browsers restrict `MediaRecorder` / `crypto.subtle` under `file://`. If "record WebM" is unavailable, serve it locally instead:
>
> ```bash
> python -m http.server 8000
> # then open http://localhost:8000
> ```

## How it works

- **Keying is done in YCbCr, not raw RGB.** Both the key color and each pixel are converted to YCbCr and compared by squared Cb/Cr distance; luma carries little weight, so unevenly lit green screens still key cleanly.
- **Soft edges.** Pixels whose distance falls in `[T, T+F]` get a proportional alpha between 0 and 1 instead of a hard cut, which kills the jaggies. `F` is 15% of the tolerance (min 2) so the two stay linked.
- **Spill removal, feathering and speckle cleanup are flat-array pixel passes** over pre-allocated `Float32Array` buffers — no per-frame allocation, so 1080p frames stay smooth while parameters are dragged.
- **Speckle cleanup is a 3×3 majority vote**: only pixels disagreeing with the neighborhood majority are flattened. Isolated noise and pinholes disappear; large semi-transparent regions are left alone.
- **Both ZIP and WebM are hand-written muxers.** `buildZip` computes CRC32 and writes local file headers plus the central directory itself; `buildAlphaWebm` assembles EBML/MKV elements (`mkvEl` / `vint` / `uintBytes` / `f64Bytes`) around VP9 to produce a genuinely transparent video. The file has zero external dependencies.
- Frames decode on demand, with `prefetch` pulling the next one ahead, so scrubbing never blocks the main thread.

## License

MIT — see [LICENSE](LICENSE).
