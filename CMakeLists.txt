cmake_minimum_required(VERSION 2.8.3)
project(snap_rtps_examples)

###########
## Build ##
###########

message( "MV_SDK: $ENV{MV_SDK}"  )
message( "HEXAGON_ARM_SYSROOT: $ENV{HEXAGON_ARM_SYSROOT}"  )

add_subdirectory(src/nodes/rtps-msgs)

## Specify additional locations of header files
## Your package locations should be listed before other locations
# include_directories(include)
include_directories(
  $ENV{MV_SDK}/usr/include/mv
  $ENV{HEXAGON_ARM_SYSROOT}/usr/include
  ${CMAKE_BINARY_DIR}/src/nodes/rtps-msgs
  src/nodes/Matrix
  src/nodes/
  /usr/local/include
)

add_definitions(-std=c++11) 

#set(CMAKE_TOOLCHAIN_FILE ${CMAKE_SOURCE_DIRECTORY}/cmake_hexagon/toolchain/Toolchain-arm-linux-gnueabihf.cmake)

include(cmake_hexagon/toolchain/Toolchain-arm-linux-gnueabihf.cmake)

## Declare a C++ library
add_library(snap_vislam
   src/camera/SnapdragonCameraManager.cpp
   src/common/SnapdragonDebugPrint.cpp
   src/camera/SnapdragonCameraUtil.cpp
   src/imu/SnapdragonImuManager.cpp
   src/vislam/SnapdragonVislamManager.cpp
)

target_include_directories( snap_vislam PUBLIC 
  src/camera/
  src/imu/
  src/vislam/
  src/common/
  $ENV{MV_SDK}/usr/include/mv )

# Make sure we include lib dir for Linux build
add_library(mv1 SHARED IMPORTED)
set_property(TARGET mv1 PROPERTY IMPORTED_LOCATION $ENV{MV_SDK}/usr/lib/libmv1.so)

add_executable( test_snap_cam_manager test/test_SnapdragonCameraManager.cpp )
target_link_libraries( test_snap_cam_manager snap_vislam sensor_imu mv1 pthread camera  )

add_executable( test_imu_manager test/test_SnapdragonImuManager.cpp )
target_link_libraries( test_imu_manager snap_vislam sensor_imu mv1 pthread camera  )

add_executable( test_vislam test/test_SnapdragonVislamManager.cpp )
target_link_libraries( test_vislam snap_vislam sensor_imu mv1 pthread camera  )

## Declare a C++ executable
add_executable(snap_vislam_node src/nodes/SnapdragonRtpsNodeVislam.cpp src/nodes/SnapdragonRtpsNodeVislam_main.cpp )

## Specify libraries to link a library or executable target against
 target_link_libraries(snap_vislam_node
   snap_vislam
   mv1 
   camera
   rtps-topics
   sensor_imu
   pthread
)

target_include_directories( snap_vislam_node PRIVATE src/nodes/ )
set(CMAKE_EXE_LINKER_FLAGS "-Wl,-gc-sections -Wl,-rpath-link,${CMAKE_SYSROOT}/linaro-rootfs/lib/arm-linux-gnueabihf -Wl,-rpath-link,${CMAKE_SYSROOT}/linaro-rootfs/usr/lib/arm-linux-gnueabihf")

