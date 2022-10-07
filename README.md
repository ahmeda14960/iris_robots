# iris_robots: Franka
## Adaptation of polymetis for Franka 


## Install instructions
If working on a workstation / separate machine from controller (NUC), make a conda env using `conda create -n polymetis python=3.8`

Install polymetis as `conda install -c pytorch -c fair-robotics -c aihabitat -c conda-forge polymetis`

Note: This will take a long time to populate (few minutes), be patient
Also need to install
gym
OpenCV
pygame
ipdb
absl

(Put these in a requirements.txt file or yml)

# Notes
The RobotEnv is a gym wrapper around the polymetis logic.
This is what takes in the step function, etc and does
The functionality we would expect from an agnostic
gym wrapper.

The FrankaRobot is a class that will launch a RoBotInterface
class internally to ping the robot at the desired ip address.
Think of FrankaRobot as a wrapper around each of the
ip commands we use to ping / recieve commands to / from the 
server and then to the robot

Ask about update gripper function

Running separate server for franka gripper is
`` launch_gripper.py gripper=franka_hand ``

Protobuf version that comes with polymetis gives error
TypeError: bases must be types

to get around it uninstall then install earlier version of protobuf

`` pip uninstall protobuf ``

`` pip install protobuf==3.20.1 `` 

Need to also install dm_control (latest verison) and dm_robotics package to
use the IK solver

``pip install dm-robotics-moma``

Also when installing on NUC to get server running conda commands don't work,
go through local server install process

# Start server
Note that you should add the iris_robots repo to the python
path of whichever machine you're using. For example

```
export PYTHONPATH=$PYTHONPATH:~/iris_robots/
```

First you must start a server to communicate with the robot
on the NUC. Thankfully, once you go through the **local** polymetis
instruction cycle on the NUC, running the server should be straight forward.
Simply start a tmux session and run the following commands

```
sudo pkill -9 run_server
conda activate polymetis-local
launch_robot.py robot_client=franka_hardware robot_client.executable_cfg.robot_ip=172.16.0.2
```

Then create a new screen within tmux (CTRL + C) and cycle to it (CTRL + N),
and run the following command to start the robotiq gripper. You might get an error
that the command was rejected and the gripper is not activated, but if the gripper moved
try the command again and it should work. The terminal should prompt that the 
robotiq gripper is activated
```
sudo chmod a+rw /dev/ttyUSB0
conda activate polymetis-local
launch_gripper.py gripper=robotiq_2f gripper.comport=/dev/ttyUSB0
```

Finally, create a new screen and run the flask server, which communicates between
the nuc / workstation. Note you ONLY need this if you're running on a workstation,
otherwise pass in the ip address as None to the robot env when running on the NUC.
```
conda activate polymetis-local
python ~/iris_robots/run_server.py
```
# Demo Collection
The repo should already have the module/submodule structure created
so run the demo recording script as a module
```
python -m iris_robots.record_demos
```
# Error FAQ
If when running the server, you see the following or similar error
```
dm_control/mjcf/physics.py", line 495, in from_mjcf_model
    return cls.from_xml_string(xml_string=xml_string, assets=assets)
  File "/home/panda5/anaconda3/envs/polymetis-local/lib/python3.8/site-packages/dm_control/mujoco/engine.py", line 424, in from_xml_string
    return cls.from_model(model)
  File "/home/panda5/anaconda3/envs/polymetis-local/lib/python3.8/site-packages/dm_control/mujoco/engine.py", line 407, in from_model
    return cls(data)
  File "/home/panda5/anaconda3/envs/polymetis-local/lib/python3.8/site-packages/dm_control/mujoco/engine.py", line 122, in __init__
    self._reload_from_data(data)
  File "/home/panda5/anaconda3/envs/polymetis-local/lib/python3.8/site-packages/dm_control/mjcf/physics.py", line 530, in _reload_from_data
    super()._reload_from_data(data)
  File "/home/panda5/anaconda3/envs/polymetis-local/lib/python3.8/site-packages/dm_control/mujoco/engine.py", line 370, in _reload_from_data
    self._warnings_before = np.empty_like(self._warnings)
  File "<__array_function__ internals>", line 180, in empty_like
RuntimeError: Caught an unknown exception! 
```

There is a mismatch between the versions installed by the local version of polymetis
and the versions you need
```
pip install mujoco==2.2.1
pip install dm-control==1.0.5
```
### Cameras are not being detected
If you are using local polymetis to control the robot, make sure you are not running a server.
The server would have taken over the cameras and for security reasons the local env cannot use them.

## User stop mode
If the robot enters user stop mode, the polymetis-server will crash unfortunately. If the robot
needs a manual reset, make sure the training script is halted, then perform the manual reset
and ensure that a new polymetis server is running before continuing.

## TODO
Resolve 'Get called buffer size' which repeatedly prints a growing buffer
size to the terminal log. Doesn't seem to affect functionality but Sasha didn't see it

Make it so we can vary image size as a flag to robot-env

go to rest, rename to randomize position, just take some random 
actions at center

clean up get observation, just return all info and remove
viz and demo collection flags

Add demo collection script to repo