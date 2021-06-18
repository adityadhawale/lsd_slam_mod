cmake_minimum_required(VERSION 3.0.2)
project(lsd_slam_viewer)

# Set the build type.  Options are:
#  Coverage       : w/ debug symbols, w/o optimization, w/ code-coverage
#  Debug          : w/ debug symbols, w/o optimization
#  Release        : w/o debug symbols, w/ optimization
#  RelWithDebInfo : w/ debug symbols, w/ optimization
#  MinSizeRel     : w/o debug symbols, w/ optimization, stripped binaries
# set(ROS_BUILD_TYPE Release)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${SSE_FLAGS} -std=c++11 -fopenmp")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS} -O3")

ADD_SUBDIRECTORY(${PROJECT_SOURCE_DIR}/thirdparty/Sophus)

set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR}/cmake)

set(Eigen3_DIR "/home/adi/rando/eigen/release/share/eigen3/cmake")
set(OpenCV_DIR "/home/adi/rando/opencv-2.4.8/release/share/OpenCV")
find_package(OpenCV 2.4.8 REQUIRED EXACT)
find_package(Eigen3 3.3.90 REQUIRED EXACT NO_DEFAULT_PATH)
find_package(Boost REQUIRED)

find_package(OpenGL REQUIRED)
set(QT_USE_QTOPENGL TRUE)
set(QT_USE_QTXML TRUE)
find_package(QGLViewer REQUIRED)
include_directories(${QT_INCLUDES} ${EIGEN3_INCLUDE_DIR} ${QGLVIEWER_INCLUDE_DIR})

find_package(catkin REQUIRED COMPONENTS
  roscpp
  cv_bridge
  dynamic_reconfigure
  rosbag
  sensor_msgs
  message_generation
  std_msgs
  roslib
)


# include(${dynamic_reconfigure_PACKAGE_PATH}/cmake/cfgbuild.cmake)
generate_dynamic_reconfigure_options(
  cfg/LSDSLAMViewerParams.cfg
)

add_message_files(DIRECTORY msg
  FILES 
  keyframeGraphMsg.msg
  keyframeMsg.msg
)
# Messages & Services
# rosbuild_genmsg()
generate_messages(DEPENDENCIES std_msgs)


set(QGLViewer_INCLUDE_DIRS ${QGLVIEWER_INCLUDE_DIR})
set(QGLViewer_LIBRARIES ${QGLVIEWER_LIBRARY})
catkin_package(
  CATKIN_DEPENDS rosbag dynamic_reconfigure roslib cv_bridge
  DEPENDS OpenCV QGLViewer Boost
)

# SSE flags
# rosbuild_check_for_sse()
# set(CMAKE_CXX_FLAGS
   # "${SSE_FLAGS}"
# )

# SSE Sources files
set(SOURCE_FILES         
  src/PointCloudViewer.cpp
  src/KeyFrameDisplay.cpp
  src/KeyFrameGraphDisplay.cpp
  src/settings.cpp
)

set(HEADER_FILES     
  src/PointCloudViewer.h
  src/KeyFrameDisplay.h
  src/KeyFrameGraphDisplay.h
  src/settings.h
)

include_directories(
  src
  cfg/cpp
  ${PROJECT_SOURCE_DIR}/thirdparty/Sophus
  ${catkin_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
)  

add_executable(viewer src/main_viewer.cpp ${SOURCE_FILES})
target_link_libraries(viewer ${catkin_LIBRARIES} ${QGLViewer_LIBRARIES} ${QT_LIBRARIES} ${QGLVIEWER_LIBRARY} GL glut GLU ${OpenCV_LIBRARIES} ${Boost_LIBRARES} boost_system)
add_dependencies(viewer ${PROJECT_NAME}_gencfg)
# link_boost(viewer thread)
add_executable(videoStitch src/main_stitchVideos.cpp)
target_link_libraries(videoStitch ${OpenCV_LIBRARIES} ${Boost_LIBRARES} boost_system)

