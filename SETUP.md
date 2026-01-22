# Setup

This repo contains a ROS2-based pick-and-place system that uses:
- A USB camera (Logitech)
- AprilTags for object detection / pose estimation
- A 3DOF arm actuated by Dynamixel motors via OpenRB-150
- Motion control using Jacobian-based IK (e.g., DLS / Jacobian Transpose depending on mode)

> This project requires hardware to run end-to-end.

---

## Hardware prerequisites
- Raspberry Pi 4 (or equivalent controller running ROS2 nodes / motor interface)
- OpenRB-150 interface board
- Dynamixel motors (as per your build)
- Motorised clamp end-effector
- USB camera (Logitech) mounted above workspace
- Printed AprilTags (see `/aruco_markers_25mm.docx` or your marker assets)
- A flat workspace with known scale + tag placement

---

## Software prerequisites
1. Install ROS2 (recommended: Iron) on the main machine(s)
2. Clone this repo
3. Install Python dependencies used by the vision/control nodes (see `environment.md`)

---

## Typical run flow (high level)
1. **Camera node**
   - Starts capturing frames and publishing images as ROS topics.

2. **Vision node**
   - Detects AprilTags, publishes tag poses (and/or TF frames).

3. **Control node**
   - Converts tag poses into target end-effector poses.
   - Runs IK (DLS / Jacobian methods) to generate joint commands.

4. **Motor interface**
   - Sends position/velocity commands to Dynamixels.
   - Reads motor feedback (current/velocity) for safety + collision handling.

---

## Calibration (recommended)
- Camera intrinsics calibration (checkerboard)
- Verify tag size matches your printed markers (e.g., 25mm / 50mm)
- Verify coordinate frames:
  - camera frame
  - world/workspace frame
  - base frame
  - end-effector frame

See: `docs/calibration.md` (create this if you don’t already have it)

---

## Safety notes
- Always test with reduced speed first.
- Ensure current-limits / stall detection is enabled before autonomous sequences.
- Keep a physical emergency stop procedure (unplug power / disable torque).

