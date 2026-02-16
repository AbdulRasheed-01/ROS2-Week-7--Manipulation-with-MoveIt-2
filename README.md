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

Once RViz launches:

Set Planning Group to panda_arm or hand

Use interactive markers to set goal pose

Click Plan to visualize path

Click Execute to animate

1.3 Command-Line Planning:

    # List available planning groups
    ros2 param get /move_group robot_description_planning
    
    # Send planning goal via action
    ros2 action send_goal /plan_kinematic_path moveit_msgs/action/GetMotionPlan "
    motion_plan_request:
      workspace_parameters:
        header:
          frame_id: panda_link0
        min_corner: [-1, -1, -1]
        max_corner: [1, 1, 1]
      start_state:
        is_diff: true
      goal_constraints:
        - joint_constraints:
            - joint_name: panda_joint1
              position: 0.0
            - joint_name: panda_joint2
              position: -0.3
            - joint_name: panda_joint3
              position: 0.0
            - joint_name: panda_joint4
              position: -2.0
            - joint_name: panda_joint5
              position: 0.0
            - joint_name: panda_joint6
              position: 2.0
            - joint_name: panda_joint7
              position: 0.8
      pipeline_id: ompl
      planner_id: RRTConnectkConfigDefault"

Exercise 2: Python Interface with PyMoveIt2 

2.1 Basic MoveIt 2 Python Node:

Create robot_manipulation/moveit/basic_move.py:

    #!/usr/bin/env python3
    import rclpy
    from rclpy.node import Node
    from pymoveit2 import MoveIt2
    from pymoveit2.robots import panda
    import time
    
    class BasicManipulation(Node):
        def __init__(self):
            super().__init__('basic_manipulation')
            
            # Create MoveIt 2 interface
            self.moveit2 = MoveIt2(
                node=self,
                joint_names=panda.joint_names(),
                base_link_name=panda.base_link_name(),
                end_effector_name=panda.end_effector_name(),
                group_name=panda.MOVE_GROUP_ARM
            )
            
            self.get_logger().info("Basic Manipulation Node Started")
        
        def move_to_joint_pose(self, joint_positions):
            """Move to specified joint positions"""
            self.get_logger().info(f"Moving to joint positions: {joint_positions}")
            
            # Plan and execute
            self.moveit2.move_to_configuration(joint_positions)
            self.moveit2.wait_until_executed()
            
            self.get_logger().info("Movement completed")
        
        def move_to_pose(self, position, quaternion):
            """Move to Cartesian pose"""
            self.get_logger().info(f"Moving to pose: pos={position}, quat={quaternion}")
            
            # Plan and execute
            self.moveit2.move_to_pose(
                position=position,
                quaternion=quaternion,
                cartesian_path=False  # Use joint space planning
            )
            self.moveit2.wait_until_executed()
            
            self.get_logger().info("Movement completed")
        
        def open_gripper(self):
            """Open the gripper"""
            self.get_logger().info("Opening gripper")
            self.moveit2.move_to_configuration(panda.OPEN_GRIPPER)
            self.moveit2.wait_until_executed()
        
        def close_gripper(self):
            """Close the gripper"""
            self.get_logger().info("Closing gripper")
            self.moveit2.move_to_configuration(panda.CLOSE_GRIPPER)
            self.moveit2.wait_until_executed()
    
    def main(args=None):
        rclpy.init(args=args)
        node = BasicManipulation()
        
        # Wait for MoveIt 2 to initialize
        time.sleep(2.0)
        
        # Example: Move to home position
        node.move_to_joint_pose(panda.HOME_JOINTS)
        time.sleep(1.0)
        
        # Example: Move to Cartesian pose
        node.move_to_pose(
            position=[0.5, 0.1, 0.5],
            quaternion=[0.0, 0.0, 0.0, 1.0]
        )
        time.sleep(1.0)
        
        # Example: Open/close gripper
        node.open_gripper()
        time.sleep(1.0)
        node.close_gripper()
        
        # Spin to keep node alive
        rclpy.spin(node)
        
        node.destroy_node()
        rclpy.shutdown()
    
    if __name__ == '__main__':
        main()

