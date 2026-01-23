# Camera Calibration (Required)

This project estimates ArUco/AprilTag marker pose from a top-down USB camera.  
To get accurate pose estimates, you must calibrate the camera intrinsics and ensure the marker size in code matches the printed markers.

Calibration is performed using:
- `src/cameraCalibration.py` (chessboard calibration)
- output file: `calibration.npz`
- used by: `src/aruco_pose.py`

---

## What calibration produces

`calibration.npz` stores:
- `camMatrix` — camera intrinsic matrix (fx, fy, cx, cy)
- `distCoef` — lens distortion coefficients

These are required for accurate marker pose estimation and stability.

---

## Prerequisites

- A printed chessboard calibration target
- Good lighting and a stable camera mount
- Python + OpenCV installed (the same OpenCV used by your project)
- A set of calibration images (or live capture, depending on how your script is written)

---

## Step 1 — Print a chessboard target

Print a chessboard pattern on A4/A3.
- Print at **100% scale** (no “fit to page”).
- Mount it flat (cardboard helps) to avoid warping.

> If your `cameraCalibration.py` expects a specific chessboard size (e.g., 7×9 corners), keep the print consistent with that.

---

## Step 2 — Capture calibration images

You need multiple images of the chessboard from different positions and angles:
- Move the chessboard across the entire camera view (corners + edges + center)
- Vary the tilt slightly (to help solve intrinsics)
- Keep the chessboard sharp and well-lit
- Avoid motion blur

Recommended: **20–40 images** minimum.

---

## Step 3 — Run calibration

From the repo root:

```bash
python3 src/cameraCalibration.py
```
This should generate:
`calibration.npz`

### Where to place calibration.npz
`aruco_pose.py` loads calibration using a relative path:

```python
np.load("calibration.npz")
```
So you have two safe options:

**Option A (simplest):** put `calibration.npz` in the repo root
✅ Works if you run scripts from the repo root.

**Option B (also fine):** put `calibration.npz` next to the script
Place it in `src/` and run `aruco_pose.py` from inside `src/`.

If you want this to be robust regardless of working directory, update `aruco_pose.py` to load calibration using an absolute path (recommended improvement).

---

## Step 4 — Verify calibration quality
After calibration, check:

### A) Visual sanity checks
- Marker detections should be stable (no sudden jumps when marker is static)
- Pose should not “flip” randomly
- Estimated distance should be reasonable (not off by 2×)

### B) Quick quantitative checks (optional)
Many calibration scripts print:
- RMS reprojection error
As a rough guide:
- **< 1.0 px** is usually decent for a typical webcam setup
- **< 0.5 px** is very good
- If it’s large (> 2–3 px), retake images with better lighting/sharpness

---

## Step 5 — Marker size consistency (very important)
Your marker size must match the printed tags.
- Printed marker document: `docs/aruco_markers_25mm.docx`
- If you use a different size (e.g., 50mm), update the marker size parameter in the vision code.
If marker size is wrong, you will see:
- incorrect depth / scale (distance estimates wrong)
- pick-and-place accuracy degraded

---

## Common issues & fixes
### “Markers detected but pose is noisy / jittery”
- Increase lighting
- Reduce motion blur (faster shutter / brighter scene)
- Ensure camera is rigidly mounted (no vibration)
- Improve calibration image quality (more images, better coverage)

### “Pose scale seems wrong”
- Marker size in code does not match printed marker size
- Chessboard print is not at 100% scale
- Wrong chessboard square size used in calibration (if your script uses square size)

### “aruco_pose.py can’t find calibration.npz”
- You ran the script from a directory where `calibration.npz` is not present
- Fix by placing `calibration.npz` in repo root or editing the load path in `aruco_pose.py`

### “OpenCV / cv_bridge import errors”
- Prefer using OpenCV consistent with your ROS install
- Avoid mixing pip OpenCV with system cv_bridge unless you know they match

---

## Recommended improvement (optional)
To make calibration loading robust, edit `aruco_pose.py` to load using an absolute path based on the script location, e.g.:
```
python
from pathlib import Path
import numpy as np

calib_path = Path(__file__).resolve().parent / "calibration.npz"
with np.load(str(calib_path)) as f:
    camMatrix = f["camMatrix"]
    distCoef = f["distCoef"]
```
This prevents “works on my machine” path issues.

---

## Next step
Once calibration is in place, run the vision node:
```bash
python3 src/aruco_pose.py
```
and confirm stable marker detection before running the full pick-and-place pipeline.
