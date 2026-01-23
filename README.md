# Vision-Guided Pick-and-Place Robot (3-DOF)

Autonomous 3-DOF pick-and-place robot using **vision-based control** (ArUco markers) to localise objects and drive the robot end-effector using **inverse kinematics** (DLS / Jacobian-based control).  
This repository contains the full project deliverables: source code, CAD, calibration notes, diagrams, media, and the final report.

> ⚠️ This project is **hardware-dependent** (camera + robot). It is not designed to be simulation-only out of the box.

---

## Project overview

**Pipeline (high level):**
1. Camera captures frames of the workspace.
2. ArUco markers are detected → object pose is estimated in the camera frame.
3. Pose is transformed into the robot/world frame using calibration.
4. A controller computes joint updates (e.g., Damped Least Squares IK) to move to target.
5. Gripper/clamp action closes/opens at the right time.
6. Optional behaviours: obstacle avoidance, multi-object sequencing, homing.

---

## Repository layout

```text
vision-guided-pick-and-place-robot/
├── README.md
├── .gitignore
├── environment.md
├── setup.md
├── calibration.md
│
├── report/
│   └── AUTONOMOUS_3DOF_PICK-AND-PLACE_ROBOT_USING_VISION-BASED_CONTROL.pdf
│
├── src/
│   ├── README.md
│   ├── (your Python/ROS scripts live here)
│   └── (configs / launch / helpers depending on your implementation)
│
├── cad/
│   ├── README.md
│   └── (Fusion 360 source + exports: .f3d/.f3z, .step, .stl, .3mf)
│
├── diagrams/
│   ├── README.md
│   └── (system diagrams, flowcharts, architecture images)
│
├── media/
│   ├── README.md
│   ├── photos/
│   └── videos/
│
└── docs/
    ├── README.md
    └── (optional: extra notes, troubleshooting, design decisions)
```

---

## Quick start
### 1) Read the setup docs
- Hardware + software setup: see `setup.md`
- Environment / dependencies: see `environment.md`
- Calibration process: see `calibration.md`

### 2) Run the project (hardware required)
The entrypoint depends on how your `src/` scripts are structured (ROS nodes vs. standalone scripts).
Go to `src/README.md` for the exact run commands.

---

## Calibration
Calibration is required to map camera measurements into the robot coordinate frame and to get stable pose estimates.
See:
- `calibration.md`
- Marker PDF / reference: `aruco_markers_25mm.docx`

---

## Results & media
- Photos and videos are under `media/`.
- Diagrams and pipeline visuals are under `diagrams/`.
- Full report (methods, experiments, results): `report/`.

---

## Notes on reproducibility
Because this is a physical system, results depend on:
- camera placement + intrinsics,
- lighting conditions,
- robot assembly tolerances,
- motor tuning / controller gains,
- marker size + print quality.
The report documents the tested configuration and evaluation.

---

## Citation
If you use or reference this work, please cite the project report in `report/`.
