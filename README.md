# Volleyball Video Analysis Notebooks + 3D Mesh Viewer

Three notebook-driven pipelines for volleyball footage:

- [`VideoEnhancementPipeline.ipynb`](VideoEnhancementPipeline.ipynb) restores raw match video with SeedVR2 and produces a cleaner `tracking_master_cv.mp4`.
- [`YOLO26_PersonTracker.ipynb`](YOLO26_PersonTracker.ipynb) tracks players frame by frame and exports both `tracked.mp4` and `tracked_tracks.json`.
- [`sam3dbody_video.ipynb`](sam3dbody_video.ipynb) packages a SAM 3D body mesh viewer as `sam3d_viewer.html` plus a `viewer_data/` bundle.

The interactive mesh viewer is deployed from [`volleyball-ui/`](volleyball-ui), not embedded directly in this README, because GitHub strips `<iframe>` embeds from rendered Markdown.

## Live Mesh Viewer

- Intended production URL: [volleyball-3d-mesh.vercel.app](https://volleyball-3d-mesh.vercel.app)
- One-click import to Vercel Pro: [Deploy `volleyball-ui` to Vercel](https://vercel.com/new/clone?repository-url=https://github.com/mazooni/Volleyball-3D-Mesh&project-name=volleyball-3d-mesh&root-directory=volleyball-ui)
- Vercel root directory: `volleyball-ui`

## Repository Layout

```text
.
├── README.md
├── VideoEnhancementPipeline.ipynb
├── YOLO26_PersonTracker.ipynb
├── sam3dbody_video.ipynb
├── assets/readme/
│   ├── video-enhancement-before.mp4
│   ├── video-enhancement-after.mp4
│   └── yolo-person-tracking.mp4
└── volleyball-ui/
    ├── index.html
    └── 3dmesh/
        ├── sam3d_viewer.html
        └── viewer_data/
```

## 1. `VideoEnhancementPipeline.ipynb`

SeedVR2-based video restoration for low-quality volleyball footage.

- Clones and prepares the SeedVR environment and supporting runtime.
- Builds a constant-frame-rate mezzanine, splits the source video into overlap-aware chunks, and chooses runtime topology from the GPUs it can see.
- Reassembles the restored chunks into a final `tracking_master_cv.mp4`.

### Before / After Demo

<table>
  <tr>
    <td width="50%" align="center">
      <strong>Original Input</strong><br />
      <video src="assets/readme/video-enhancement-before.mp4" controls muted playsinline loop width="100%"></video><br />
      <sub><a href="assets/readme/video-enhancement-before.mp4">Open clip directly</a></sub>
    </td>
    <td width="50%" align="center">
      <strong>Enhanced Output</strong><br />
      <video src="assets/readme/video-enhancement-after.mp4" controls muted playsinline loop width="100%"></video><br />
      <sub><a href="assets/readme/video-enhancement-after.mp4">Open clip directly</a></sub>
    </td>
  </tr>
</table>

## 2. `YOLO26_PersonTracker.ipynb`

Persistent person detection and ID tracking tuned for the near side of the court.

- Loads `video.mp4`, applies a `-0.55` degree rotation correction, and filters tracking to `KEEP_COURT_SIDE = "near"`.
- Uses `yolo26x.pt` for person detection and `yolo26x-cls.pt` for ReID embeddings through a custom `botsort_reid.yaml`.
- Renders `tracked.mp4` and exports `tracked_tracks.json` with frame-by-frame IDs and boxes.

### Tracking Demo

<p align="center">
  <video src="assets/readme/yolo-person-tracking.mp4" controls muted playsinline loop width="100%"></video>
</p>

<p align="center">
  <a href="assets/readme/yolo-person-tracking.mp4">Open the YOLO person-tracking demo directly</a>
</p>

## 3. `sam3dbody_video.ipynb`

SAM 3D body mesh extraction and viewer export for a volleyball clip.

- Loads `facebook/sam-3d-body-dinov3` and optional helper components such as the human detector and FOV estimator when available.
- Extracts frames from `video.mp4`, runs per-frame body estimation, and serializes the output into chunked mesh data.
- Writes an exported viewer package containing:
  - `sam3d_viewer.html`
  - `viewer_data/manifest.json`
  - `viewer_data/faces.bin`
  - `viewer_data/chunk_*.bin`

The deployable site in [`volleyball-ui/`](volleyball-ui) is the mesh-only version of that export. The old dashboard, clip browser, and other non-mesh website pieces were removed so the Vercel app is only the 3D viewer.

## Deploying the Viewer

1. Import this repo into Vercel Pro.
2. Set the root directory to `volleyball-ui`.
3. Deploy as a static site.

`volleyball-ui/index.html` is the public entrypoint, and the mesh payload is served from `volleyball-ui/3dmesh/viewer_data/*`.

## Notes

- The MP4 files in `assets/readme/` are optimized demo exports for GitHub display.
- The larger local source videos are intentionally not committed.
- The mesh viewer data is large, so this repo assumes Vercel Pro for a self-contained deployment of the full `viewer_data/` bundle.
