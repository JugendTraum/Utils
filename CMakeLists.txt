cmake_minimum_required(VERSION 3.16)

# ------ Project name ------
project(PipelineInfer)

# ------ Standard ------
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Export compile_commands.json
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# ------ Build tool ------
set(CMAKE_VERBOSE_MAKEFILE ON)
set(CMAKE_GENERATOR "Unix Makefiles")

# ------ Build type ------
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "RelWithDebInfo") # default build type
endif()
message(STATUS "CMAKE_BUILD_TYPE: ${CMAKE_BUILD_TYPE}")
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
  add_compile_definitions(__DEBUG__) # set __DEBUG__ for debug release
endif()
if(CMAKE_BUILD_TYPE STREQUAL "RelWithSizeOp")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Oz -pipe -fno-math-errno -flto=auto")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Oz -pipe -fno-math-errno -flto=auto")
  message(${CMAKE_CXX_FLAGS})
endif()
if(CMAKE_BUILD_TYPE STREQUAL "Release")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -pipe -fno-math-errno -flto=auto")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pipe -fno-math-errno -flto=auto")
  message(${CMAKE_CXX_FLAGS})
endif()

# configure files before glob sources.
configure_file(${PROJECT_SOURCE_DIR}/config/config.h.in ${PROJECT_SOURCE_DIR}/config/config.h)

# ------ hardware(待添加) ------
option(WITH_GPU "Use GPU to infer" ON)
option(WITH_RKNPU "Use RKNPU to infer" OFF)

if(WITH_GPU)
# do nothing ... 
endif()

if (WITH_RKNPU)
# do nothing ...
endif()


# ------ developing test ------
option(DEVELOPING_TEST "add test demo in developing" OFF)

if(DEVELOPING_TEST)
  set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin)
  add_executable(test ${PROJECT_SOURCE_DIR}/test/test.cpp)
  target_include_directories(test PRIVATE ${PROJECT_SOURCE_DIR}/config)
  target_include_directories(test PRIVATE ${PROJECT_SOURCE_DIR}/include)
  target_link_libraries(test glog::glog)
endif()

# ------ project ------
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin)
FILE(GLOB_RECURSE ALL_SRC ${PROJECT_SOURCE_DIR}/src/*.cpp)
FILE(GLOB_RECURSE ALL_UTILS ${PROJECT_SOURCE_DIR}/utils/*.cpp)
SET(MAIN_SRC ${PROJECT_SOURCE_DIR}/src/main.cpp)
add_executable(main ${MAIN_SRC} ${ALL_SRC} ${ALL_UTILS})

# src include 
target_include_directories(main PRIVATE ${PROJECT_SOURCE_DIR}/config)
target_include_directories(main PRIVATE ${PROJECT_SOURCE_DIR}/include)
target_include_directories(main PRIVATE ${PROJECT_SOURCE_DIR}/utils)
target_include_directories(main PRIVATE ${PROJECT_SOURCE_DIR}/src)


# ------ dependencies ------
# Fastdeploy
include(${PROJECT_SOURCE_DIR}/cmake/fastdeploy/FastDeploy.cmake)
target_include_directories(main PRIVATE ${FASTDEPLOY_INCS})
target_link_libraries(main ${FASTDEPLOY_LIBS})

# glog
find_package(glog 0.6.0 REQUIRED)
target_link_libraries(main glog::glog)

# restbed
set(RESTBED_INSTALL ${PROJECT_SOURCE_DIR}/3rdparty/restbed/distribution)
target_include_directories(main PRIVATE ${RESTBED_INSTALL}/include)
target_link_libraries(main ${RESTBED_INSTALL}/library/librestbed.so)
