# ShakeItUp
An OpenArm robot (by Enactic) that shakes a container and guess its content.

Built during the Unaite Robotics Hackathon in the conference GOSIM Paris 2026.

The goal is to sense the jaw deformation (strain) and to use the vibration data to guess the content of the container. 
The experiments have been made with a Dragonfly IEPE piezo strain sensor (Wormsensing ref. DGF-UNI-W220405-10), and an acquisition system IOLITE-X (Dewesoft). 
The strain data is streamed with openDAQ library (https://docs.opendaq.com/manual/opendaq/3.30/tutorials/tutorial_application.html). 
Thus, you are free to update it to match your own setup.

The main code of the acquisition is in the submodule lespectrobot/src/lerobot/robots, and the code is currently working with robots SO-101 and OpenArm (for the Bi-OpenArm, just replace any of the two OpenArm by a OpenFollowerDragonTactile Arm in the file bi_openarm_follower.py).


## Installation

Clone the repository:

```bash
git clone --recursive https://github.com/jogarulfo/ShakeItUp.git
```

Install dependencies:

```bash
uv pip install -e .
cd lespectrobot
uv pip install -e .[damiao] 
```

You would use "feetech" instead of "damiao" if you use SO-100 or SO-101 instead of OpenArm.


## Step-by-step guide

The following is a step-by-step guide of the commands used during the hackaton to teleoperate, record and train OpenArm with a OpenArm_mini (https://github.com/pkooij/open-arms-mini) as the leader.

### Environment setup for OpenArm

```bash
conda create -n hackthespectrobot python=3.12 -y
conda activate hackthespectrobot
conda install -c conda-forge numpy matplotlib scipy polars pyarrow -y
pip install opencv-python opendaq
cd hacklespectrobot
pip install -e ".[damiao]"
pip install rerun-sdk
```

### Camera
```bash 
lerobot-find-cameras opencv 
```

### Setup CAN interfaces
```bash
lerobot-setup-can --mode=setup --interfaces=can0,can1
```
### Test motor communication
```bash
lerobot-setup-can --mode=test --interfaces=can0,can1
```
### Run speed/latency test
```bash
lerobot-setup-can --mode=speed --interfaces=can0
```

### Teleoperation and data collection
Follower Arm (Robot)
```bash
lerobot-calibrate \
    --robot.type=bi_openarm_follower \
    --robot.left_arm_config.port=can0 \
    --robot.left_arm_config.side=left \
    --robot.right_arm_config.port=can1 \
    --robot.right_arm_config.side=right \
    --robot.id=my_biopenarm
```

Leader Arm (Teleoperator) \
leader_right : /dev/ttyACM0
leader_left : /dev/ttyACM1
```bash
lerobot-calibrate --teleop.type=openarm_mini --teleop.port_left=/dev/ttyACM3 --teleop.port_right=/dev/ttyACM2   --teleop.id=my_leader
```

### Bimanual teleoperation

To teleoperate a bimanual OpenArm setup with two leader arms and two follower arms:
```bash
lerobot-teleoperate \
    --robot.type=bi_openarm_follower \
    --robot.left_arm_config.port=can0 \
    --robot.left_arm_config.side=left \
    --robot.right_arm_config.port=can1 \
    --robot.right_arm_config.side=right \
    --robot.id=my_biopenarm \
    --teleop.type=openarm_mini \
    --teleop.port_left=/dev/ttyACM0 \
    --teleop.port_right=/dev/ttyACM1 \
    --teleop.id=my_leader \
    --display_data=true
```

### With cameras : 

  ```bash
lerobot-teleoperate \
    --robot.type=bi_openarm_follower \
    --robot.left_arm_config.port=can0 \
    --robot.left_arm_config.side=left \
    --robot.right_arm_config.port=can1 \
    --robot.right_arm_config.side=right \
    --robot.cameras="{ top: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, wrist_right: {type: opencv, index_or_path: 4, width: 1280, height: 720, fps: 30},wrist_left: {type: opencv, index_or_path: 2, width: 1280, height: 720, fps: 30} }" \
    --robot.id=my_biopenarm \
    --teleop.type=openarm_mini \
    --teleop.port_left=/dev/ttyACM3 \
    --teleop.port_right=/dev/ttyACM2 \
    --teleop.id=my_leader \
    --display_data=true
```

### Recording Data

To record a dataset during teleoperation:
```bash
lerobot-record \
    --robot.type=bi_openarm_follower \
    --robot.left_arm_config.port=can0 \
    --robot.left_arm_config.side=left \
    --robot.right_arm_config.port=can1 \
    --robot.right_arm_config.side=right \
    --robot.cameras="{ top: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, wrist_right: {type: opencv, index_or_path: 4, width: 1280, height: 720, fps: 30},wrist_left: {type: opencv, index_or_path: 2, width: 1280, height: 720, fps: 30} }" \
    --robot.id=my_biopenarm \
    --teleop.type=openarm_mini \
    --teleop.port_left=/dev/ttyACM0 \
    --teleop.port_right=/dev/ttyACM1 \
    --teleop.id=my_leader \
    --display_data=true \
    --dataset.repo_id=jogarulfop/shakeitup4 \
    --dataset.num_episodes=10
```


### Inference : 

```bash
lerobot-rollout \
    --strategy.type=base \
    --policy.path=emmanuel-v/policy_2026-05-05_jr_openarm_shakeitup2_f \
    --robot.type=bi_openarm_follower \
    --robot.left_arm_config.port=can0 \
    --robot.left_arm_config.side=left \
    --robot.right_arm_config.port=can1 \
    --robot.right_arm_config.side=right \
    --robot.cameras="{ top: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, wrist_right: {type: opencv, index_or_path: 4, width: 1280, height: 720, fps: 30},wrist_left: {type: opencv, index_or_path: 2, width: 1280, height: 720, fps: 30} }" \
    --task="Put the coins in the box if they are inside the bottle" \
    --duration=90
```

### Typical outputs (ongoing cleaning)

Configuration Options
Follower Configuration
Parameter 	Default 	Description
port 	- 	CAN interface (e.g., can0)
side 	None 	Arm side: "left", "right", or None for custom limits
use_can_fd 	True 	Enable CAN FD for higher data rates
can_bitrate 	1000000 	Nominal bitrate (1 Mbps)
can_data_bitrate 	5000000 	CAN FD data bitrate (5 Mbps)
max_relative_target 	None 	Safety limit for relative target positions
position_kp 	Per-joint 	Position control proportional gains
position_kd 	Per-joint 	Position control derivative gains
Leader Configuration
Parameter 	Default 	Description
port 	- 	CAN interface (e.g., can1)
manual_control 	True 	Disable torque for manual movement
use_can_fd 	True 	Enable CAN FD for higher data rates
can_bitrate 	1000000 	Nominal bitrate (1 Mbps)
can_data_bitrate 	5000000 	CAN FD data bitrate (5 Mbps)
Motor Configuration

OpenArm uses Damiao motors with the following default configuration:
Joint 	Motor Type 	Send ID 	Recv ID
joint_1 (Shoulder pan) 	DM8009 	0x01 	0x11
joint_2 (Shoulder lift) 	DM8009 	0x02 	0x12
joint_3 (Shoulder rotation) 	DM4340 	0x03 	0x13
joint_4 (Elbow flex) 	DM4340 	0x04 	0x14
joint_5 (Wrist roll) 	DM4310 	0x05 	0x15
joint_6 (Wrist pitch) 	DM4310 	0x06 	0x16
joint_7 (Wrist rotation) 	DM4310 	0x07 	0x17
gripper 	DM4310 	0x08 	0x18
Troubleshooting
No Response from Motors

    Check power supply connections
    Verify CAN wiring (CAN-H, CAN-L, GND)
    Run diagnostics: lerobot-setup-can --mode=test --interfaces=can0
    See the Damiao troubleshooting guide for more details

CAN Interface Not Found

Ensure the CAN interface is configured:

ip link show can0

Resources

    OpenArm Website
    OpenArm Documentation
    OpenArm GitHub
    Safety Guide
    Damiao Motors and CAN Bus


debugpy --wait-for-client --listen 0.0.0.0:5678 /home/josephrigal/workspace/hacklespectrobot/lespectrobot/src/lerobot/scripts/lerobot_record.py --robot.type=bi_openarm_follower     --robot.left_arm_config.port=can0     --robot.left_arm_config.side=left     --robot.right_arm_config.port=can1     --robot.right_arm_config.side=right     --robot.cameras="{ top: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, wrist_right: {type: opencv, index_or_path: 4, width: 1280, height: 720, fps: 30},wrist_left: {type: opencv, index_or_path: 2, width: 1280, height: 720, fps: 30} }"     --robot.id=my_biopenarm     --teleop.type=openarm_mini     --teleop.port_left=/dev/ttyACM0     --teleop.port_right=/dev/ttyACM1     --teleop.id=my_leader     --display_data=true     --dataset.repo_id=jogarulfop/shakeitup3     --dataset.num_episodes=10
