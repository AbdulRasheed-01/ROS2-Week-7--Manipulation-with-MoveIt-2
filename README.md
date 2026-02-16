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
2.2 Safe Movement with Workspace Constraints 

Create robot_manipulation/moveit/safe_move.py:

    #!/usr/bin/env python3
    import rclpy
    from rclpy.node import Node
    from pymoveit2 import MoveIt2
    from pymoveit2.robots import panda
    import numpy as np
    
    class SafeManipulation(Node):
        def __init__(self):
            super().__init__('safe_manipulation')
            
            self.moveit2 = MoveIt2(
                node=self,
                joint_names=panda.joint_names(),
                base_link_name=panda.base_link_name(),
                end_effector_name=panda.end_effector_name(),
                group_name=panda.MOVE_GROUP_ARM
            )
            
            # Workspace boundaries (example values for Panda) [citation:3]
            self.workspace_limits = {
                'x': {'min': 0.15, 'max': 0.68},
                'y': {'min': -0.41, 'max': 0.41},
                'z': {'min': 0.10, 'max': 0.60}
            }
        
        def is_pose_safe(self, position):
            """Check if target position is within workspace"""
            x, y, z = position
            
            if not (self.workspace_limits['x']['min'] <= x <= self.workspace_limits['x']['max']):
                self.get_logger().error(f"UNSAFE: X={x:.3f} outside range")
                return False
            
            if not (self.workspace_limits['y']['min'] <= y <= self.workspace_limits['y']['max']):
                self.get_logger().error(f"UNSAFE: Y={y:.3f} outside range")
                return False
            
            if not (self.workspace_limits['z']['min'] <= z <= self.workspace_limits['z']['max']):
                self.get_logger().error(f"UNSAFE: Z={z:.3f} outside range")
                return False
            
            self.get_logger().info(f"SAFE: Position {position} within bounds")
            return True
        
        def safe_move_to_pose(self, position, quaternion):
            """Move only if target is safe"""
            if not self.is_pose_safe(position):
                self.get_logger().error("Movement aborted - target unsafe")
                return False
            
            self.get_logger().info(f"Moving to safe pose: {position}")
            self.moveit2.move_to_pose(position, quaternion)
            self.moveit2.wait_until_executed()
            return True
        
        def add_collision_box(self, name, position, size):
            """Add collision object to planning scene"""
            self.get_logger().info(f"Adding collision box: {name}")
            
            # Use MoveIt 2 collision object interface
            self.moveit2.add_collision_box(
                name=name,
                position=position,
                quaternion=[0.0, 0.0, 0.0, 1.0],
                size=size,
                frame_id=panda.base_link_name()
            )
        
        def remove_collision_object(self, name):
            """Remove collision object from planning scene"""
            self.get_logger().info(f"Removing collision object: {name}")
            self.moveit2.remove_collision_object(name)
    
    def main(args=None):
        rclpy.init(args=args)
        node = SafeManipulation()
        
        # Add a table to the planning scene
        node.add_collision_box(
            name="table",
            position=[0.4, 0.0, -0.1],
            size=[0.8, 0.8, 0.2]
        )
        
        # Try safe movement
        node.safe_move_to_pose(
            position=[0.5, 0.2, 0.3],
            quaternion=[0.0, 0.0, 0.0, 1.0]
        )
        
        # This should fail (outside workspace)
        node.safe_move_to_pose(
            position=[1.0, 0.5, 0.8],
            quaternion=[0.0, 0.0, 0.0, 1.0]
        )
        
        rclpy.spin(node)
        node.destroy_node()
        rclpy.shutdown()
    
    if __name__ == '__main__':
        main()
Exercise 3: MoveIt Task Constructor for Pick and Place 

3.1 Install MTC Demos and Dependencies:

    # Install MTC demos
    sudo apt-get install ros-humble-moveit-task-constructor-demos
    
    # Clone MTC source for tutorials
    cd ~/ros2_ws/src
    git clone -b humble https://github.com/moveit/moveit_task_constructor.git
    cd ~/ros2_ws
    rosdep install --from-paths . --ignore-src -y
    colcon build --packages-select moveit_task_constructor_core moveit_task_constructor_demo
        
3.2 Run MTC Pick and Place Demo:

    # Terminal 1: Launch Panda environment
    ros2 launch moveit_task_constructor_demo demo.launch.py
    
    # Terminal 2: Run pick and place demo
    ros2 launch moveit_task_constructor_demo run.launch.py exe:=pick_place_demo
3.3 Custom MTC Node in C++:

Create src/mtc_pick_place_node.cpp:

    #include <rclcpp/rclcpp.hpp>
    #include <moveit/planning_scene/planning_scene.hpp>
    #include <moveit/planning_scene_interface/planning_scene_interface.hpp>
    #include <moveit/task_constructor/task.h>
    #include <moveit/task_constructor/solvers.h>
    #include <moveit/task_constructor/stages.h>
    #include <tf2_geometry_msgs/tf2_geometry_msgs.hpp>
    #include <tf2_eigen/tf2_eigen.hpp>
    
    static const rclcpp::Logger LOGGER = rclcpp::get_logger("mtc_pick_place");
    namespace mtc = moveit::task_constructor;
    
    class MTCPickPlaceNode
    {
    public:
      MTCPickPlaceNode(const rclcpp::NodeOptions& options);
      
      rclcpp::node_interfaces::NodeBaseInterface::SharedPtr getNodeBaseInterface();
      
      void setupPlanningScene();
      void doTask();
      mtc::Task createTask();
    
    private:
      rclcpp::Node::SharedPtr node_;
      mtc::Task task_;
    };
    
    MTCPickPlaceNode::MTCPickPlaceNode(const rclcpp::NodeOptions& options)
      : node_{ std::make_shared<rclcpp::Node>("mtc_node", options) }
    {
    }
    
    rclcpp::node_interfaces::NodeBaseInterface::SharedPtr MTCPickPlaceNode::getNodeBaseInterface()
    {
      return node_->get_node_base_interface();
    }
    
    void MTCPickPlaceNode::setupPlanningScene()
    {
      // Add a cylinder object to pick
      moveit_msgs::msg::CollisionObject object;
      object.id = "cylinder";
      object.header.frame_id = "world";
      object.primitives.resize(1);
      object.primitives[0].type = shape_msgs::msg::SolidPrimitive::CYLINDER;
      object.primitives[0].dimensions = { 0.1, 0.02 };  // height, radius
    
      geometry_msgs::msg::Pose pose;
      pose.position.x = 0.4;
      pose.position.y = 0.0;
      pose.position.z = 0.1;
      pose.orientation.w = 1.0;
      object.pose = pose;
    
      moveit::planning_interface::PlanningSceneInterface psi;
      psi.applyCollisionObject(object);
      
      RCLCPP_INFO(LOGGER, "Added cylinder to planning scene at (0.4, 0.0, 0.1)");
    }
    
    void MTCPickPlaceNode::doTask()
    {
      task_ = createTask();
      
      try
      {
        task_.init();
      }
      catch (mtc::InitStageException& e)
      {
        RCLCPP_ERROR_STREAM(LOGGER, e);
        return;
      }
      
      if (!task_.plan(5))
      {
        RCLCPP_ERROR_STREAM(LOGGER, "Task planning failed");
        return;
      }
      
      // Publish solution for visualization
      task_.introspection().publishSolution(*task_.solutions().front());
      
      RCLCPP_INFO(LOGGER, "Task planning succeeded, executing...");
      
      auto result = task_.execute(*task_.solutions().front());
      if (result.val != moveit_msgs::msg::MoveItErrorCodes::SUCCESS)
      {
        RCLCPP_ERROR_STREAM(LOGGER, "Task execution failed");
        return;
      }
      
      RCLCPP_INFO(LOGGER, "Task execution succeeded!");
    }
    
    mtc::Task MTCPickPlaceNode::createTask()
    {
      mtc::Task task;
      task.stages()->setName("pick place task");
      task.loadRobotModel(node_);
      
      const auto& arm_group = "panda_arm";
      const auto& hand_group = "hand";
      const auto& hand_frame = "panda_hand";
      
      // Set task properties
      task.setProperty("group", arm_group);
      task.setProperty("eef", hand_group);
      task.setProperty("ik_frame", hand_frame);
      
      // Create solvers
      auto sampling_planner = std::make_shared<mtc::solvers::PipelinePlanner>(node_);
      sampling_planner->setProperty("goal_joint_tolerance", 1e-5);
      
      auto cartesian_planner = std::make_shared<mtc::solvers::CartesianPath>();
      cartesian_planner->setMaxVelocityScalingFactor(0.5);
      cartesian_planner->setMaxAccelerationScalingFactor(0.5);
      cartesian_planner->setStepSize(0.005);
      
      // Stage 1: Current state
      auto stage_current = std::make_unique<mtc::stages::CurrentState>("current");
      task.add(std::move(stage_current));
      
      // Stage 2: Open hand
      auto stage_open_hand = std::make_unique<mtc::stages::MoveTo>("open hand", sampling_planner);
      stage_open_hand->setGroup(hand_group);
      stage_open_hand->setGoal("open");
      task.add(std::move(stage_open_hand));
      
      // Stage 3: Move to pick approach position
      auto stage_move_to_pick = std::make_unique<mtc::stages::Connect>(
          "move to pick",
          mtc::stages::Connect::GroupPlannerVector({ { arm_group, sampling_planner } }));
      stage_move_to_pick->setTimeout(5.0);
      task.add(std::move(stage_move_to_pick));
      
      // Stage 4: Approach object
      auto stage_approach_object = std::make_unique<mtc::stages::MoveRelative>(
          "approach object", cartesian_planner);
      stage_approach_object->setGroup(arm_group);
      stage_approach_object->setIKFrame(hand_frame);
      stage_approach_object->setMinMaxDistance(0.1, 0.15);
      
      geometry_msgs::msg::Vector3Stamped approach_direction;
      approach_direction.header.frame_id = "world";
      approach_direction.vector.z = -1.0;  // Move down
      stage_approach_object->setDirection(approach_direction);
      task.add(std::move(stage_approach_object));
      
      // Stage 5: Generate grasp pose
      auto stage_grasp = std::make_unique<mtc::stages::GenerateGraspPose>("generate grasp");
      stage_grasp->setAngle(M_PI / 6);
      stage_grasp->setPreGraspPose("open");
      stage_grasp->setGraspPose("closed");
      stage_grasp->setMonitoredStage(task.stages()->back());  // use approach as reference
      
      // Stage 6: Attach object
      auto stage_attach = std::make_unique<mtc::stages::ModifyPlanningScene>("attach object");
      stage_attach->attachObject("cylinder", hand_frame);
      task.add(std::move(stage_attach));
      
      // Stage 7: Lift object
      auto stage_lift = std::make_unique<mtc::stages::MoveRelative>("lift object", cartesian_planner);
      stage_lift->setGroup(arm_group);
      stage_lift->setIKFrame(hand_frame);
      stage_lift->setMinMaxDistance(0.1, 0.3);
      
      geometry_msgs::msg::Vector3Stamped lift_direction;
      lift_direction.header.frame_id = "world";
      lift_direction.vector.z = 1.0;  // Move up
      stage_lift->setDirection(lift_direction);
      task.add(std::move(stage_lift));
      
      return task;
    }
    
    int main(int argc, char** argv)
    {
      rclcpp::init(argc, argv);
      
      rclcpp::NodeOptions options;
      options.automatically_declare_parameters_from_overrides(true);
      
      auto mtc_node = std::make_unique<MTCPickPlaceNode>(options);
      
      // Setup planning scene with object
      mtc_node->setupPlanningScene();
      
      // Execute pick and place task
      mtc_node->doTask();
      
      rclcpp::shutdown();
      return 0;
    }

3.4 CMakeLists.txt for MTC Node:

    cmake_minimum_required(VERSION 3.8)
    project(mtc_tutorial)
    
    if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
      add_compile_options(-Wall -Wextra -Wpedantic)
    endif()
    
    # find dependencies
    find_package(ament_cmake REQUIRED)
    find_package(rclcpp REQUIRED)
    find_package(rclcpp_action REQUIRED)
    find_package(moveit_ros_planning_interface REQUIRED)
    find_package(moveit_task_constructor_core REQUIRED)
    find_package(moveit_visual_tools REQUIRED)
    find_package(geometric_shapes REQUIRED)
    find_package(tf2 REQUIRED)
    find_package(tf2_eigen REQUIRED)
    find_package(tf2_geometry_msgs REQUIRED)
    
    # Create library for MTC utilities (optional)
    add_library(mtc_utils SHARED src/mtc_utils.cpp)
    target_include_directories(mtc_utils PUBLIC
      $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
      $<INSTALL_INTERFACE:include>
    )
    target_link_libraries(mtc_utils PUBLIC
      rclcpp::rclcpp
      moveit_task_constructor_core::moveit_task_constructor_core
    )
    
    # Main executable
    add_executable(mtc_pick_place src/mtc_pick_place_node.cpp)
    target_link_libraries(mtc_pick_place PUBLIC
      rclcpp::rclcpp
      moveit_task_constructor_core::moveit_task_constructor_core
      moveit_ros_planning_interface::moveit_ros_planning_interface
      ${tf2_eigen_LIBRARIES}
    )
    
    ament_target_dependencies(mtc_pick_place
      rclcpp
      moveit_task_constructor_core
      tf2
      tf2_eigen
      tf2_geometry_msgs
    )
    
    install(TARGETS mtc_pick_place mtc_utils
      DESTINATION lib/${PROJECT_NAME}
    )
    
    if(BUILD_TESTING)
      find_package(ament_lint_auto REQUIRED)
      ament_lint_auto_find_test_dependencies()
    endif()
    
    ament_package()

Exercise 4: Integration with Gazebo Simulation 

4.1 Launch UR Robot in Gazebo with MoveIt:

    # Terminal 1: Launch UR10e in Gazebo
    ros2 launch ur_simulation_gz ur_sim_control.launch.py ur_type:=ur10e
    
    # Terminal 2: Launch MoveIt with Gazebo integration
    ros2 launch ur_simulation_gz ur_sim_moveit.launch.py ur_type:=ur10e
    
    # Alternative: Panda in Gazebo with MoveIt
    ros2 launch panda_moveit_config gazebo.launch.py
4.2 Custom MoveIt + Gazebo Launch File:

Create launch/gazebo_moveit.launch.py:

    from launch import LaunchDescription
    from launch.actions import IncludeLaunchDescription, TimerAction
    from launch.launch_description_sources import PythonLaunchDescriptionSource
    from launch_ros.actions import Node
    from launch_ros.substitutions import FindPackageShare
    
    def generate_launch_description():
        return LaunchDescription([
            # Start Gazebo with robot
            IncludeLaunchDescription(
                PythonLaunchDescriptionSource([
                    FindPackageShare('gazebo_simulation'),
                    '/launch/spawn_robot.launch.py'
                ]),
                launch_arguments={
                    'world': 'empty.sdf',
                    'use_sim_time': 'True'
                }.items()
            ),
            
            # Start robot state publisher
            TimerAction(
                period=5.0,
                actions=[
                    Node(
                        package='robot_state_publisher',
                        executable='robot_state_publisher',
                        name='robot_state_publisher',
                        parameters=[{'use_sim_time': True,
                                     'robot_description': '$(find robot_manipulation)/urdf/manipulator.urdf'}]
                    )
                ]
            ),
            
            # Start MoveIt 2
            TimerAction(
                period=8.0,
                actions=[
                    IncludeLaunchDescription(
                        PythonLaunchDescriptionSource([
                            FindPackageShare('robot_manipulation'),
                            '/launch/moveit_planning.launch.py'
                        ]),
                        launch_arguments={
                            'use_sim_time': 'True'
                        }.items()
                    )
                ]
            ),
            
            # Start RViz with MoveIt plugins
            TimerAction(
                period=12.0,
                actions=[
                    Node(
                        package='rviz2',
                        executable='rviz2',
                        name='rviz2',
                        arguments=['-d', '$(find robot_manipulation)/config/moveit_planning.rviz'],
                        parameters=[{'use_sim_time': True}]
                    )
                ]
            )
        ])
