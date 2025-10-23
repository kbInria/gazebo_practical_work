# Gazebo practical work
This project provides hands-on experience with ROS 2 and the Gazebo simulator.

## Included packages

* `ros_gz_example_description` - holds the sdf description of the simulated system and any other assets.

* `ros_gz_example_gazebo` - holds gazebo specific code and configurations. Namely this is where systems end up.

* `ros_gz_example_application` - holds ros2 specific code and configurations.

* `ros_gz_example_bringup` - holds launch files and high level utilities.


## Install *(5 min)*

### Requirements

Pull the docker image containing ros2 humble and gazebo fortress

    ```bash
    docker pull ghcr.io/sloretz/ros:humble-simulation
    ```

### Installations

1. Run the docker image **in the folder containing the clone of this repository**

    ```bash
    docker run -it --rm --name="gazebo_simulator" --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" --volume="./gazebo_practical_work:/opt/catkin_ws:rw" --env="DISPLAY" -e XDG_RUNTIME_DIR=$XDG_RUNTIME_DIR --gpus all --device=/dev/dri ghcr.io/sloretz/ros:humble-simulation bash
    ```

1. If it is the first run, install and build as follow

    ```bash
    apt update && apt install -y ros-humble-rviz2
    cd ~/catkin_ws
    source /opt/ros/<ROS_DISTRO>/setup.bash
    colcon build --cmake-args -DBUILD_TESTING=ON
    . ~/catkin_ws/install/setup.sh
    ```

    and in another terminal

    ```
    docker commit gazebo_simulator ghcr.io/sloretz/ros:humble-simulation
    ```

1. Run the following simulation to check the installations

    ```bash
    ros2 launch ros_gz_example_bringup diff_drive.launch.py
    ```

**Note:** You can always open a new bash session to the docker image by running in another terminal


```bash
docker exec -it gazebo_simulator bash
```


## Practical work

### Navigating the project *(5 min)*

This will help you understand where the main files you'll need to modify are located.

1. Find the path to the worlds folder
    <details>
        <summary>Answer</summary>
        <code>gazebo_practical_work/ros_gz_example_gazebo/worlds</code>
    </details>
    <br>

1. Find the path to the models folder
    <details>
        <summary>Answer</summary>
        <code>gazebo_practical_work/ros_gz_example_description/models</code>
    </details>
    <br>

1. Find the path to the launch folder
    <details>
        <summary>Answer</summary>
        <code>gazebo_practical_work/ros_gz_example_bringup/launch</code>
    </details>
    <br>

1. Check the gazebo path variables<br>
    `env | grep -Ei 'gazebo|ign|gz_'`

**Note:** Keep in mind that a model or a plugin that is not part of this path will not be found by gazebo!

### Launching worlds *(10 min)*

Using [this tutorial](https://docs.ros.org/en/humble/Tutorials/Advanced/Simulators/Gazebo/Gazebo.html), do the following:

1. Launch an empty world with gazebo only
    <details>
        <summary>Answer</summary>
        <code>ign gazebo -v 4 -r default.sdf</code>
    </details>
    <br>

1. Get the list of all the gazebo topics when the simulation is launched (in another terminal)
    <details>
        <summary>Answer</summary>
        <code>ign topic -l</code>
    </details>
    <br>

1. In the GUI, add any shape and check it in the entity tree

1. Check the worlds folder and launch a simulation with a world containing a model
    <details>
        <summary>Answer</summary>
        <code>ign gazebo -v 4 -r diff_drive.sdf</code>
    </details>
    <br>

### Communicating with ROS2 *(20 min)*

So far, only gazebo was launched. We have seen that it publishes topics that can be listened to or published to. In this section, let's see how ROS2 can interact with gazebo.

To do so, we will need to bridge topics from gazebo to/from ros2. This will done using this [ros2 package](https://github.com/gazebosim/ros_gz/tree/ros2/ros_gz_bridge), check it for more information about ros2 bridge and correspondences between topics.

1. Listen to all ros2 topics
    <details>
        <summary>Answer</summary>
        <code>ros2 topic list</code>
    </details>
    <br>

1. Bridge the topic `/model/diff_drive/odometry` from gazebo to ros2 (bilateral)
    <details>
    <summary>Answer</summary>

    ```bash
    ros2 run ros_gz_bridge parameter_bridge /diff_drive/odometry@nav_msgs/msg/Odometry@gz.msgs.Odometry
    ```
    </details>

1. Check if the bridge was successful
    ```bash
    ign topic -e -t /model/diff_drive/odometry
    ros2 topic echo /model/diff_drive/odometry
    ```
1. Ros2 bridge can also be configured using a yaml file. Find the bridge configuration yaml file in the project
    <details>
        <summary>Answer</summary>
        <code>ros_gz_example_bringup/config/ros_gz_example_bridge.yaml</code>
    </details>
    <br>

1. Bridge all the simulation topics
    <details>
        <summary>Answer</summary>
        <code>ros2 run ros_gz_bridge parameter_bridge     --ros-args     -p config_file:=/root/catkin_ws/src/ros_gz_example_bringup/config/ros_gz_example_bridge.yaml</code>
    </details>
    <br>

1. Now that all the topics are parts of the ros2 bridge, send velocities command to the robot so that it turns in circle
    <details>
        <summary>Answer</summary>
        <code>ros2 topic pub /diff_drive/cmd_vel geometry_msgs/msg/Twist "angular: { z: 0.1 }"</code>
    </details>
    <br>

### Customizing a simulation *(30 min)*

The current robot does not have any sensors. Using [this tutorial](https://gazebosim.org/docs/latest/sensors/#lidar-sensor) let's add a gpu lidar to the robot.

1. The robot already has a `lidar_link`. Add the sensor to that link
    <details>
        <summary>Which file should be modify?</summary>
        <code>gazebo_practical_work/ros_gz_example_description/models/diff_drive/model.sdf</code>
    </details>
    <details>
    <summary>What should be added?</summary>

    ```xml
        <sensor name='gpu_lidar' type='gpu_lidar'>
          <pose>0 0 0 0 0 0</pose>
          <topic>scan</topic>
          <ignition_frame_id>diff_drive/lidar_link</ignition_frame_id>
          <update_rate>10</update_rate>
          <lidar>
            <scan>
              <horizontal>
                <samples>640</samples>
                <resolution>1</resolution>
                <min_angle>-1.396263</min_angle>
                <max_angle>1.396263</max_angle>
              </horizontal>
              <vertical>
                <samples>1</samples>
                <resolution>1</resolution>
                <min_angle>0.0</min_angle>
                <max_angle>0.0</max_angle>
              </vertical>
            </scan>
            <range>
              <min>0.08</min>
              <max>10.0</max>
              <resolution>0.01</resolution>
            </range>
          </lidar>
          <visualize>true</visualize>
        </sensor>
    ```
    </details>
    <br>

1. The `lidar_link` exists but it cannot be added to the robot model because no information is given about its relative pose on the robot. To give this information, you need to write a join between the `lidar_link` and the `chassis` links.
    <details>
        <summary>Which file should be modify?</summary>
        <code>gazebo_practical_work/ros_gz_example_description/models/diff_drive/model.sdf</code>
    </details>
    <details>
    <summary>What should be added?</summary>

    ```xml
        <joint name='lidar_joint' type='fixed'>
            <parent>chassis</parent>
            <child>lidar_link</child>
        </joint>
    ```
    </details>
    <br>

1. Rerun the simulation and check that the topic is being published in gazebo<br>
    `ign topic -e -t /scan`

1. Yet the lidar topic is not available to ros2. Add this connection to the bridge configuration yaml file
    <details>
    <summary>What should be added?</summary>

    ```yml
          - ros_topic_name: "/diff_drive/scan"
            gz_topic_name: "/scan"
            ros_type_name: "sensor_msgs/msg/LaserScan"
            gz_type_name: "gz.msgs.LaserScan"
            direction: GZ_TO_ROS
    ```
    </details>
    <br>

1. Rerun the simulation (gazebo + bridge) and check that the topic is being published in gazebo<br>
    `ros2 topic echo /diff_drive/scan`

1. It is fastidious to run each command in  separate terminals. We want to write a launchfile so that every part of the simulation is launched simultaneously. Complete the `gazebo_practial_work.launch.py` launchfile so that it launches the gazebo simulation. You can find more information about launchfiles [here](https://docs.ros.org/en/humble/Tutorials/Intermediate/Launch/Launch-system.html) and [here](https://github.com/ros2/launch/blob/10b99ae4e1b53bf0bb6cbd00b638e82127aca4bd/launch/launch/launch_description_sources/python_launch_description_source.py#L25).
    <details>
    <summary>What should be added?</summary>

    ```py
        gz_sim = IncludeLaunchDescription(
            PythonLaunchDescriptionSource(
                os.path.join(pkg_ros_gz_sim, 'launch', 'gz_sim.launch.py')),
            launch_arguments={'gz_args': PathJoinSubstitution([
                pkg_project_gazebo,
                'worlds',
                'diff_drive.sdf'
            ])}.items(),
        )
    ```
    </details>
    <br>


### Quizz *(10 min)*

If everything about this practical work is clear, you should be able to answer the following questions:

1. Which command is used to source the ROS 2 Humble environment in a new terminal?
    <details>
     <summary>Answer</summary>
        source /opt/ros/humble/setup.bash
    </details>
    <br>

1. What are the different steps to adding a new sensor to a model?
    <details>
     <summary>Answer</summary>
        1. Defining a sensor link to the model<br>
        2. Adding a sensor plugin to the link<br>
        3. Creating a joint between the sensor link and any of the model frame<br>
        4. Bridging the gazebo topic to a ros2 topic 
    </details>
    <br>

1. What are the different steps to adding a new sensor to a model?
    <details>
     <summary>Answer</summary>
        1. Defining a sensor link to the model<br>
        2. Adding a sensor plugin to the link<br>
        3. Creating a joint between the sensor link and any of the model frame<br>
        4. Bridging the gazebo topic to a ros2 topic 
    </details>
    <br>

1. The GPU lidar we are using is currently 2D. What should we do to make it 3D?
    <details>
    <summary>Answer</summary>

    ```diff
        <lidar>
            <scan>
              <horizontal>
                <samples>640</samples>
                <resolution>1</resolution>
                <min_angle>-1.396263</min_angle>
                <max_angle>1.396263</max_angle>
              </horizontal>
              <vertical>
                <samples>1</samples>
                <resolution>1</resolution>
    -           <min_angle>0.0</min_angle>
    -           <max_angle>0.0</max_angle>
    +           <min_angle>-0.2</min_angle>
    +           <max_angle>0.2</max_angle>
              </vertical>
            </scan>
        <!-- ... -->
        </lidar>
    ```
    </details>
    <br>

1. I have created and compiled (it's c++!) a new plugin but it is not loaded in my simulation. What could be the problem?
    <details>
     <summary>Answer</summary>
        Check the gazebo path variables:  <code>env | grep -Ei 'gazebo|ign|gz_'</code>
    </details>
    <br>