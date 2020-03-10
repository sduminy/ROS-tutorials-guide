cmake_minimum_required(VERSION 2.8.3)
project(tutorial_pkg)

## Compile as C++11, supported in ROS Kinetic and newer
add_compile_options(-std=c++11)

find_package(catkin REQUIRED COMPONENTS
  roscpp tf
)
find_package( OpenCV REQUIRED )

###########
## Build ##
###########

## Specify additional locations of header files
## Your package locations should be listed before other locations
include_directories(
  ${catkin_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
)

## Declare a C++ executable
## With catkin_make all packages are built within a single CMake context
## The recommended prefix ensures that target names across packages don't collide
add_executable(${PROJECT_NAME}_node src/tutorial_pkg_node.cpp)
add_executable(action_controller_node src/action_controller.cpp)
add_executable(drive_controller_node src/drive_controller.cpp)

## Specify libraries to link a library or executable target against
target_link_libraries(tutorial_pkg_node
   ${catkin_LIBRARIES}
)
target_link_libraries(action_controller_node
   ${catkin_LIBRARIES}
   ${OpenCV_LIBRARIES}
)
target_link_libraries(drive_controller_node
   ${catkin_LIBRARIES}
)
