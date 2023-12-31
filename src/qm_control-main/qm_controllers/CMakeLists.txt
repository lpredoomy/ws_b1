cmake_minimum_required(VERSION 3.10)
project(qm_controllers)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

## Find catkin macros and libraries
find_package(catkin REQUIRED
        COMPONENTS
        roscpp
        qm_common
        qm_interface
        qm_estimation
        qm_wbc
        controller_interface
        ocs2_legged_robot_ros
        ocs2_self_collision_visualization
        angles
        dynamic_reconfigure
        qm_msgs
        )

find_package(PkgConfig REQUIRED)
pkg_check_modules(pinocchio REQUIRED pinocchio)

generate_dynamic_reconfigure_options(
      cfg/weight.cfg
)    

catkin_package(
        INCLUDE_DIRS
        include
            ${EIGEN3_INCLUDE_DIRS}
        LIBRARIES
            ${PROJECT_NAME}
        CATKIN_DEPENDS
            roscpp
            qm_common
            qm_interface
            qm_estimation
            controller_interface
            ocs2_legged_robot_ros
            angles
            dynamic_reconfigure
            qm_msgs
            ocs2_legged_robot_ros
            ocs2_self_collision_visualization
        DEPENDS
            pinocchio
)

###########
## Build ##
###########

include_directories(
        include
        ${catkin_INCLUDE_DIRS}
        ${pinocchio_INCLUDE_DIRS}
        ${EIGEN3_INCLUDE_DIRS}
        ${Boost_INCLUDE_DIRS}
)

link_directories(
        ${pinocchio_LIBRARY_DIRS}
)

## Declare a cpp library
add_library(${PROJECT_NAME}
        src/QMController.cpp
        )
add_dependencies(${PROJECT_NAME}
        ${catkin_EXPORTED_TARGETS}
        ${PROJECT_NAME}_gencfg
        )
target_link_libraries(${PROJECT_NAME}
        ${catkin_LIBRARIES}
        ${pinocchio_LIBRARIES}
        )

target_compile_options(${PROJECT_NAME} PUBLIC ${OCS2_CXX_FLAGS})


###########
##  EXE  ##
###########
add_executable(qm_target_trajectories_publisher
     src/QmTargetTrajectoriesPublisher_node.cpp
     src/QmTargetTrajectoriesPublisher.cpp
     )
add_dependencies(qm_target_trajectories_publisher
     ${catkin_EXPORTED_TARGETS}
     )
target_link_libraries(qm_target_trajectories_publisher
     ${catkin_LIBRARIES}
     )
target_compile_options(qm_target_trajectories_publisher PRIVATE ${OCS2_CXX_FLAGS})

#########################
###   CLANG TOOLING   ###
#########################
find_package(cmake_clang_tools QUIET)
if (cmake_clang_tools_FOUND)
    message(STATUS "Run clang tooling for target " ${PROJECT_NAME})
    add_clang_tooling(
            TARGETS ${PROJECT_NAME} legged_target_trajectories_publisher
            SOURCE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/src ${CMAKE_CURRENT_SOURCE_DIR}/include
            CT_HEADER_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/include
            CF_WERROR
    )
endif (cmake_clang_tools_FOUND)

#############
## Install ##
#############

# Mark executables and/or libraries for installation
install(TARGETS ${PROJECT_NAME}
        ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
        LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
        RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
        )

# Mark cpp header files for installation
install(DIRECTORY include/${PROJECT_NAME}/
        DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
        FILES_MATCHING PATTERN "*.h"
        )

# Install plugins
install(FILES legged_controllers_plugins.xml
        DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
        )




