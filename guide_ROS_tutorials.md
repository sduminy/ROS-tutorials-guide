---
author:
- 'Segolene Duminy, Northeastern University'
date: '03/05/2020'
title: 'Guide for ROS Tutorials using ROSbot 2.0'
---

Preamble {#preamble .unnumbered}
========

The purpose of this guide is to help ROS beginners succeed the ROS
tutorials made by husarion, available
[here](https://husarion.com/tutorials/). I will tackle only the
technical issues, since the husarion tutorials explain the theory pretty
well.\
This guide may not be perfect, I was myself a ROS beginner 2 weeks ago.
In case you encounter some difficulties, please ask someome qualified
from your team.\
Good luck !

[ROS introduction](https://husarion.com/tutorials/ros-tutorials/1-ros-introduction/)
====================================================================================

Connecting to ROSbot
--------------------

The easiest way to programm your ROSbot 2.0 is to connect him to the
same network as your computer, which will be the `master`. To configure
the ROS connection between your ROSbot and `master`, you will have to
set the IP addresses on the ` /.bashrc` file. After connecting to the
ROSbot : , open the ` /.bashrc` file and add the following lines : You
should now be able to ping `master` and `husarion`.

Starting the system with `roslaunch` - Example
----------------------------------------------

Since the `master` has the screen and husarion programms the ROSbot, we
will have to split each launch file so that the nodes with screen output
(or later the ones, which require more computational power) are launched
from `master` and the others are launched in `husarion`. The
communication between the `master` and `husarion` is ensured by ROS if
the configuration was done properly. In `workspace/src`, create a
`catkin_pkg` named `tutorial_pkg` with the following command : You will
have to build the package with the command in your catkin workspace.
After that, you can create a folder `launch` inside of `tutorial_pkg` in
`master` and in `husarion`. Here, you will gather all the launch files
for the different tutorials. For the first tutorial, you can copy paste
the following :

    <launch>

        <arg name="use_gazebo" default="false"/>

        <include unless="$(arg use_gazebo)" file="$(find astra_launch)/launch/astra.launch"/>
        <include if="$(arg use_gazebo)" file="$(find rosbot_description)/launch/rosbot.launch"/>
        
    </launch>

    <launch>

        <node pkg="image_view" type="image_view" name="image_view">
            <remap from="/image" to="/camera/rgb/image_raw"/>
        </node>

    </launch>

To test the first tutorial, run with `master` and in another terminals
launch with `master` and with `husarion`.

[Creating nodes](https://husarion.com/tutorials/ros-tutorials/2-creating-nodes/)
================================================================================

Running your node
-----------------

On `master`, create a `cpp` file in `tutorial_pkg/src/`.

    #include <ros/ros.h>

    int main(int argc, char **argv)
    {
        ros::init(argc, argv, "example_node");
        ros::NodeHandle n("~");
        ros::Rate loop_rate(50);
        while (ros::ok())
        {
            ros::spinOnce();
            loop_rate.sleep();
        }
    }

You should modify the `CMakeLists.txt`, which should look like this :

    cmake_minimum_required(VERSION 2.8.3)
    project(tutorial_pkg)

    add_compile_options(-std=c++11)

    find_package(catkin REQUIRED COMPONENTS
      roscpp
    )

    catkin_package(
      CATKIN_DEPENDS
    )

    include_directories(${catkin_INCLUDE_DIRS})

    add_executable(${PROJECT_NAME}_node src/tutorial_pkg_node.cpp)

    target_link_libraries(${PROJECT_NAME}_node
      ${catkin_LIBRARIES}
    )

You should now build your project with the command in your `workspace`.
You will create a launch file in the `launch` folder of `master`.

    <launch>

        <node pkg="tutorial_pkg" type="tutorial_pkg_node" name="tutorial_pkg_node" output="screen">
        </node>

    </launch>

You can launch it on `master` with the command

Subscribing to topic
--------------------

You will modify your previous code, so it will look like this :

    #include <ros/ros.h>
    #include <sensor_msgs/Image.h>

    void imageCallback(const sensor_msgs::ImageConstPtr &image)
    {
       long long sum = 0;
       for (int value : image->data)
       {
          sum += value;
       }
       int avg = sum / image->data.size();
       std::cout << "Brightness: " << avg << std::endl;
    }

    int main(int argc, char **argv)
    {
       ros::init(argc, argv, "example_node");
       ros::NodeHandle n("~");
       ros::Subscriber sub = n.subscribe("/camera/rgb/image_raw", 10, imageCallback);
       ros::Rate loop_rate(50);
       while (ros::ok())
       {
          ros::spinOnce();
          loop_rate.sleep();
       }
    }

Along with this new code, you can launch your node with the following
launch files on `husarion` and on `master` :

    <launch>

        <arg name="use_gazebo" default="false"/>

        <include unless="$(arg use_gazebo)" file="$(find astra_launch)/launch/astra.launch"/>
        <include if="$(arg use_gazebo)" file="$(find rosbot_description)/launch/rosbot.launch"/>

    </launch>

    <launch>

        <include file="$(find tutorial_pkg)/launch/tutorial_pkg_node.launch"/>
        
    </launch>

You can launch it with the following commands on `husarion` and on
`master` .

Receiving parameters
--------------------

Again, you will have to change you cpp file as :

    #include <ros/ros.h>
    #include <sensor_msgs/Image.h>

    bool print_b;

    void imageCallback(const sensor_msgs::ImageConstPtr &image)
    {
       long long sum = 0;
       for (int value : image->data)
       {
          sum += value;
       }
       int avg = sum / image->data.size();
       if (print_b)
       {
          std::cout << "Brightness: " << avg << std::endl;
       }
    }

    int main(int argc, char **argv)
    {
       ros::init(argc, argv, "example_node");
       ros::NodeHandle n("~");
       ros::Subscriber sub = n.subscribe("/camera/rgb/image_raw", 10, imageCallback);
       n.param<bool>("print_brightness", print_b, false);
       ros::Rate loop_rate(50);
       while (ros::ok())
       {
          ros::spinOnce();
          loop_rate.sleep();
       }
    }

Accordingly, you will change the `tutorial_pkg_node.launch` file as :

    <launch>

        <node pkg="tutorial_pkg" type="tutorial_pkg_node" name="tutorial_pkg_node" output="screen">
            <param name="print_brightness" value="true" />
        </node>

    </launch>

You can test it by running the 2 previous launch files and try modifying
the `print_brightness` parameter from “true” to “false”.

Publishing to topic
-------------------

Once more, you should modify your cpp code as follows :

    #include <ros/ros.h>
    #include <sensor_msgs/Image.h>
    #include <std_msgs/UInt8.h>

    bool print_b;
    ros::Publisher brightness_pub;

    void imageCallback(const sensor_msgs::ImageConstPtr &image)
    {
       long long sum = 0;
       for (int value : image->data)
       {
          sum += value;
       }
       int avg = sum / image->data.size();
       if (print_b)
       {
          std::cout << "Brightness: " << avg << std::endl;
       }
       std_msgs::UInt8 brightness_value;
       brightness_value.data = avg;
       brightness_pub.publish(brightness_value);
    }

    int main(int argc, char **argv)
    {
       ros::init(argc, argv, "example_node");
       ros::NodeHandle n("~");
       ros::Subscriber sub = n.subscribe("/camera/rgb/image_raw", 10, imageCallback);
       n.param<bool>("print_brightness", print_b, false);
       brightness_pub = n.advertise<std_msgs::UInt8>("brightness", 1);
       ros::Rate loop_rate(50);
       while (ros::ok())
       {
          ros::spinOnce();
          loop_rate.sleep();
       }
    }

You can run the command to see the published topic.

Calling the service
-------------------

Once again, you have to modify the cpp code :

    #include <ros/ros.h>
    #include <sensor_msgs/Image.h>
    #include <std_msgs/UInt8.h>
    #include <std_srvs/Empty.h>

    bool print_b;
    ros::Publisher brightness_pub;
    int frames_passed = 0;

    void imageCallback(const sensor_msgs::ImageConstPtr &image)
    {
       long long sum = 0;
       for (int value : image->data)
       {
          sum += value;
       }
       int avg = sum / image->data.size();
       if (print_b)
       {
          std::cout << "Brightness: " << avg << std::endl;
       }
       std_msgs::UInt8 brightness_value;
       brightness_value.data = avg;
       brightness_pub.publish(brightness_value);
       frames_passed++;
    }

    int main(int argc, char **argv)
    {
       ros::init(argc, argv, "example_node");
       ros::NodeHandle n("~");
       ros::Subscriber sub = n.subscribe("/camera/rgb/image_raw", 10, imageCallback);
       n.param<bool>("print_brightness", print_b, false);
       brightness_pub = n.advertise<std_msgs::UInt8>("brightness", 1);
       ros::ServiceClient client = n.serviceClient<std_srvs::Empty>("/image_saver/save");
       std_srvs::Empty srv;
       ros::Rate loop_rate(50);
       while (ros::ok())
       {
          ros::spinOnce();
          if (frames_passed > 100)
          {
             frames_passed = 0;
             client.call(srv);
          }
          loop_rate.sleep();
       }
    }

You have to create another launch file, which will run the `image_saver`
node.

    <launch>

        <node pkg="image_view" type="image_saver" name="image_saver">
            <param name="save_all_image" value="false" />
            <param name="filename_format" value="$(env HOME)/ros_workspace/image%04d.%s"/>
            <remap from="/image" to="/camera/rgb/image_raw"/>
        </node>

    </launch>

You should then modify the launch file from `master` :

    <launch>

        <include file="$(find tutorial_pkg)/launch/tutorial_pkg_node.launch"/>
        <include file="$(find tutorial_pkg)/launch/image_saver.launch"/>
        
    </launch>

You can delete the saved images by running

Providing a service
-------------------

Last but not least, you can modify your cpp code as :

    #include <ros/ros.h>
    #include <sensor_msgs/Image.h>
    #include <std_msgs/UInt8.h>
    #include <std_srvs/Empty.h>
    #include <std_srvs/Trigger.h>

    bool print_b;
    ros::Publisher brightness_pub;
    int frames_passed = 0;
    int saved_imgs = 0;

    void imageCallback(const sensor_msgs::ImageConstPtr &image)
    {
       long long sum = 0;
       for (int value : image->data)
       {
          sum += value;
       }
       int avg = sum / image->data.size();
       if (print_b)
       {
          std::cout << "Brightness: " << avg << std::endl;
       }
       std_msgs::UInt8 brightness_value;
       brightness_value.data = avg;
       brightness_pub.publish(brightness_value);
       frames_passed++;
    }

    bool saved_img(std_srvs::Trigger::Request &req, std_srvs::Trigger::Response &res)
    {
       res.success = 1;
       std::string str("Saved images: ");
       std::string num = std::to_string(saved_imgs);
       str.append(num);
       res.message = str;
       return true;
    }

    int main(int argc, char **argv)
    {
       ros::init(argc, argv, "example_node");
       ros::NodeHandle n("~");
       ros::Subscriber sub = n.subscribe("/camera/rgb/image_raw", 10, imageCallback);
       n.param<bool>("print_brightness", print_b, false);
       brightness_pub = n.advertise<std_msgs::UInt8>("brightness", 1);
       ros::ServiceClient client = n.serviceClient<std_srvs::Empty>("/image_saver/save");
       std_srvs::Empty srv;
       ros::ServiceServer service = n.advertiseService("saved_images", saved_img);
       ros::Rate loop_rate(50);
       while (ros::ok())
       {
          ros::spinOnce();
          if (frames_passed > 100)
          {
             frames_passed = 0;
             client.call(srv);
             saved_imgs++;
          }
          loop_rate.sleep();
       }
    }

After launching the `tutorial_2_` files, you can call the service by
running the command : .

[Simple kinematics for mobile robot](https://husarion.com/tutorials/ros-tutorials/3-simple-kinematics-for-mobile-robot/)
========================================================================================================================

You will need the following launch files in order to control your ROSbot
2.0 with your keyboard (do not forget to run beforehand on `master`).

    <launch>

        <arg name="use_rosbot" default="true"/>
        <arg name="use_gazebo" default="false"/>

        <include if="$(arg use_gazebo)" file="$(find rosbot_gazebo)/launch/rosbot.launch"/>    

        <include if="$(arg use_rosbot)" file="$(find rosbot_ekf)/launch/all.launch"/>

    </launch>

    <launch>

        <node name="teleop_twist_keyboard" pkg="teleop_twist_keyboard" type="teleop_twist_keyboard.py" output="screen"/>

    </launch>

To test the kinematics controller, run :

-   -   -   

You might want to reduce the linear speed using the key ’x’ for safety.

[Visual Object Recognition](https://husarion.com/tutorials/ros-tutorials/4-visual-object-recognition/)
======================================================================================================

Teaching objects
----------------

We will use the package `find-object-2d` so we first need to install it
on `master` with this command : . Then you will need to create a folder
where you can store the learnt objects : for example. You can now create
the launch files on `husarion` and `master` :

    <launch>

        <arg name="use_rosbot" default="false"/>
        <arg name="use_gazebo" default="true"/>

        <arg name="teach" default="true"/>
        <arg name="recognize" default="false"/>

        <arg if="$(arg teach)" name="chosen_world" value="rosbot_world_teaching"/>
        <arg if="$(arg recognize)" name="chosen_world" value="rosbot_world_recognition"/>

        <include if="$(arg use_rosbot)" file="$(find astra_launch)/launch/astra.launch"/>
        
        <include if="$(arg use_rosbot)" file="$(find rosbot_ekf)/launch/all.launch"/>

        <include if="$(arg use_gazebo)" file="$(find rosbot_gazebo)/launch/$(arg chosen_world).launch"/>
        <include if="$(arg use_gazebo)" file="$(find rosbot_gazebo)/launch/rosbot.launch"/>

    </launch>

    <launch>

        <arg name="teach" default="true"/>
        <arg name="recognize" default="false"/>

        <node name="teleop_twist_keyboard" pkg="teleop_twist_keyboard" type="teleop_twist_keyboard.py" output="screen"/>

        <node pkg="find_object_2d" type="find_object_2d" name="find_object_2d">
            <remap from="image" to="/camera/rgb/image_raw"/>
            <param name="gui" value="$(arg teach)"/>
            <param if="$(arg recognize)" name="objects_path" value="~/Documents/objects"/>
        </node>

    </launch>

After launching the different nodes, you can follow the steps of the
husarion tutorial.

Making decision with recognized object
--------------------------------------

You will have to create a new cpp file and place it in on `master`.

    #include <ros/ros.h>
    #include <std_msgs/Float32MultiArray.h>
    #include <geometry_msgs/Twist.h>

    #define SMILE 14
    #define ARROW_RIGHT 11
    #define ARROW_LEFT 12
    #define ARROW_UP 13

    int id = 0;
    ros::Publisher action_pub;
    geometry_msgs::Twist set_vel;

    void objectCallback(const std_msgs::Float32MultiArrayPtr &object)
    {
       if (object->data.size() > 0)
       {
          id = object->data[0];

          switch (id)
          {
          case ARROW_LEFT:
             set_vel.linear.x = 0;
             set_vel.angular.z = 1;
             break;
          case ARROW_UP:
             set_vel.linear.x = 1;
             set_vel.angular.z = 0;
             break;
          case ARROW_RIGHT:
             set_vel.linear.x = 0;
             set_vel.angular.z = -1;
             break;
          default: // other object
             set_vel.linear.x = 0;
             set_vel.angular.z = 0;
          }
          action_pub.publish(set_vel);
       }
       else
       {
          // No object detected
          set_vel.linear.x = 0;
          set_vel.angular.z = 0;
          action_pub.publish(set_vel);
       }
    }

    int main(int argc, char **argv)
    {

       ros::init(argc, argv, "action_controller");
       ros::NodeHandle n("~");
       ros::Rate loop_rate(50);
       ros::Subscriber sub = n.subscribe("/objects", 1, objectCallback);
       action_pub = n.advertise<geometry_msgs::Twist>("/cmd_vel", 1);
       set_vel.linear.x = 0;
       set_vel.linear.y = 0;
       set_vel.linear.z = 0;
       set_vel.angular.x = 0;
       set_vel.angular.y = 0;
       set_vel.angular.z = 0;
       while (ros::ok())
       {
          ros::spinOnce();
          loop_rate.sleep();
       }
    }

Of course, you have to adapt the code regarding with the object your
ROSbot 2.0 has learned and their ID. Do not forget to modify the
`CMakeLists.txt` to allow execution of your cpp file and build your
project when it is done. To test your programm, you just have to add the
`action_controller_node` in the launch file on `master`, so that it
looks like this :

    <launch>

        <arg name="teach" default="true"/>
        <arg name="recognize" default="false"/>

        <node name="teleop_twist_keyboard" pkg="teleop_twist_keyboard" type="teleop_twist_keyboard.py" output="screen"/>

        <node pkg="find_object_2d" type="find_object_2d" name="find_object_2d">
            <remap from="image" to="/camera/rgb/image_raw"/>
            <param name="gui" value="$(arg teach)"/>
            <param if="$(arg recognize)" name="objects_path" value="~/Documents/objects"/>
        </node>
        
        <node pkg="tutorial_pkg" type="action_controller_node" name="action_controller" output="screen"/>

    </launch>

Now, you can test that the ROSbot 2.0 does has expected by your
controller.

Following the object
--------------------

There is no difficulty here, just follow the husarion tutorial.

[Connecting the ROSbot to multiple machines](https://husarion.com/tutorials/ros-tutorials/5-running-ros-on-multiple-machines/)
==============================================================================================================================

See the husarion tutorial.

[SLAM navigation](https://husarion.com/tutorials/ros-tutorials/6-slam-navigation/)
==================================================================================

This is where having a master different from the ROSbot 2.0 take an
interest. Because SLAM navigation requires a lot of computational power,
we will need to run it on `master`.

Running the laser scanner
-------------------------

Let’s create a `drive_controller.cpp` file :

    #include <ros/ros.h>
    #include <geometry_msgs/PoseStamped.h>
    #include <tf/transform_broadcaster.h>

    tf::Transform transform;
    tf::Quaternion q;

    void pose_callback(const geometry_msgs::PoseStampedPtr &pose)
    {
       static tf::TransformBroadcaster br;
       q.setX(pose->pose.orientation.x);
       q.setY(pose->pose.orientation.y);
       q.setZ(pose->pose.orientation.z);
       q.setW(pose->pose.orientation.w);

       transform.setOrigin(tf::Vector3(pose->pose.position.x, pose->pose.position.y, 0.0));
       transform.setRotation(q);

       br.sendTransform(tf::StampedTransform(transform, ros::Time::now(), "odom", "base_link"));
    }

    int main(int argc, char **argv)
    {
       ros::init(argc, argv, "drive_controller");
       ros::NodeHandle n("~");
       ros::Subscriber pose_sub = n.subscribe("/pose", 1, pose_callback);
       ros::Rate loop_rate(100);
       while (ros::ok())
       {
          ros::spinOnce();
          loop_rate.sleep();
       }
    }

Do not forget to modify the `CMakeLists.txt` accordingly. We can now
create the launch files, which will run the SLAM algorithm.

    <launch>

        <arg name="use_rosbot" default="true"/>
        <arg name="use_gazebo" default="false"/>
        
        <include if="$(arg use_gazebo)" file="$(find rosbot_gazebo)/launch/maze_world.launch"/>
        <include if="$(arg use_gazebo)" file="$(find rosbot_gazebo)/launch/rosbot_world.launch"/>

        <include if="$(arg use_rosbot)" file="$(find rosbot_ekf)/launch/all.launch" /> 
        
        <node if="$(arg use_rosbot)" pkg="rplidar_ros" type="rplidarNode" name="rplidar" />
        
        <node if="$(arg use_rosbot)" pkg="tf" type="static_transform_publisher" name="laser_broadcaster" args="0 0 0 3.14 0 0 base_link laser_frame 100" />

    </launch>

    <launch>
        
        <node pkg="tutorial_pkg" type="drive_controller_node" name="drive_controller"/>

        <node pkg="teleop_twist_keyboard" type="teleop_twist_keyboard.py" name="teleop_twist_keyboard" output="screen"/>

        <node pkg="rviz" type="rviz" name="rviz"/>
        
    </launch>

Then, follow the indications of the husarion tutorial.

Navigation and map building
---------------------------

Just add the `gmapping` package to the `master` launch file :

    <launch>
        
        <node pkg="tutorial_pkg" type="drive_controller_node" name="drive_controller"/>

        <node pkg="teleop_twist_keyboard" type="teleop_twist_keyboard.py" name="teleop_twist_keyboard" output="screen"/>

        <node pkg="rviz" type="rviz" name="rviz"/>
        
        <node pkg="gmapping" type="slam_gmapping" name="gmapping">
            <param name="base_frame" value="base_link"/>
            <param name="odom_frame" value="odom" />
            <param name="delta" value="0.1" />
        </node>

    </launch>

And follow the indications.

[Path Planning](https://husarion.com/tutorials/ros-tutorials/7-path-planning/)
==============================================================================

Follow the instructions to create the different configurations files on
`master`. You can use the following launch files to test the path
planner.

    <launch>

        <arg name="use_rosbot" default="true"/>
        <arg name="use_gazebo" default="false"/>

        <!-- Gazebo -->
        <include if="$(arg use_gazebo)" file="$(find rosbot_gazebo)/launch/maze_world.launch"/>
        <include if="$(arg use_gazebo)" file="$(find rosbot_gazebo)/launch/rosbot.launch"/>
        <param if="$(arg use_gazebo)" name="use_sim_time" value="true"/>

        <!-- ROSbot 2.0 -->
        <node if="$(arg use_rosbot)" pkg="rplidar_ros" type="rplidarNode" name="rplidar">
            <param name="angle_compensate" type="bool" value="true"/>
            <param name="serial_baudrate" type="int" value="115200"/><!--model A2 (ROSbot 2.0) -->
        </node>

        <include if="$(arg use_rosbot)" file="$(find rosbot_ekf)/launch/all.launch"/>

        <node if="$(arg use_rosbot)" pkg="tf" type="static_transform_publisher" name="laser_broadcaster" args="0 0 0 3.14 0 0 base_link laser_frame 100" />

    </launch>

    <launch>

        <node pkg="gmapping" type="slam_gmapping" name="gmapping">
            <param name="base_frame" value="base_link"/>
            <param name="odom_frame" value="odom" />
            <param name="delta" value="0.1" />
        </node>

        <node pkg="move_base" type="move_base" name="move_base" output="screen">
            <param name="controller_frequency" value="10.0"/>
            <rosparam file="$(find tutorial_pkg)/config/costmap_common_params.yaml" command="load" ns="global_costmap" />
            <rosparam file="$(find tutorial_pkg)/config/costmap_common_params.yaml" command="load" ns="local_costmap" />
            <rosparam file="$(find tutorial_pkg)/config/local_costmap_params.yaml" command="load" />
            <rosparam file="$(find tutorial_pkg)/config/global_costmap_params.yaml" command="load" />
            <rosparam file="$(find tutorial_pkg)/config/trajectory_planner.yaml" command="load" />
        </node>
        
        <node pkg="rviz" type="rviz" name="rviz" />

    </launch>

Then, just follow the instructions to set a target destination and see
how the ROSbot 2.0 is moving toward it !

[Unknown Environment Exploration](https://husarion.com/tutorials/ros-tutorials/8-unknown-environment-exploration/)
==================================================================================================================

For the exploration of unknown environment, we will use the
`explore_lite` package. Since it is distributed only on kinetic version,
we will use in on `husarion`. Create the configurations file
`exploration.yaml` on `husarion` as indicated on the husarion tutorial.
The launch file for `master` will stay the same as the previous
tutorial. You just need to add the node on the launch file on `husarion`
:

    <launch>

        <arg name="use_rosbot" default="true"/>
        <arg name="use_gazebo" default="false"/>

        <!-- Gazebo -->
        <include if="$(arg use_gazebo)" file="$(find rosbot_gazebo)/launch/maze_world.launch"/>
        <include if="$(arg use_gazebo)" file="$(find rosbot_gazebo)/launch/rosbot.launch"/>
        <param if="$(arg use_gazebo)" name="use_sim_time" value="true"/>

        <!-- ROSbot 2.0 -->
        <node if="$(arg use_rosbot)" pkg="rplidar_ros" type="rplidarNode" name="rplidar">
            <param name="angle_compensate" type="bool" value="true"/>
            <param name="serial_baudrate" type="int" value="115200"/><!--model A2 (ROSbot 2.0) -->
        </node>

        <include if="$(arg use_rosbot)" file="$(find rosbot_ekf)/launch/all.launch"/>

        <node if="$(arg use_rosbot)" pkg="tf" type="static_transform_publisher" name="laser_broadcaster" args="0 0 0 3.14 0 0 base_link laser_frame 100" />

        <node pkg="explore_lite" type="explore" respawn="false" name="explore" output="screen">
                <rosparam file="$(find tutorial_pkg)/config/exploration.yaml" command="load" />
        </node>

    </launch>
