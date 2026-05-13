# ShakeItUp (GOSIM Paris 2026)
ShakeItUp is a project involving a bimanual robot (OpenArm by Enactic) that shakes an opaque bottle, guesses its content, and opens it only when needed.
This project was presented at the Unaite Robotics Hackathon during the GOSIM Paris 2026 conference.

<p align="center">
  <img src="gif/demo.gif" width="400"/>
  <br>Teleoperation during GOSIM Paris 2026</em>
</p>

The idea is to measure the dynamic deformation (strain) of the gripper and use this strain data to infer the content of an opaque bottle.

To achieve this, the hackathon experiments were performed using:
- a piezo strain sensor (Dragonfly by Wormsensing, ref. DGF-UNI-W220405-10) glued to the left jaw of the left OpenArm gripper
- an IEPE acquisition system (Dewesoft, ref. IOLITE-X-8xACC)

The strain data was streamed through the open-source openDAQ library (https://docs.opendaq.com/manual/opendaq/3.30/introduction.html) into the open-source Hugging Face LeRobot library (https://github.com/huggingface/lerobot).
You can adapt the pipeline to match your own acquisition setup.

The code for data acquisition is located in the submodule `lespectrobot/src/lerobot/robots`.
The code currently supports the SO-101 and OpenArm robots. 
For a bimanual OpenArm setup, simply replace one of the two OpenArm instances with an `OpenFollowerDragonTactile` arm in the file `bi_openarm_follower.py`.


## Installation

Clone the repository

```bash
git clone --recursive https://github.com/jogarulfo/ShakeItUp.git
```

Install dependencies

```bash
uv venv
uv pip install -e .
cd lespectrobot
uv pip install -e .[damiao] 
```

You would use "feetech" instead of "damiao" if you use SO-100 or SO-101 instead of OpenArm.


## Guide

The following is a step-by-step guide to the commands used during the hackathon to teleoperate, record, and train OpenArm using an OpenArm Mini (https://github.com/pkooij/open-arms-mini) as the leader arm.

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

### Calibrations
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

Leader Arm (Teleoperator)
```bash
lerobot-calibrate --teleop.type=openarm_mini --teleop.port_left=/dev/ttyACM3 --teleop.port_right=/dev/ttyACM2   --teleop.id=my_leader
```

### Bimanual teleoperation without cameras

To teleoperate a bimanual OpenArm setup with two leader arms and two follower arms
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

### Bimanual teleoperation with cameras 

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

To record a dataset during teleoperation
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