cmake_minimum_required(VERSION 3.0.2)
project(lsd_slam_core)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${SSE_FLAGS} -std=c++11 -fopenmp")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS} -O3")

# set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
# set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)
# set(CMAKE_MODULE_PATH   ${PROJECT_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH})

set(Eigen3_DIR "/home/adi/rando/eigen/release/share/eigen3/cmake")
set(OpenCV_DIR "/home/adi/rando/opencv-2.4.8/release/share/OpenCV")
find_package(Eigen3 3.3.90 REQUIRED EXACT NO_DEFAULT_PATH)
find_package(OpenCV 2.4.8 REQUIRED EXACT NO_DEFAULT_PATH)
find_package(Boost REQUIRED)

# find_package(SuiteParse REQUIRED) # Apparently needed by g2o
find_package(X11 REQUIRED)

find_package(catkin REQUIRED COMPONENTS
  roscpp
  cv_bridge
  dynamic_reconfigure
  rosbag
  sensor_msgs
  std_msgs
  lsd_slam_viewer
  roslib
)
# FabMap
# uncomment this part to enable fabmap
#add_subdirectory(${PROJECT_SOURCE_DIR}/thirdparty/openFabMap)
#include_directories(${PROJECT_SOURCE_DIR}/thirdparty/openFabMap/include)
#add_definitions("-DHAVE_FABMAP")
#set(FABMAP_LIB openFABMAP )

# Dynamic Reconfigure Services
# rosbuild_find_ros_package(dynamic_reconfigure)
# include(${dynamic_reconfigure_PACKAGE_PATH}/cmake/cfgbuild.cmake)
# gencfg()

# include(${dynamic_reconfigure_PACKAGE_PATH}/cmake/cfgbuild.cmake)
generate_dynamic_reconfigure_options(
  cfg/LSDDebugParams.cfg
  cfg/LSDParams.cfg
)

# SSE flags
# rosbuild_check_for_sse()
# add_definitions("-DUSE_ROS")
# add_definitions("-DENABLE_SSE")

# Also add some useful compiler flag
# set(CMAKE_CXX_FLAGS
   # "${CMAKE_CXX_FLAGS} ${SSE_FLAGS} -march=native  -std=c++0x"
# )

# Set source files
set(lsd_SOURCE_FILES
  ${PROJECT_SOURCE_DIR}/src/DataStructures/Frame.cpp
  ${PROJECT_SOURCE_DIR}/src/DataStructures/FramePoseStruct.cpp
  ${PROJECT_SOURCE_DIR}/src/DataStructures/FrameMemory.cpp
  ${PROJECT_SOURCE_DIR}/src/SlamSystem.cpp
  ${PROJECT_SOURCE_DIR}/src/LiveSLAMWrapper.cpp
  ${PROJECT_SOURCE_DIR}/src/DepthEstimation/DepthMap.cpp
  ${PROJECT_SOURCE_DIR}/src/DepthEstimation/DepthMapPixelHypothesis.cpp
  ${PROJECT_SOURCE_DIR}/src/util/globalFuncs.cpp
  ${PROJECT_SOURCE_DIR}/src/util/SophusUtil.cpp
  ${PROJECT_SOURCE_DIR}/src/util/settings.cpp
  ${PROJECT_SOURCE_DIR}/src/util/Undistorter.cpp
  ${PROJECT_SOURCE_DIR}/src/Tracking/Sim3Tracker.cpp
  ${PROJECT_SOURCE_DIR}/src/Tracking/Relocalizer.cpp
  ${PROJECT_SOURCE_DIR}/src/Tracking/SE3Tracker.cpp
  ${PROJECT_SOURCE_DIR}/src/Tracking/TrackingReference.cpp
  ${PROJECT_SOURCE_DIR}/src/IOWrapper/Timestamp.cpp
  ${PROJECT_SOURCE_DIR}/src/GlobalMapping/FabMap.cpp
  ${PROJECT_SOURCE_DIR}/src/GlobalMapping/KeyFrameGraph.cpp
  ${PROJECT_SOURCE_DIR}/src/GlobalMapping/g2oTypeSim3Sophus.cpp
  ${PROJECT_SOURCE_DIR}/src/GlobalMapping/TrackableKeyFrameSearch.cpp
)
set(SOURCE_FILES
  ${lsd_SOURCE_FILES}
  ${PROJECT_SOURCE_DIR}/src/IOWrapper/ROS/ROSImageStreamThread.cpp
  ${PROJECT_SOURCE_DIR}/src/IOWrapper/ROS/ROSOutput3DWrapper.cpp
  ${PROJECT_SOURCE_DIR}/src/IOWrapper/OpenCV/ImageDisplay_OpenCV.cpp
)

include_directories(
  ${EIGEN3_INCLUDE_DIR}
  ${PROJECT_SOURCE_DIR}/src
  ${PROJECT_SOURCE_DIR}/thirdparty/Sophus
  ${CSPARSE_INCLUDE_DIR} #Has been set by SuiteParse
  ${CHOLMOD_INCLUDE_DIR} #Has been set by SuiteParse
  ${catkin_INCLUDE_DIRS}
  /home/adi/rando/g2o/release/include
  /usr/include/suitesparse
  cfg/cpp
)


# build shared library.
add_library(lsdslam SHARED ${SOURCE_FILES})
# target_link_libraries(lsdslam ${FABMAP_LIB} g2o_core g2o_stuff csparse cxsparse g2o_solver_csparse g2o_csparse_extension g2o_types_sim3 g2o_types_sba X11)
target_link_libraries(lsdslam ${catkin_LIBRARIES} ${FABMAP_LIB} /home/adi/rando/g2o/release/lib/libg2o_core.so /home/adi/rando/g2o/release/lib/libg2o_stuff.so /home/adi/rando/g2o/release/lib/libg2o_solver_csparse.so /home/adi/rando/g2o/release/lib/libg2o_solver_csparse.so /home/adi/rando/g2o/release/lib/libg2o_csparse_extension.so /home/adi/rando/g2o/release/lib/libg2o_types_sim3.so /home/adi/rando/g2o/release/lib/libg2o_types_sba.so X11 ${Boost_LIBRARES} boost_system ${OpenCV_LIBRARIES})
# rosbuild_link_boost(lsdslam thread)


# build live ros node
add_executable(live_slam src/main_live_odometry.cpp)
target_link_libraries(live_slam lsdslam)


# build image node
add_executable(dataset_slam src/main_on_images.cpp)
target_link_libraries(dataset_slam lsdslam)
