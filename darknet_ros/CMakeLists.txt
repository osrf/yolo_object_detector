cmake_minimum_required(VERSION 2.8.3)
project(darknet_ros)

## Compile as C++11, supported in ROS Kinetic and newer
# add_compile_options(-std=c++11)

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED)

## System dependencies are found with CMake's conventions
# find_package(Boost REQUIRED COMPONENTS system)

## CPU or GPU
option(CPUOnly "CPUOnly" OFF)

find_package(Threads)
list(APPEND LIBRARIES ${CMAKE_THREAD_LIBS_INIT})


if(NOT CPUOnly)
find_package(CUDA)
endif()

if (CUDA_FOUND)
    message(STATUS "CUDA Version:" ${CUDA_VERSION_STRINGS})
    message(STATUS "CUDA Libraries: " ${CUDA_LIBRARIES})
    include_directories(SYSTEM ${CUDA_INCLUDE_DIRS})
    list(APPEND LIBRARIES ${CUDA_LIBRARIES} ${CUDA_CUBLAS_LIBRARIES} ${CUDA_curand_LIBRARY} ${CUDA_cusparse_LIBRARY})
    list(APPEND CUDA_NVCC_FLAGS "-std=c++11;-O2;-Xcompiler \"-fPIC\" ")
    # list(APPEND CUDA_NVCC_FLAGS "-gencode arch=compute_20,code=compute_20 ")
    # list(APPEND CUDA_NVCC_FLAGS "-gencode arch=compute_30,code=compute_30 ")
    # list(APPEND CUDA_NVCC_FLAGS "-gencode arch=compute_35,code=compute_35 ")
    # list(APPEND CUDA_NVCC_FLAGS "-gencode arch=compute_50,code=compute_50 ")
    list(APPEND CUDA_NVCC_FLAGS "-gencode arch=compute_52,code=compute_52 ")
    set(CUDA_PROPAGATE_HOST_FLAGS OFF)
    add_definitions(-DGPU)
    cuda_include_directories(src)
else()
    list(APPEND LIBRARIES "m")
endif()

find_package(OpenCV REQUIRED)

add_definitions( -DOPENCV)
add_definitions( -O4 -g)
include_directories( SYSTEM ${OpenCV_INCLUDE_DIR})

include_directories(src)
set(SRC_DIR src)

FILE(GLOB SRC_FILES ${SRC_DIR}/*.c)
FILE(GLOB CU_SRC_FILES ${SRC_DIR}/*.cu)
LIST(REMOVE_ITEM SRC_FILES "${PROJECT_SOURCE_DIR}/src/darknet.c")
LIST(REMOVE_ITEM SRC_FILES "${PROJECT_SOURCE_DIR}/src/demo.c")
LIST(REMOVE_ITEM SRC_FILES "${PROJECT_SOURCE_DIR}/src/yolo.c")
LIST(REMOVE_ITEM SRC_FILES "${PROJECT_SOURCE_DIR}/src/coco.c")
LIST(REMOVE_ITEM SRC_FILES "${PROJECT_SOURCE_DIR}/src/detector.c")

catkin_package(
  INCLUDE_DIRS src
  LIBRARIES darknet_ros
)

if (CUDA_FOUND)
  message(STATUS "CUDA FOUND")
  cuda_add_library(${PROJECT_NAME} SHARED ${SRC_FILES} ${CU_SRC_FILES})
  target_link_libraries(${PROJECT_NAME} ${LIBRARIES} ${OpenCV_LIBS})
else ()
  add_library(${PROJECT_NAME} SHARED ${SRC_FILES})
  target_link_libraries(${PROJECT_NAME} ${LIBRARIES} ${OpenCV_LIBS})
endif()

#############
## Install ##
#############

# all install targets should use catkin DESTINATION variables
# See http://ros.org/doc/api/catkin/html/adv_user_guide/variables.html

## Mark executable scripts (Python etc.) for installation
## in contrast to setup.py, you can choose the destination
# install(PROGRAMS
#   scripts/my_python_script
#   DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

## Mark executables and/or libraries for installation
# install(TARGETS ${PROJECT_NAME} ${PROJECT_NAME}_node
#   ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#   LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#   RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

install(TARGETS ${PROJECT_NAME}
        LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
	RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

## Mark cpp header files for installation
# install(DIRECTORY include/${PROJECT_NAME}/
#   DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
#   FILES_MATCHING PATTERN "*.h"
#   PATTERN ".svn" EXCLUDE
# )

install(DIRECTORY ${PROJECT_NAME} 
	DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
            FILES_MATCHING PATTERN "*.h" )

## Mark other files for installation (e.g. launch and bag files, etc.)
# install(FILES
#   # myfile1
#   # myfile2
#   DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
# )

#############
## Testing ##
#############

## Add gtest based cpp test target and link libraries
# catkin_add_gtest(${PROJECT_NAME}-test test/test_darknet_ros.cpp)
# if(TARGET ${PROJECT_NAME}-test)
#   target_link_libraries(${PROJECT_NAME}-test ${PROJECT_NAME})
# endif()

## Add folders to be run by python nosetests
# catkin_add_nosetests(test)
