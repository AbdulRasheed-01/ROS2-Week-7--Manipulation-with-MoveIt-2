# ROS2-Week-7--Manipulation-with-MoveIt-2
By the end of this week, you will be able to:

✅ Understand MoveIt 2 architecture and components

✅ Set up MoveIt 2 for robotic arms

✅ Configure motion planning pipelines

✅ Implement pick and place operations

✅ Use MoveIt Task Constructor for complex tasks

✅ Integrate perception with manipulation

✅ Handle collision avoidance and motion planning

✅ Deploy on real robots with ros2_control

📚 Theory Content

7.1 Core Components :


Component	          |    Purpose	                  |     Key Features

MoveGroup Interface	|    Main user-facing API	      |     Plan, execute, add objects

Planning Pipeline	  |    Sequence of planners	      |     OMPL, Pilz, STOMP

Planning Scene	    |    World representation	      |     Collision objects, constraints

Robot Model	        |    URDF/SRDF representation	  |     Kinematics, joints, links

Motion Planners	    |    Path planning algorithms	  |    RRT, PRM, LazyPRM

Kinematics	        |    Forward/Inverse kinematics	|    KDL, TRAC-IK, BioIK

Stage Types 

Generators: Create independent solutions (IK sampling for grasp poses)

Propagators: Extend from start/goal (Cartesian approach paths)

Connectors: Bridge between states (free-space motion)

Wrappers: Filter/modify solutions (constraint checking)

Containers: Sequence or parallel stages

7.2 Collision Checking and Planning Scene

Planning Scene Layers:

Robot Model: Self-collisions, joint limits

World Objects: Static collision objects

Attached Objects: Objects held by gripper

Constraints: Joint/position constraints


7.5 Perception Integration

Object Detection and Pose Estimation :

Method	                         |       Use Case	                |    Performance

FoundationPose	                 |       Novel objects, accurate	|      ~1 FPS

DOPE (Deep Object Pose Estimation)	|    Trained objects, faster	|      ~5 FPS

RT-DETR	                            |    2D detection only	      |       Real-time

Segment Anything	                  |    Segmentation masks	      |      ~10 FPS

⚙️ Setup and Installation

Step 1: Install MoveIt 2 and Dependencies

    # Install MoveIt 2 core packages
    sudo apt-get install ros-humble-moveit
    sudo apt-get install ros-humble-moveit-resources  
    sudo apt-get install ros-humble-moveit-visual-tools
    sudo apt-get install ros-humble-moveit-servo
    sudo apt-get install ros-humble-geometric-shapes

    # Install planners
    sudo apt-get install ros-humble-moveit-planners-ompl
    sudo apt-get install ros-humble-pilz-industrial-motion-planner
    sudo apt-get install ros-humble-moveit-planners-chomp

    # Install kinematics
    sudo apt-get install ros-humble-moveit-kinematics
    sudo apt-get install ros-humble-trac-ik-kinematics-plugin

    # Install MoveIt Task Constructor
    sudo apt-get install ros-humble-moveit-task-constructor-core
    sudo apt-get install ros-humble-moveit-task-constructor-demos
    sudo apt-get install ros-humble-moveit-task-constructor-capabilities
    sudo apt-get install ros-humble-moveit-task-constructor-visualization

    # Install PyMoveIt2 (Python bindings) [citation:3]
    pip3 install pymoveit2

    # Or build from source
    cd ~/ros2_ws/src
    git clone https://github.com/AndrejOrsula/pymoveit2.git
    cd ~/ros2_ws
    colcon build --packages-select pymoveit2

    # Install additional tools
    sudo apt-get install ros-humble-joint-state-publisher-gui
    sudo apt-get install ros-humble-xacro
    sudo apt-get install ros-humble-tf2-tools
    sudo apt-get install ros-humble-ros2-control ros-humble-ros2-controllers
    sudo apt-get install ros-humble-control-toolbox

Step 2: Install Robot Drivers and Simulation

    # Install Universal Robots driver (for UR robots) [citation:3][citation:10]
    sudo apt-get install ros-humble-ur
    sudo apt-get install ros-humble-ur-robot-driver
    sudo apt-get install ros-humble-ur-calibration
    sudo apt-get install ros-humble-joint-trajectory-controller

    # Or build from source for latest version
    cd ~/ros2_ws/src
    git clone -b humble https://github.com/UniversalRobots/Universal_Robots_ROS2_Driver.git
    git clone -b humble https://github.com/UniversalRobots/Universal_Robots_ROS2_Gazebo_Simulation.git
    git clone -b ros2 https://github.com/ros-industrial/universal_robot.git

    # Install Gazebo simulation packages
    sudo apt-get install ros-humble-gazebo-ros-pkgs
    sudo apt-get install ros-humble-gazebo-ros2-control

    # Install Panda robot resources
    sudo apt-get install ros-humble-moveit-resources-panda-moveit-config
    sudo apt-get install ros-humble-franka-description

Step 3: Create Manipulation Package

    cd ~/ros2_ws/src
    ros2 pkg create robot_manipulation --build-type ament_python \
        --dependencies rclpy rclcpp moveit_msgs moveit_ros_planning_interface \
                     moveit_visual_tools moveit_task_constructor_core \
                     geometric_shapes tf2_ros geometry_msgs \
        --description "Week 7: Robotic Manipulation with MoveIt 2"
    
    cd robot_manipulation
    mkdir -p robot_manipulation/{moveit,perception,controllers,pick_place}
    mkdir -p {config,launch,urdf,meshes,scenes,behaviors}
    mkdir -p config/{moveit_params,srdf,kinematics}
    mkdir -p scenes/{pick_place,assembly}

🔧 Practical Exercises

Exercise 1: First Steps with MoveIt 2 and Panda Robot

1.1 Launch Panda Robot in RViz with MoveIt 2 

    # Terminal 1: Launch Panda with MoveIt 2
    ros2 launch moveit_resources_panda_moveit_config demo.launch.py
    
    # Alternative with custom RViz config
    ros2 launch moveit_resources_panda_moveit_config demo.launch.py \
        rviz_config:=/opt/ros/humble/share/moveit_resources_panda_moveit_config/config/moveit.rviz
    
1.2 Basic Motion Planning with GUI:

