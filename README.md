# f1tenth_ws
This a repository that contains ready-to-run autonomous racing packages for the [F1TENTH](https://f1tenth.org/) on ROS2 Foxy. It can be directly be deployed on the physical car. We've also included launch and config files for the [simulation environment](https://github.com/f1tenth/f1tenth_gym_ros), which uses slightly different topics for odometry.

Below is a demo of the car running the code from this repository in the E7 building at the University of Waterloo at a top speed of ~25km/h, record on March 30th 2023.

https://user-images.githubusercontent.com/43485866/231013515-00d6f492-87c0-4cf6-8257-f16cb2daadb6.mp4



## Software Stack Overview
**Our current software stack consists of**
- [slam_toolbox](https://github.com/SteveMacenski/slam_toolbox) for **mapping** (reference the following [slides](https://docs.google.com/presentation/d/1DP2F9l-yHe9gQobk2CzYduk6KR5QtDCp7sLsxqR2fag/edit#slide=id.g115c48c178d_0_1) for running it on the physical car)
- [Particle Filter](./src/particle_filter/) for **localization** $\rightarrow$ `src/particle_filter`
- [Pure Pursuit](./src/pure_pursuit/) for **waypoint following** (planning + control) $\rightarrow$ `src/pure_pursuit`
- [RRT](./src/rrt) for a pure pursuit algorithm that includes **dynamic obstacle avoidance** (slightly slower) $\rightarrow$ `src/rrt`

Racing lines are generated through the [Cl2-UWaterloo/Raceline-Optimization](https://github.com/CL2-UWaterloo/Raceline-Optimization) repository.

**Other algorithms that are not used, but are in this repository include**
- [Waypoint Generator](./src/waypoint_generator/) for manually generating waypoints in simulation $\rightarrow$ `src/waypoint_generator` (this has been replaced with a script that automatically generates optimal racelines given a map)
- A [PID controller](./src/wall_follow/) for staying at a constant distance to the wall $\rightarrow$ `src/wall_follow`
- [Scan matching](./src/scan_matching) $\rightarrow$ `src/scan_matching` (To be completed)
- [gap_follow](./src/gap_follow) $\rightarrow$ `src/gap_follow` (To be completed)

## High-Level Usage Guide
These are the high level steps followed to get the F1TENTH driving in a new location, with [accompanying notes](https://stevengong.co/notes/F1TENTH-Field-Usage):
1. Run SLAM on the physical car to generate a map with `slam_toolbox`
2. Clean up map in Photoshop, and generate a racing line using the [Cl2-UWaterloo/Raceline-Optimization](https://github.com/CL2-UWaterloo/Raceline-Optimization) repository.
3. Store the racing lines under `src/pure_pursuit/racelines/` and `src/rrt/racelines/` (for dynamic obstacle avoidance)
4. Run `particle_filter` with the new map to localize the car properly
5. Run `pure_pursuit` or `rrt` to follow the racing line. Make sure to incrementally increase the `velocity_profile` inside the [config.yaml](./src/pure_pursuit/config/config.yaml) file.


## Getting Started
If you are a complete beginner and want to build an autonomous RC car, you might want to first follow the [F1TENTH Course](https://docs.google.com/spreadsheets/d/1kAd0bf6nc1OVi_4IP1P3-H6PPU97hLjqW8d0mTLCsxg/edit#gid=29915317), where they teach you ROS2 and the intuition behind the algorithms in this repository. The [official documentation](https://f1tenth.readthedocs.io/en/foxy_test/) contains information on setting up the hardware and software for the F1TENTH car.

Once you have access to an F1TENTH vehicle, you can consult our accompanying notes for the commands used to deploy the code in this repository: https://stevengong.co/notes/F1TENTH-Field-Usage. Note that they are mainly written for our own personal reference, so the paths will be different in your setup. **You'll notice that we clone the repository on our local computer, and upload the code over SSH to the car**.

Alternatively, you can try this code out in simulation without a physical car (see below).

## Getting Started in Simulation
If you want to try out the ROS nodes in simulation, you can follow the steps below.We've tested these steps on a Ubuntu 20.04 native machine using [Docker](https://www.docker.com/).

First, we will need to clone the [simulation repository](https://github.com/f1tenth/f1tenth_gym_ros) by running
```bash
git clone https://github.com/f1tenth/f1tenth_gym_ros
```

Then, build the docker image by running:
```
cd f1tenth_gym_ros
sudo docker build -t f1tenth_gym_ros -f Dockerfile .
```

You then need to mount the `f1tenth_ws` repository into the docker container. You will need to change the path in the command below:
```
cd <INSERT_PATH_HERE>/f1tenth_ws/src
sudo rocker --nvidia --x11 --volume .:/sim_ws/src -- f1tenth_gym_ros
```
You will then be entered the docker container. The `f1tenth_ws` code is mounted under `/sim_ws/src/`


Now, we need to load/change the default map in the simulation. In the container, modify the `map_path` parameter inside the `sim.yaml` file (located at `/sim_ws/install/f1tenth_gym_ros/share/f1tenth_gym_ros/config/sim.yaml`) to the path of the map you desire.

For example, 
```yaml
map_path: '/sim_ws/src/particle_filter/maps/e7_floor5'
```

Run the following commands from the `/sim_ws` directory:
```
source /opt/ros/foxy/setup.bash
source install/local_setup.bash
ros2 launch f1tenth_gym_ros gym_bridge_launch.py
```

> Other tip: when your vehicle ends up in really bad spots, you can just use the "2D Pose Estimate" button inside rviz and select where you want to respawn the vehicle. Quite convenient!

To run the ROS2 nodes from this repository (ex: pure_pursuit), you need to run these commands from `/sim_ws/src/f1tenth_gym_ros` in another window (tip: use `tmux`):
```bash
colcon build
. install/setup.bash
ros2 launch pure_pursuit sim_pure_pursuit_launch.py
```
If you don't know what the above commands do, you probably should get familiar with ROS2 first.


## Next Steps
**A non-exhaustive list of things we want to do in the future, include**

- Running SLAM + pure pursuit on the fly, without having to do the offline computation. Something like [this](https://www.youtube.com/watch?v=aCDPwZZm9C4&ab_channel=AMZFormulaStudent) would be incredible
- Trying out MPC, something like [this](https://www.youtube.com/watch?v=JoHfJ6LEKVo&ab_channel=IfAETHZurich) is awesome!

## About Us
The Control, Learning and Logic (CL2) group at the University of Waterloo works on research that aims to develop methods for reliable decision-making of autonomous systems in the wild, led by professor Yash Vardhan Pant. Current members working on the F1TENTH is composed of Steven Gong, Oluwatofolafun Damilola Opeoluwa-Calebs, and Soham Lakhi.
