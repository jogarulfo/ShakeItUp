# ShakeItUp (Hackathon at GOSIM Paris 2026)
ShakeItUp is project involving a bimanual robot (OpenArm by Enactic) that shakes an opaque bottle, guess its content, and open it only when needed.
This was presented for the Unaite Robotics Hackathon in the conference GOSIM Paris 2026.

The trick is to measure the gripper dynamic deformation (strain) and to use this strain data to guess the content of an opaque bottle. 
To this end, the hackathon experiments have been made with a piezo strain sensor (Dragonfly by Wormsensing, ref. DGF-UNI-W220405-10) glued to the left jaw of the left arm of OpenArm, and an IEPE acquisition system (IOLITE-X by Dewesoft, ref. IOLITE-X-8xACC). 

The strain data was streamed through the open-source openDAQ library (https://docs.opendaq.com/manual/opendaq/3.30/introduction.html) towards the open-source Hugging Face LeRobot library (https://github.com/huggingface/lerobot). You can update the pipeline to match your own acquisition setup.

The code for data acquisition is in the submodule `lespectrobot/src/lerobot/robots`. 
The code is currently working with robots SO-101 and OpenArm (for the bi-OpenArm, just replace any of the two OpenArm by an `OpenFollowerDragonTactile` arm in the file `bi_openarm_follower.py`).


## Installation

Clone the repository

```bash
git clone --recursive https://github.com/jogarulfo/ShakeItUp.git
```

Install dependencies

```bash
uv pip install -e .
cd lespectrobot
uv pip install -e .[damiao] 
```

You would use "feetech" instead of "damiao" if you use SO-100 or SO-101 instead of OpenArm.


## Guide

The following is a step-by-step guide of the commands used during the hackaton to teleoperate, record and train OpenArm with a OpenArm_mini (https://github.com/pkooij/open-arms-mini) as the leader.

### Environment setup for OpenArm

```bash
conda create -n ShakeItUp_env python=3.12 -y
conda activate ShakeItUp_env
conda install -c conda-forge numpy matplotlib scipy polars pyarrow -y
pip install opencv-python opendaq
cd ShakeItUp
pip install -e ".[damiao]"
pip install rerun-sdk
```

<!-- ### Cameras
```bash 
lerobot-find-cameras opencv 
``` -->

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