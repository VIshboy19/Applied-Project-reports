# RoArm-M2-S Robotic Arm Control Using ESP32 and Python

## 1. Objective
This guide outlines the setup and methodology for controlling the **RoArm-M2-S robotic arm** using an **ESP32 microcontroller** and **JSON-based command protocols** via Python. The objective is to enable robust, real-time communication between a **Windows PC** and the robotic arm through **UART serial communication**.


## 2. System Overview
The RoArm-M2-S operates via **Python scripts** that utilize structured **JSON commands**. Although the ESP32 can be programmed through the Arduino IDE, compatibility issues with JSON data formatting make Python the preferred method. 

JSON (JavaScript Object Notation) is a lightweight and human-readable data-interchange format that allows for easy control and feedback.



## 3. Firmware Installation Using ESP32 Flash Download Tool

### 3.1 Tool Setup

1. Download the ESP32 Flash Download Tool (v3.9.5)
2. Extract and run `flash_download_tool_3.9.5.exe`
3. Two windows will appear: a GUI and a terminal log window

### 3.2 Configure Flashing Settings

- Set **Chip Type** to `ESP32`
- Set **WorkMode** to `Factory`
- Use relative paths for firmware binaries
- Click **OK** to proceed

### 3.3 Firmware Flashing Procedure

1. Connect the RoArm-M2-S to PC via USB
2. Launch the ESP32 Flash Tool
3. Upload the demo firmware (.bin file)
4. Select correct COM port (e.g., `COM3`)
5. Set baud rate: `921600`
6. Click **START**
7. Wait until status changes from `IDLE` to `FINISH`

### 3.4 Powering the Robotic Arm

- Disconnect USB after flashing
- Connect a **12V 5A power supply**
- On startup, the arm is ready to accept **Python-based JSON commands over UART**



## 4. UART Communication Setup Using Python

### 4.1 Objective
Establish UART serial communication between the ESP32 and PC using Python for sending JSON command packets.

### 4.2 Python Installation

1. Download **Python 3.12.0** from the [official site](https://www.python.org)
2. Run installer (`python-3.12.0-amd64.exe`)
3. Check **"Add Python to PATH"**
4. Select **"Customize Installation"** and proceed
5. Complete installation

### 4.3 Virtual Environment Setup

```bash
# Navigate to your desired directory
cd C:\Users\<YourName>\RoArm-M2-S_Python

# Create a virtual environment
python -m venv venv

# Activate it (Windows)
venv\Scripts\activate

# Install dependencies (e.g., pyserial)
pip install -r requirements.txt
```



## 5. JSON Command Interface for RoArm-M2-S

### 5.1 System Initialization and Reset

```json
{"T":100}
```

- `CMD_MOVE_INIT`: Move all joints to their initial position.
- This command **blocks** further execution until motion is complete.



### 5.2 Joint Angle Control (Radians)

```json
{"T":101,"joint":1,"rad":1.57,"spd":0,"acc":10}
```

- `CMD_SINGLE_JOINT_CTRL`: Rotate a single joint.
- `joint`: 1=Base, 2=Shoulder, 3=Elbow, 4=EOAT
- `rad`: Rotation angle in radians
- `spd`: Speed (0 = max), unit: steps/s
- `acc`: Acceleration (0–254), unit: 100 steps/s²

```json
{"T":102,"base":0,"shoulder":0,"elbow":1.57,"hand":3.14,"spd":0,"acc":10}
```

- `CMD_JOINTS_RAD_CTRL`: Control all joints together

```json
{"T":106,"cmd":3.14,"spd":0,"acc":0}
```

- `CMD_EOAT_HAND_CTRL`: Control wrist/clamp only



### 5.3 Joint Angle Control (Degrees)

```json
{"T":121,"joint":2,"angle":45,"spd":10,"acc":10}
```

- `CMD_SINGLE_JOINT_ANGLE`: Control individual joint in degrees

```json
{"T":122,"b":0,"s":0,"e":90,"h":180,"spd":10,"acc":10}
```

- `CMD_JOINTS_ANGLE_CTRL`: Control all joints in degrees



### 5.4 Cartesian Control (Inverse Kinematics)

```json
{"T":103,"axis":3,"pos":50,"spd":0.25}
```

- `CMD_SINGLE_AXIS_CRTL`: Move EOAT along a single axis

```json
{"T":104,"x":235,"y":0,"z":234,"t":3.14,"spd":0.25}
```

- `CMD_XYZT_GOAL_CTRL`: Move EOAT to X/Y/Z and rotation `t` (blocking)

```json
{"T":1041,"x":235,"y":0,"z":234,"t":3.14}
```

- `CMD_XYZT_DIRECT_CTRL`: Direct movement (non-blocking)



### 5.5 Feedback and Monitoring

```json
{"T":105}
```

- `CMD_SERVO_RAD_FEEDBACK`: Returns:
  - Coordinates: `x, y, z`
  - Joint angles: `b, s, e, t`
  - Torques: `torB, torS, torE, torH`



### 5.6 Continuous Control

```json
{"T":123,"m":0,"axis":1,"cmd":1,"spd":10}
```

- `CMD_CONSTANT_CTRL`: For real-time continuous movement
  - `m`: Mode (0 = angle, 1 = coordinate)
  - `axis`: Axis or joint
  - `cmd`: 1 = increase, 2 = decrease, 0 = stop
  - `spd`: Speed coefficient (0–20 recommended)





##  Dependencies

- Python 3.12+
- `pyserial`
- USB Drivers (e.g., CH340/CP210x depending on board)



##  License & Credits

This implementation is based on the official RoArm-M2-S Python demo and community adaptations. All JSON command formats are derived from their public SDK documentation.
