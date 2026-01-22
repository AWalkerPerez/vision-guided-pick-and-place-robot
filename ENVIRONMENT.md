# Environment

This project is a hardware-integrated ROS2 system (camera + AprilTags + Dynamixel motors via OpenRB-150).  
It is not simulation-first; you will need the physical robot setup (or equivalent hardware) to run the full stack.

## Tested platform
- Host OS: Ubuntu (recommended: 22.04 or newer)
- ROS 2: Iron (recommended)
- Python: 3.x (ROS2 system Python)

## Core dependencies
### ROS 2 packages (typical)
- `rclpy`
- `sensor_msgs`, `geometry_msgs`, `std_msgs`
- `tf2_ros`
- `cv_bridge` (if using ROS image transport + OpenCV)
- `image_transport` (optional)

### Python packages
- `numpy`
- `opencv-python` (or system OpenCV depending on ROS install)
- AprilTag detection (one of the following depending on your implementation):
  - `apriltag` (Python wrapper), or
  - `pupil_apriltags`, or
  - OpenCV AprilTag module (if available in your build)

### Motor / hardware interface
- Dynamixel control stack as used in this project (OpenRB-150 + Dynamixel SDK / ROS wrapper depending on your implementation)

## Notes
- If you are using ROS-installed OpenCV/cv_bridge, prefer system packages over pip OpenCV to avoid ABI conflicts.
- For large binaries (videos/CAD), consider Git LFS.

