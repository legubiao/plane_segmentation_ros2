cmake_minimum_required(VERSION 3.14)
project(convex_plane_decomposition_ros)
set(CMAKE_BUILD_TYPE Release)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")

# Catkin dependencies
set(dependencies
        rclcpp
        grid_map_ros
        convex_plane_decomposition
        convex_plane_decomposition_msgs
        tf2_ros
        visualization_msgs
)

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(grid_map_ros REQUIRED)
find_package(convex_plane_decomposition REQUIRED)
find_package(convex_plane_decomposition_msgs REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(visualization_msgs REQUIRED)

# Cpp standard version
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

###########
## Build ##
###########
add_library(${PROJECT_NAME}
        src/ConvexPlaneDecompositionRos.cpp
        src/MessageConversion.cpp
        src/ParameterLoading.cpp
        src/RosVisualizations.cpp
)
target_include_directories(${PROJECT_NAME}
        PUBLIC
        ${grid_map_ros_INCLUDE_DIRS}
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
        "$<INSTALL_INTERFACE:include/${PROJECT_NAME}>")
target_compile_definitions(${PROJECT_NAME} PUBLIC
        ${grid_map_ros_DEFINITIONS}
)
target_link_libraries(${PROJECT_NAME} PUBLIC
        ${rclcpp_TARGETS}
        ${grid_map_ros_LIBRARIES}
        ${grid_map_ros_TARGETS}
        ${convex_plane_decomposition_TARGETS}
        ${convex_plane_decomposition_msgs_TARGETS}
        ${tf2_ros_TARGETS}
        ${visualization_msgs_TARGETS}
)
target_link_options(${PROJECT_NAME} PUBLIC
        ${grid_map_ros_LINK_FLAGS}
)


add_executable(${PROJECT_NAME}_node src/ConvexPlaneDecompositionNode.cpp)
target_link_libraries(${PROJECT_NAME}_node ${PROJECT_NAME})

add_executable(${PROJECT_NAME}_add_noise src/noiseNode.cpp)
target_link_libraries(${PROJECT_NAME}_add_noise ${PROJECT_NAME})

add_executable(${PROJECT_NAME}_save_elevationmap src/SaveElevationMapAsImageNode.cpp)
target_link_libraries(${PROJECT_NAME}_save_elevationmap ${PROJECT_NAME})

add_executable(${PROJECT_NAME}_approximation_demo_node src/ConvexApproximationDemoNode.cpp)
target_link_libraries(${PROJECT_NAME}_approximation_demo_node ${PROJECT_NAME})

#############
## Install ##
#############

install(
        TARGETS ${PROJECT_NAME}
        EXPORT export_${PROJECT_NAME}
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include/${PROJECT_NAME}
)
install(
        DIRECTORY include/
        DESTINATION include/${PROJECT_NAME}
)
install(
        DIRECTORY config data launch rviz
        DESTINATION share/${PROJECT_NAME}
)
ament_export_dependencies(${dependencies})
ament_export_targets(export_${PROJECT_NAME} HAS_LIBRARY_TARGET)

#############
## Testing ##
#############
add_executable(${PROJECT_NAME}_TestShapeGrowing
        test/TestShapeGrowing.cpp
)
target_link_libraries(${PROJECT_NAME}_TestShapeGrowing
        ${PROJECT_NAME}
)

install(TARGETS
        ${PROJECT_NAME}_node
        ${PROJECT_NAME}_add_noise
        ${PROJECT_NAME}_save_elevationmap
        ${PROJECT_NAME}_approximation_demo_node
        ${PROJECT_NAME}_TestShapeGrowing
        DESTINATION lib/${PROJECT_NAME}
)

ament_package()