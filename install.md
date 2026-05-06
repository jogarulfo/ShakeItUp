# Installation guide

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

#### Camera
```bash 
lerobot-find-cameras opencv 
```


# Setup CAN interfaces
```bash
lerobot-setup-can --mode=setup --interfaces=can0,can1
```
# Test motor communication
```bash
lerobot-setup-can --mode=test --interfaces=can0,can1
```


### Teleoperation and Data Collection
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



#### Leader Arm (Teleoperator)


leader_right : /dev/ttyACM0
leader_left : /dev/ttyACM1
```bash
lerobot-calibrate --teleop.type=openarm_mini --teleop.port_left=/dev/ttyACM3 --teleop.port_right=/dev/ttyACM2   --teleop.id=my_leader
```
Teleoperation

Bimanual Teleoperation

To teleoperate a bimanual OpenArm setup with two leader and two follower arms:
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

#### w cam : 

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
Recording Data

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


Inférence : 

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