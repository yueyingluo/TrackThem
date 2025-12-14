# [Track them! Learning Priority to Catch in Flight]

The environment implements a Gymnasium-based reinforcement learning environment 
for Dynamic Catching tasks using MuJoCo physics simulation. 

Agent must touch two objects sequentially with the robot's palm
First touches object1, then switches target to object2
Episode succeeds when both objects are touched

## TODO
- implement two objects [Done]
- object trajectory prediction []
- target change logic []

modified from https://mobile-dex-catch.github.io/

# Installation
- Create conda environment and install pytorch (see pytorch.org):
```
conda create -n dcmm python=3.8
conda activate dcmm
pip install torch torchvision torchaudio
pip install -e
pip install -r requirements.txt
```

# Space Definition
## Observation Space (dim=18)
- base (dim=2): Dict
  - v_lin: 2d linear velocities
- arm (dim=10): Dict
  - ee_pos3d: 3d position of the end-effector
  - ee_v_lin_3d: 3d linear veloity of the end-effector
  - ee_quat: quaternion of the end-effector
- object (dim=6): Dict
  - pos3d: 3d position of the object
  - v_lin_3d: 3d linear velocity of the object
## Actions Space (dim=6)
- base (dim=2): 2d linear velocities of the mobile base
- arm (dim=4): delta x-y-z and delta roll of the arm

# Simulation Environment Test
## Keyboard Control Test
Under `gym_dcmm/envs/`, run:

```bash
python3 DcmmVecEnv.py --viewer
```

Keyboard control:

1. `↑` (up) : increase the y linear velocity (base frame) by 1 m/s;
2. `↓` (down) : decrease the y linear velocity (base frame) by 1 m/s;
3. `←` (left) : increase x linear velocity (base frame) by 1 m/s;
4. `→` (right) : decrease x linear velocity (base frame) by 1 m/s;
5. `4` (turn left) : decrease counter-clockwise angular velocity by 0.2 rad/s;
6. `6` (turn right) : increase counter-clockwise angular velocity by 0.2 rad/s;
7. `+`: increase the position & roll of the arm end effector by (0.1, 0.1, 0.1, 0.1) m;
8. `-`: decrease the position & roll of the arm end effector by (0.1, 0.1, 0.1, 0.1) m;
9. `7`: increase the joint position of the hand by (0.2, 0.2, 0.2, 0.2) rad;
10. `9`: decrease the joint position of the hand by (0.2, 0.2, 0.2, 0.2) rad;

# Test/Train
Under the root `catch_it`:
```bash
python3 train_DCMM.py test=True task=Tracking num_envs=1 checkpoint_tracking=$(path_to_tracking_model)(example pth in assets folder) object_eval=True viewer=$(open_mujoco_viewer_or_not) imshow_cam=$(imshow_camera_or_not)
```
```bash
python3 train_DCMM.py test=False task=Tracking num_envs=$(number_of_CPUs)
```
