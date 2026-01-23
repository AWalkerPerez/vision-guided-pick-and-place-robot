# src

Core ROS 2 (Python) scripts for the **vision-guided 3DOF pick-and-place robot**.

This project is **hardware-first** (camera + Dynamixel/OpenRB-150 + physical robot).  
There is no simulation workflow provided.

---

## Files

### ROS 2 nodes
- `camera_publisher.py`  
  Publishes camera frames.
  - Publishes: `camera/image` (`sensor_msgs/Image`)

- `aruco_pose.py`  
  Detects ArUco markers in `camera/image`, computes each marker pose **relative to marker ID 0**, and publishes a topic per marker ID.
  - Subscribes: `camera/image`
  - Publishes: `desired_pos_<id>` (`std_msgs/Float32MultiArray`) e.g. `desired_pos_7`
  - Parameter: `marker_size` (default **0.025 m** = 25 mm)

- `marker_tracking.py`  
  High-level **pick-and-place state machine**.
  - Subscribes: `joint_state` and dynamically subscribes to `desired_pos_<id>` topics
  - Publishes: `joint_pos`, `cartesian_pos`, `gripper_command`
  - Two stages:
    - **Stage 1 (recording):** records desired goal positions from **even marker IDs**
    - **Stage 2 (moving):** tracks objects from **odd marker IDs**
  - Pairing rule used in code:
    - Object marker `X` (odd) uses goal marker `(X + 1)` (even)

- `control_dls.py`  
  Damped Least Squares (DLS) IK controller: Cartesian target → joint targets.
  - Subscribes: `joint_state`
  - Publishes: `joint_pos`, `cartesian_pos`

- `hardware_interface.py`  
  Hardware bridge for Dynamixels via OpenRB-150.
  - Publishes: `/joint_state`, `/cartesian_pos`, `/ee_min_max`, `/emergency_stop`
  - Subscribes: `/joint_pos`, `/joint_vel`, `/joint_cur`, `/joint_pos_rel`, `/gripper_command`

### Utility (not a ROS node)
- `cameraCalibration.py`  
  Chessboard calibration utility. Saves intrinsics to `calibration.npz`.

---

## Marker pose message format (important)

`aruco_pose.py` publishes a `Float32MultiArray` with 6 values:

`[x_mm, y_mm, z_mm, roll_deg, pitch_deg, yaw_deg]`

In the current implementation, the first two values are published as:

- `msg.data[0] = relative_y_mm`
- `msg.data[1] = relative_x_mm`

`marker_tracking.py` then uses only the first two values:

- `pos = [msg.data[0], msg.data[1]]`
- `yaw = msg.data[5]`

So the system is internally consistent, but note the **X/Y ordering** is “swapped” compared to some comments.

---

## Calibration (required)

`aruco_pose.py` loads calibration like this:

```python
np.load("calibration.npz")
```
So the simplest workflow is to generate the file in the repo root and run nodes from the repo root.

Generate calibration:
```bash
python3 src/cameraCalibration.py
```
This creates:
`calibration.npz` (in your current working directory)

---

## Running the system (typical order)
Open separate terminals and ensure ROS 2 is sourced in each.

### 1) Camera publisher
```bash
python3 src/camera_publisher.py
```
### 2) Marker detection
```bash
python3 src/aruco_pose.py
```
### 3) Hardware interface (robot must be connected)
```bash
python3 src/hardware_interface.py
```
### 4) Autonomy / control
You typically run the pick-and-place state machine (and optionally DLS control depending on your test):
```bash
python3 src/marker_tracking.py
```
Optional (if you use the DLS node separately):
```bash
python3 src/control_dls.py
```

---

## Debugging quick commands
List topics:
```bash
ros2 topic list
```
Echo key topics:
```bash
ros2 topic echo /joint_state
ros2 topic echo desired_pos_0
ros2 topic echo desired_pos_1
```

---

## Common issues
**- No markers detected**

    - Check lighting / focus
    
    - Confirm printed marker size matches marker_size (25mm default)
    
    - Confirm calibration.npz exists where you run aruco_pose.py
    
    - Robot moves wrong direction / mirrored
    
    - Double-check the X/Y ordering (see message format section)
    
    - Verify camera mounting orientation and coordinate assumptions

**- Motors not responding**

    - Check device connection/permissions/port configuration inside hardware_interface.py
    
    - Confirm motor IDs and baud rate match your hardware setup
