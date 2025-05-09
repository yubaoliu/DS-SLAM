cmake_minimum_required(VERSION 3.18)
#cmake_minimum_required(VERSION 2.4.6)
project(ORB_SLAM2_PointMap_SegNetM)
include($ENV{ROS_ROOT}/core/rosbuild/rosbuild.cmake)
message(STATUS  "ROS_ROOT: $ENV{ROS_ROOT}")

rosbuild_init()

# SET(ROS_BUILD_TYPE Debug)

IF(NOT ROS_BUILD_TYPE)
  SET(ROS_BUILD_TYPE Release)
ENDIF()

MESSAGE("Build type: " ${ROS_BUILD_TYPE})

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -Wall  -O3 -march=native ")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall  -O3 -march=native")

# Check C++11 or C++0x support
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
   add_definitions(-DCOMPILEDWITHC11)
   message(STATUS "Using flag -std=c++11.")
elseif(COMPILER_SUPPORTS_CXX0X)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
   add_definitions(-DCOMPILEDWITHC0X)
   message(STATUS "Using flag -std=c++0x.")
else()
   message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

LIST(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/../../../cmake_modules)

find_package(OpenCV  QUIET) #3.3.1
if(NOT OpenCV_FOUND)
   find_package(OpenCV 2.4.3 QUIET)
   if(NOT OpenCV_FOUND)
      message(FATAL_ERROR "OpenCV > 2.4.3 not found.")
   endif()
endif()

find_package(Caffe HINTS "/home/yubao/data/software/install/caffe-segnet/share/Caffe/" )
add_definitions(${Caffe_DEFINITIONS})

find_package(Eigen3 3.1.0 REQUIRED)
# find_package(Eigen3  REQUIRED)
find_package(Pangolin REQUIRED)
#find_package( PCL 1.7 REQUIRED )
find_package( PCL  REQUIRED )
find_package( octomap REQUIRED )

include_directories(
    ${PROJECT_SOURCE_DIR}
    ${PROJECT_SOURCE_DIR}/../../../
    ${PROJECT_SOURCE_DIR}/../../../include
    ${Pangolin_INCLUDE_DIRS}
    ${PROJECT_SOURCE_DIR}/../../../Examples/ROS/ORB_SLAM2_PointMap_SegNetM/libsegmentation
    ${Caffe_INCLUDE_DIRS}
    ${PCL_INCLUDE_DIRS}
    ${OCTOMAP_INCLUDE_DIRS}
    # ${PROJECT_SOURCE_DIR}/../../../Examples/ROS/ORB_SLAM2_PointMap_SegNetM/caffe-segnet-cudnn5/include 
#${PROJECT_SOURCE_DIR}/../../../Examples/ROS/ORB_SLAM2_PointMap_SegNetM/caffe-segnet-cudnn5/build/include
    # /usr/local/include  
    /usr/local/cuda/include  
    # /usr/include
)

set( LIBS 
    ${Caffe_LIBRARIES}
    ${OpenCV_LIBS} 
    ${EIGEN3_LIBS}
    ${Pangolin_LIBRARIES}
    ${PCL_LIBRARIES}
    ${OCTOMAP_LIBRARIES}
    ${PROJECT_SOURCE_DIR}/../../../Thirdparty/DBoW2/lib/libDBoW2.so
    ${PROJECT_SOURCE_DIR}/../../../Thirdparty/g2o/lib/libg2o.so
    ${PROJECT_SOURCE_DIR}/../../../Examples/ROS/ORB_SLAM2_PointMap_SegNetM/libsegmentation/build/libsegmentation.so
    ${PROJECT_SOURCE_DIR}/../../../build/libORB_SLAM2_PointMap_SegNetM.so
    # ${PROJECT_SOURCE_DIR}/../../../Examples/ROS/ORB_SLAM2_PointMap_SegNetM/caffe-segnet-cudnn5/lib/libcaffe.so  
    /usr/lib/x86_64-linux-gnu/libglog.so 
    -lboost_system
)


add_definitions( ${PCL_DEFINITIONS} )
link_directories( ${PCL_LIBRARY_DIRS} )

rosbuild_add_executable(TUM
    ros_tum_realtime.cc
)
target_link_libraries(TUM
    ${LIBS}
)

rosbuild_add_executable(TUM_bag
    ros_tum_bag.cpp
)
target_link_libraries(TUM_bag
    ${LIBS}
)
