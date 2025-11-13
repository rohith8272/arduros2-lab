# arduros2-lab
A Docker-based setup for running ROS2 with ArduPilot SITL (Software In The Loop) simulation.

## Prerequisites

- Docker
- Docker Compose

## Quick Start

### 1. Clone the Repository
```bash
git clone https://github.com/rohith8272/arduros2-lab.git
cd arduros2-lab
```

### Build and Run the Docker Container
Start docker desktop first.
```bash
docker compose build
docker compose run --rm ardupilot_ros bash
```
By using the --rm option, Docker will automatically clean up the container once you close the application. 

### Build ROS2 Workspace (Inside Container)
```bash
cd ~/ros2_ws
colcon test --packages-select ardupilot_dds_tests
colcon build --packages-up-to ardupilot_sitl

colcon build --packages-up-to ardupilot_gz_bringup
colcon build --packages-select ardupilot_gazebo --symlink-install
```

### Source the Workspace
```bash
source ~/ros2_ws/install/setup.bash
```

### Terminal 1: Start ArduPilot SITL

# Run the DDS AP SITL (inside container)
```bash
ros2 launch ardupilot_sitl sitl_dds_udp.launch.py \
  transport:=udp4 \
  synthetic_clock:=True \
  wipe:=False \
  model:=quad \
  speedup:=1 \
  slave:=0 \
  instance:=0 \
  defaults:=$(ros2 pkg prefix ardupilot_sitl)/share/ardupilot_sitl/config/default_params/copter.parm,$(ros2 pkg prefix ardupilot_sitl)/share/ardupilot_sitl/config/default_params/dds_udp.parm \
  sim_address:=127.0.0.1 \
  master:=tcp:127.0.0.1:5760 \
  sitl:=127.0.0.1:5501
```

### Terminal 2: Connect with MAVProxy

```bash
# Find your running container
docker ps

# Execute into the running container
docker exec -it <your-ROS-container-ID> bash

# Inside container, run MAVProxy
source ~/ros2_ws/install/setup.bash
mavproxy.py --console --map --aircraft test --master=:14550
```

### Terminal 3: Monitor ROS2 Nodes and Topics
```bash
# Execute into container (if not already inside)
docker exec -it <your-ROS-container-ID> bash

# Source workspace and check ROS2 nodes
source ~/ros2_ws/install/setup.bash

ros2 node list
ros2 topic list
ros2 node info /ap

#check battery topic being published
ros2 topic echo /ap/battery
```
### Network Configuration
The setup uses the following ports:

MAVLink: 14550 (UDP)

SITL: 5501 (TCP)

Master: 5760 (TCP)
