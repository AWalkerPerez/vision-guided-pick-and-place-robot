# src

Core ROS 2 (Python) scripts for the **vision-guided 3DOF pick-and-place robot**.

This project is **hardware-first**:
- USB camera (top-down)
- Physical 3DOF arm with Dynamixels (via OpenRB-150)
- Printed ArUco/AprilTag markers
- End-effector / gripper hardware

There is no simulation workflow provided.

---

## Files in this folder

### ROS 2 nodes
- `camera_publisher.py`  
  Publishes camera frames as ROS 2 images.
  - **Publishes:** `camera/image` (`sensor_msgs/Image`)

- `aruco_pose.py`  
  Subscribes to camera images, detects markers, and publishes marker position per ID.
  - **Subscribes:** `camera/image`
  - **Publishes:** `desired_pos_<id>` (`std_msgs/Float32MultiArray`)  
    Example: `desired_pos_7` for marker id 7.

  > **Important:** this script loads camera intrinsics from `calibration.npz` using a relative path:
  > `np.load('calibration.npz')`  
  > So you must run it from a directory where `calibration.npz` exists (or edit the path in the script).

- `control_dls.py`  
  Damped Least Squares (DLS) IK controller: converts target Cartesian position into joint commands.
  - **Subscribes:** `joint_state`
  - **Publishes:** `joint_pos`, `cartesian_pos`

- `marker_tracking.py`  
  High-level pick-and-place logic/state machine. Sends joint targets and gripper commands.
  - **Subscribes:** `joint_state`
  - **Publishes:** `joint_pos`, `cartesian_pos`, `gripper_command`

- `hardware_interface.py`  
  Hardware node for the Dynamixel/OpenRB-150 system. Publishes joint feedback and listens for command topics.
  - **Publishes:** `/joint_state`, `/cartesian_pos`, `/ee_min_max`, `/emergency_stop`
  - **Subscribes:** `/joint_pos`, `/joint_vel`, `/joint_cur`, `/joint_pos_rel`, `/gripper_command`

  > This node is the “bridge” to the physical robot. Without hardware connected, this script will not function end-to-end.

### Utility (not a ROS node)
- `cameraCalibration.py`  
  Offline chessboard camera calibration utility. Produces `calibration.npz`.

---

## Prerequisites

- ROS 2 installed and sourced (these scripts use `rclpy`)
- OpenCV + `cv_bridge`
- Python packages: `numpy`, `opencv-python` (or system OpenCV)
- Dynamixel/OpenRB dependencies as required by `hardware_interface.py`

If you hit OpenCV/cv_bridge import issues, prefer **system packages** that match your ROS install
(avoid mixing pip OpenCV with apt-installed cv_bridge unless you know the ABI matches).

---

## Calibration (required before marker detection)

Run the calibration utility and generate `calibration.npz`:

```bash
python3 src/cameraCalibration.py
```

Then ensure `calibration.npz` is available from the working directory when running `aruco_pose.py`,
or update the path inside `aruco_pose.py`.

---

## How to run (typical order)
In separate terminals, always source ROS 2 first.

### Terminal 1 — Camera publisher
```bash
python3 src/camera_publisher.py
```
### Terminal 2 — Marker detection
```bash
python3 src/aruco_pose.py
```
### Terminal 3 — Hardware interface (robot must be connected)
```bash
python3 src/hardware_interface.py
```
### Terminal 4 — Control / pick-and-place logic
Depending on how you ran experiments, you may run one or both:
```bash
python3 src/control_dls.py
python3 src/marker_tracking.py
```

----

## Notes on topics and naming
- Some scripts publish topics without a leading `/` (e.g. `joint_pos`)
- Some scripts subscribe/publish with a leading `/` (e.g. `/joint_pos`)

ROS 2 treats these as different names in some contexts depending on node namespaces.
If you see “no messages received”, check topic names with:
```bash
ros2 topic list
ros2 topic echo /joint_state
```
If needed, standardise topic names (either always with `/` or always without) across scripts.

---

## Common issues
**- No marker detections**
  - Confirm marker size in code matches your printed markers (25mm / 50mm)
  - Confirm `calibration.npz` is being loaded (correct working directory)
  - Improve lighting / focus
**- Robot moves unpredictably**
  - Increase damping in DLS (in `control_dls.py`)
  - Verify coordinate frame assumptions (camera frame vs robot base frame)
  - Verify link lengths / kinematic model constants
**- Motors not responding**
  - Check serial port permissions and correct device name
  - Confirm motor IDs and baud rate in `hardware_interface.py`
  - Confirm torque enable logic is active
