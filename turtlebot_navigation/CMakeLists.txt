cmake_minimum_required(VERSION 2.8.3)
project(turtlebot_navigation)

# Load catkin and all dependencies required for this package
find_package(catkin REQUIRED COMPONENTS tf roscpp sensor_msgs)

# What other packages will need to use this package
catkin_package(
    CATKIN_DEPENDS tf roscpp sensor_msgs
)

catkin_add_env_hooks(25.turtlebot-navigation SHELLS sh DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/env-hooks)


###########
## Build ##
###########
include_directories(${catkin_INCLUDE_DIRS})

# Add_executables
add_executable(laser_footprint_filter src/laser_footprint_filter.cpp)
target_link_libraries(laser_footprint_filter ${catkin_LIBRARIES})

# Generate camera param files
include($ENV{ROS_ROOT}/core/rosbuild/rosbuild.cmake)
macro(generateCameraParamFiles camera)		
	if(NOT EXISTS ${PROJECT_SOURCE_DIR}/launch/includes/amcl/${camera}_amcl.launch.xml)
		add_custom_target(
			${camera}_amcl_target ALL
			# Generate the camera-specific amcl.launch.xml file
			COMMAND ln -s amcl.launch.xml ${camera}_amcl.launch.xml
			WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/launch/includes/amcl/		
		)
	endif()
	if(NOT EXISTS ${PROJECT_SOURCE_DIR}/launch/includes/gmapping/${camera}_gmapping.launch.xml)
		add_custom_target(
			${camera}_gmapping_target ALL
			# Generate the camera-specific gmapping.launch.xml file
			COMMAND ln -s gmapping.launch.xml ${camera}_gmapping.launch.xml
			WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/launch/includes/gmapping/
		)
	endif()
	if(NOT EXISTS ${PROJECT_SOURCE_DIR}/param/${camera}_costmap_params.yaml)
		add_custom_target(
			${camera}_move_base_costmap_target ALL
			# Generate the camera-specific move_base costmap params file
			COMMAND ln -s dummy.yaml ${camera}_costmap_params.yaml
			WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/param/
		)
	endif()
endmacro()
# use file() to get list of all camera file names from turtlebot_bringup
rosbuild_find_ros_package(turtlebot_bringup)
file(GLOB camera_file_names RELATIVE "${turtlebot_bringup_PACKAGE_PATH}/launch/includes/3dsensor/" "${turtlebot_bringup_PACKAGE_PATH}/launch/includes/3dsensor/*.launch.xml")
# foreach file name, use regex to strip out camera name, and add it to cameras list
foreach(camera_file_name ${camera_file_names})
	STRING(REGEX REPLACE ".launch.xml" "" camera_name ${camera_file_name})
	generateCameraParamFiles(${camera_name})	
endforeach()





#############
## Install ##
#############

# Mark executables and/or libraries for installation
install(TARGETS laser_footprint_filter
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

# Mark anything (useful) else for installation
install(DIRECTORY laser
        DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

install(DIRECTORY launch
        DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

install(DIRECTORY param
        DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

install(DIRECTORY maps
        DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
