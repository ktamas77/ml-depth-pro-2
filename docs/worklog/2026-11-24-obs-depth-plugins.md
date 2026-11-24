# 2026-11-24 — Fork with a background cutoff, and two OBS plugins on the same model

**Incident / change:** Published a fork of Depth Pro 2 with a background cutoff in the Mac demo, then built and released two OBS filter plugins that run the model inside OBS.
**Severity / type:** feat
**Reference:** https://github.com/ktamas77/ml-depth-pro-2, https://github.com/ktamas77/depth-background-removal, https://github.com/ktamas77/depth-rgb-trails. Continues `2026-11-24-realtime-camera-depth.md`.

---

## Timeline (UTC)

| Time | Event |
|---|---|
| 14:11 | Mac demo gained `--cutoff METERS`: the camera nearer than the cutoff on green, `[` and `]` to move it |
| 14:13 | Forked `apple-aiml-research/ml-depth-pro-2` to `ktamas77/ml-depth-pro-2`; change pushed to `background-cutoff`, made the fork's default branch |
| 14:46 | First OBS plugin committed: a filter that makes a source transparent behind a cutoff distance |
| 14:49 | `v0.1.0` of it built and released by GitHub Actions |
| 14:52 | Read the OBS forum's resource policy: plugins written largely with AI tools are not accepted there |
| 15:11 | Second OBS plugin committed: Depth RGB Trails, with a preview app that runs without OBS |
| 15:15 | `v0.1.0` of Depth RGB Trails released |
| 15:19 | First plugin renamed from `obs-depth-background-removal` to `depth-background-removal` |
| 15:21 | `v0.1.1` of it released under the new name |

## Actions taken

### Fork (`ktamas77/ml-depth-pro-2`)
- ✅ `--cutoff` in `demo/apple` (`Render.swift`, `Options.swift`, `LiveApp.swift`), checked on the live camera
- ✅ README section on what the fork adds, how to run it, and how to capture it in OBS with a chroma key
- ✅ Focal-length label moved off the green pane, so a window capture of it is clean

### Depth Background Removal (`ktamas77/depth-background-removal`)
- ✅ Standalone repo with Apple's history; GitHub allows one fork of a repository per account
- ✅ `plugin/plugin.m`: Objective-C against the OBS C API, Core ML on a background queue; `plugin/depth_mask.effect`: the shader
- ✅ Build without building OBS: headers from the `obs-studio` tag, linked against the `libobs` in `OBS.app`; 6 s locally, 24 s in CI
- ✅ Workflow builds on every push and pull request and publishes a release for a tag `v*`
- ✅ Released build checked in OBS 33.0.3 over obs-websocket: camera cut out at a 1 m cutoff, model 28 to 41 ms per frame at 640x360, OBS at 30 fps
- ✅ Renamed away from the `obs-` prefix; the filter's id is unchanged, so saved scenes keep working
- ❌ Not listed on the OBS forum: its policy excludes plugins written largely with AI tools

### Depth RGB Trails (`ktamas77/depth-rgb-trails`)
- ✅ Filter that trails the foreground with red, blue and green copies one, two and three delays old, over the background, which can be blurred
- ✅ Keeps the last 96 frames at the model's size, with the foreground as alpha; two shader techniques, `Store` and `Draw`
- ✅ Checked in OBS: after forcing everything from foreground to background, the picture went red, blue, green for one delay each, then back to normal
- ✅ `demo/trails.sh`: the Mac demo with a `--trails` mode, the same layers from the webcam without OBS
- ✅ `v0.1.0` released; the released build re-checked in OBS

## Outstanding work

- [ ] Choosing the model through OBS's file dialog is untested; the checks set it over obs-websocket
- [ ] A downloaded, quarantined plugin zip is untested; the READMEs tell users to clear the quarantine mark
- [ ] Both plugins are tested on one machine only: M1 Max, OBS 33.0.3, OpenGL renderer
- [ ] Trails: a surface right at the cutoff flickers and leaves stray colored copies; smoothing the mask over time would reduce it
- [ ] Trails: the look lives in two shaders, `plugin/rgb_trails.effect` and `demo/apple/Sources/demo/Render.swift`, which must be changed together
- [ ] Check the intended use against `LICENSE_MODEL` (research-only, non-commercial)

## Notes / observations

- In a clone with a remote named `upstream`, `gh` acts on that remote by default. A watch on the new repo's first build looked at Apple's repository until `gh repo set-default` was set.
- The trails repo was cloned from the first plugin and inherited its local `v0.1.0` tag. `git tag` then refused, and the chained push never ran, so the first release attempt did nothing. Deleting the inherited local tag fixed it.
- OBS pictures are premultiplied by their alpha: a filter that fades a picture scales all four channels.
- The stray colored bands seen in a synthetic sliding clip were first put down to the ceiling sitting at the cutoff. A nearer cutoff reduced them but did not remove them there. On the live camera they appeared at 1.0 m only, and not at 0.8 or 1.5 m, so the explanation holds for the real scene; the synthetic clip also shifts the model's sense of scale as it slides.
- A stale model path in a saved scene is handled: the plugin logs that the model does not exist and passes the picture through.
- OBS was driven for every check through its WebSocket server, in two scene collections added for the purpose, "Depth Background Removal" and "Depth RGB Trails". The user's collection was restored each time.
