# Motion Tracker Automated

**Automate video-to-camera tracking and sparse 3D reconstruction with FFmpeg and COLMAP.**

Motion Tracker Automated is a Windows batch workflow that processes videos into extracted frames, estimated camera poses, and sparse 3D scene data. It connects FFmpeg frame extraction with COLMAP feature extraction, sequential matching, reconstruction, and text export.

[**Download Motion Tracker v1**](https://github.com/Uplinkdevv/Motion-Tracker-automated/releases/download/Release/automated.tracker.rar) · [Release notes](https://github.com/Uplinkdevv/Motion-Tracker-automated/releases/tag/Release)

## Download and quick start

The full project is distributed through **GitHub Releases**. Download the attached `automated.tracker.rar` archive (approximately **145 MB**) from [Motion Tracker v1](https://github.com/Uplinkdevv/Motion-Tracker-automated/releases/tag/Release).

1. Download `automated.tracker.rar` using the link above.
2. Extract the entire archive to your Desktop using an archive utility that supports RAR files.
3. Open the extracted project folder and keep its numbered folders together.
4. Put your video files in `02 VIDEOS`.
5. Open `05 SCRIPT` and double-click `batch_reconstruct.bat`.
6. Find each video's results in `04 SCENES`.

The release instructions describe setup as downloading and extracting the project. Use the attached **RAR asset**; GitHub's automatic **Source code (zip)** and **Source code (tar.gz)** downloads contain the repository snapshot, not the uploaded project package.

## Features

- **Batch processing:** processes files from a shared input folder, with a separate output folder for each video.
- **Automatic frame extraction:** saves video frames as numbered JPEG images.
- **Camera and scene reconstruction:** runs COLMAP to estimate camera poses and sparse 3D points.
- **Text export:** converts sparse model `0` into COLMAP's TXT format.
- **Completion checks:** skips completed scenes when the completion marker and required model files exist.
- **Frame reuse:** reuses extracted JPEGs when a frame-completion marker is present.
- **Failure reporting:** checks processing commands, reports failed videos, and continues with remaining jobs.

## How it works

| Stage | Tool | Result |
| --- | --- | --- |
| 1. Extract frames | FFmpeg | Numbered JPEG images from the input video |
| 2. Extract features | COLMAP `feature_extractor` | Image features stored in a scene database |
| 3. Match frames | COLMAP `sequential_matcher` | Feature correspondences across nearby frames |
| 4. Reconstruct | COLMAP `mapper` | Sparse models containing camera poses and 3D points |
| 5. Export | COLMAP `model_converter` | Text representation of sparse model `0` |

The script resolves paths relative to its own location, so the workflow uses the folder structure below rather than a fixed installation path.

## Requirements

To use the release, you need:

- **Windows**, with Command Prompt support for `.bat` files.
- An archive utility that can extract `.rar` files.
- GPU support compatible with the packaged COLMAP build for feature extraction.
- Enough free storage for extracted frames, the database, and reconstruction outputs.

The workflow uses **FFmpeg** and **COLMAP** from the extracted project folders. If either tool is missing after extraction, check that the full release archive was extracted. For a manual setup, obtain [FFmpeg](https://ffmpeg.org/download.html) and [COLMAP](https://github.com/colmap/colmap/releases), preserving the folder layout below and using a COLMAP build compatible with the listed options.

The script enables GPU feature extraction and CPU feature matching. It does not provide an automatic CPU fallback for feature extraction.

### Folder structure

Keep these folders together:

```text
Motion-Tracker-automated/
├── 01 COLMAP/
│   ├── colmap.exe          # Alternatively: bin/colmap.exe
│   ├── plugins/
│   │   └── platforms/
│   │       └── qwindows.dll
│   └── ...                # Other COLMAP distribution files
├── 02 VIDEOS/
│   └── your-video.mp4
├── 03 FFMPEG/
│   └── ffmpeg.exe          # Alternatively: bin/ffmpeg.exe
├── 04 SCENES/
└── 05 SCRIPT/
    └── batch_reconstruct.bat
```

Keep the COLMAP distribution's supporting libraries and plugin folders intact. The script checks for `plugins/platforms/qwindows.dll` before processing.

## Usage

After extracting the release archive:

1. Open the extracted project folder and confirm the numbered folders remain together.
2. Put your input videos in `02 VIDEOS`.
3. Double-click `05 SCRIPT/batch_reconstruct.bat`, or run it from Command Prompt:
   ```bat
   "05 SCRIPT\batch_reconstruct.bat"
   ```
4. Follow the per-video progress and error messages in the console.
5. Open `04 SCENES/<video-name>/` to inspect the generated files.

Use distinct video filenames **before the extension**: `clip.mp4` and `clip.mov` both map to the same `clip` output folder. Keep unrelated files out of the input folder, because the script does not filter files by video extension.

## Output files

A successful scene typically contains:

```text
04 SCENES/<video-name>/
├── images/
│   ├── frame_000001.jpg
│   └── ...
├── database.db
├── sparse/
│   ├── 0/
│   │   ├── cameras.bin
│   │   ├── images.bin
│   │   └── points3D.bin
│   ├── cameras.txt
│   ├── images.txt
│   └── points3D.txt
├── frames.complete
└── reconstruction.complete
```

- **Camera data** describes reconstructed camera parameters and registered image poses.
- **Point data** describes the sparse 3D scene.
- **Completion markers** help the script decide what to reuse or skip on later runs.

COLMAP may create multiple numbered models. The script exports model `0`; it does not compare models or automatically select the largest reconstruction.

## Default processing settings

| Setting | Current value |
| --- | --- |
| Frame sampling | Every frame; no explicit downsampling |
| JPEG quality | FFmpeg `-qscale:v 2` |
| Camera handling | `ImageReader.single_camera 1` |
| Feature extraction | GPU enabled |
| Maximum extraction image size | `4096` |
| Feature matching | CPU |
| Sequential matching overlap | `15` |
| Mapper threads | Windows `NUMBER_OF_PROCESSORS` value |
| Export format | COLMAP TXT |

These settings are embedded in the batch file; the workflow has no settings interface.

## Reruns and recovery

A scene is skipped only when `reconstruction.complete`, `sparse/0/images.bin`, and `sparse/images.txt` all exist.

If a scene is incomplete, processing is attempted again. Frame extraction is skipped when `frames.complete` and JPEG frames exist; later COLMAP stages are invoked again using the scene's existing paths. This is partial reuse, not a full checkpoint system.

The script does not detect changed input videos. To process revised footage independently, use a new, unique base filename.

## Troubleshooting

| Problem | What to check |
| --- | --- |
| FFmpeg or COLMAP cannot be found | Place the executable in its tool folder or that folder's `bin` subfolder. |
| Missing Windows Qt plugin | Restore the full COLMAP distribution, including `plugins/platforms/qwindows.dll`. |
| Feature extraction fails | Check GPU availability and whether your COLMAP build supports the configured options. |
| No sparse model is produced | Inspect COLMAP's console output and try footage with clearer detail and overlapping views. |
| Long processing time or high disk use | Every frame is extracted; begin with a short clip. |
| A scene is unexpectedly skipped | Check whether completion markers and existing model files belong to an earlier input with the same base filename. |

## Scope and limitations

This workflow reconstructs camera motion and sparse scene geometry from video. It does not implement real-time object tracking, dense reconstruction, mesh generation, or a rendered scene viewer.

Results depend on the input footage and successful COLMAP reconstruction. The workflow details are based on inspection of the batch script; download instructions follow the published Motion Tracker v1 release. The release archive has not been independently runtime-tested for this documentation update.

## Credits

The batch script credits **[Polyfjord](https://youtube.com/polyfjord)** as its original author.

The workflow uses **[COLMAP](https://github.com/colmap/colmap)** for reconstruction and **[FFmpeg](https://ffmpeg.org/)** for frame extraction. These external tools retain their own licenses.

## License

This repository uses **CC0 1.0 Universal**. See [LICENSE](LICENSE) for the full terms.
