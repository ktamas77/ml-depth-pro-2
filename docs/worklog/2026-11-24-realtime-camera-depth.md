# 2026-11-24 — Depth Pro 2 running locally and live on the laptop camera

**Incident / change:** Checked whether Depth Pro 2 runs locally and in real time on a camera feed; it does, at about 27 fps on the M1 Max with the S model at 640x360.
**Severity / type:** chore (evaluation, no code changes)
**Reference:** `README.md`, `demo/README.md`, https://arxiv.org/abs/2611.26603

---

## Timeline (UTC)

| Time | Event |
|---|---|
| 13:59 | `uv run depth-pro-2 predict -i assets/example.jpg -o out/ -m depthpro2-s` ran on MPS in bfloat16; wrote `out/example-depth.png` |
| 14:00 | Exported `depthpro2-s_480x854_w4.mlpackage` and built the Swift demo (`demo/apple`, 12 s) |
| 14:00 | Model-only timing at 854x480, window 4: 69.9 ms per frame (14 fps), not real time |
| 14:01 | Exported and timed three more S variants (table below) |
| 14:02 | First live launch at 640x360 window 4: window showed 0 fps on every line |
| 14:03 | Cause found in `Camera.swift`: the FaceTime HD Camera reports no focal length, so every frame is dropped. Exported `depthpro2-fov_512x896.mlpackage` and relaunched |
| 14:04 | Live reading: camera 30.0 fps, model 27.2 fps, step 38.5 ms, 10.3% dropped, latency p50 138 ms / p95 156 ms |

## Actions taken

### Local run
- ✅ `uv sync` and the photo prediction work out of the box on the M1 Max (64 GB, macOS 27.2.1, Xcode 27.0)
- ✅ S weights (93 MB) and field-of-view weights (90 MB) downloaded to `~/.cache/torch/hub/checkpoints`
- ❌ The example video (`assets/example.mp4`) was not run; the camera question took priority

### Model-only timing (Core ML, CPU and GPU, S model)
Median of 60 predictions after 10 warm-up, driven from Python with `coremltools`, so it includes a little Python overhead.

| Input size | Window 4 | Single frame |
|---|---|---|
| 640x360 | 33.4 ms (30 fps) | 22.9 ms (44 fps) |
| 854x480 | 69.9 ms (14 fps) | 40.6 ms (25 fps) |

### Live camera demo
- ✅ `./demo/run.sh depthpro2-s_360x640_w4.mlpackage` runs on the FaceTime HD Camera (1280x720 at 30 fps)
- ✅ Focal length estimated by the field-of-view package: 856 px at 1280x720
- ✅ Scene depth range on the color bar: 0.52 m to 4.72 m

## Outstanding work

- [ ] OBS filter question, deferred: how to get the depth output into OBS as a filter or source
- [ ] Try window 2 at 640x360, which may give steadier depth than single frame while holding 30 fps
- [ ] Try the B model and `--compute-units ane`
- [ ] Run the example video in buffered mode
- [ ] Confirm the intended use fits `LICENSE_MODEL` (research-only, non-commercial)

## Notes / observations

- The M1 Max is roughly 2.5x slower than the M4 Max figures in `demo/README.md` (33 ms against 13 ms at 640x360, window 4), so the demo's default 854x480 is not real time here.
- The demo fails quietly without a focal length: the window opens, the camera runs, and the stats stay at 0 fps. The only hint is a pill in the window and one line on stderr. The fix is `uv run --extra coreml python -m demo.export fov`, with the package left next to the model, or `--focal-px`.
- The live step time (38.5 ms) is higher than the model-only time (33.4 ms) because the GPU also handles the camera preprocessing and rendering.
- Exported packages, `out/` and `.venv` are gitignored; the repo had no tracked changes after the session.
