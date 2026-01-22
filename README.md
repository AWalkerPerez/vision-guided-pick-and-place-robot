# Autonomous 3DOF Pick-and-Place Robot (Vision-Based Control)

ROS2-based autonomous pick-and-place with AprilTag detection and Jacobian-based inverse kinematics, running on a physical 3DOF arm (Dynamixel + OpenRB-150) with a top-down camera.

## Demo
- See `/media` (videos/photos) for example runs:
  - obstacle avoidance
  - multi-object pick-and-place
  - accuracy evaluation

## What’s in this repo
- ROS2 nodes for camera streaming, AprilTag detection, and motion control
- Experimental plots + diagrams used in the final report
- CAD exports for the robot + end-effector
- Media (photos/videos) documenting the setup and results
- Final report PDF

## Repo structure


## Quickstart (hardware required)
**1.** Install ROS2 (Iron recommended)

**2.** Install dependencies (see `environment.md`)

**3.** Follow hardware setup and calibration (see `setup.md`)

**4.** Run the ROS2 nodes (see your `src/` package README or launch instructions)

## Report
`/report/AUTONOMOUS_3DOF_PICK-AND-PLACE_ROBOT_USING_VISION-BASED_CONTROL.pdf`

## Notes
- This project is hardware-dependent and not simulation-first.
