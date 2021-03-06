cmake_minimum_required(VERSION 3.11 FATAL_ERROR)
enable_testing()

# Project
set(PROJECT_NAME dnn-kernel)
project(${PROJECT_NAME} LANGUAGES C CXX)


# Default to Debug build type
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Debug CACHE STRING "Build type" FORCE)
endif()

# Project settings
set(DNNK_INCLUDE_DIRS ${PROJECT_SOURCE_DIR} ${PROJECT_SOURCE_DIR}/include)
set(DNNK_CXX_FLAGS "-g" "-Wall" "-Wno-uninitialized" "-Wno-unused-function" "-Wno-unknown-pragmas")

# Vivado HLS
set(VIVADO_HLS_ROOT "/opt/Xilinx/VivadoHLS/2019.1" CACHE STRING "Path to Vivado HLS root directory")
set(VHLS_INCLUDE_DIRS ${VIVADO_HLS_ROOT}/include)


# thirdpartys
include(ExternalProject)
include(FetchContent)

## libtorch
FetchContent_Declare(
  libtorch
  URL https://download.pytorch.org/libtorch/cpu/libtorch-cxx11-abi-shared-with-deps-1.4.0%2Bcpu.zip
)
FetchContent_GetProperties(libtorch)
if(NOT libtorch_POPULATED)
  FetchContent_Populate(libtorch)
endif()

list(APPEND CMAKE_PREFIX_PATH ${libtorch_SOURCE_DIR})
set(TORCH_LIBRARY_DIRS ${libtorch_SOURCE_DIR}/lib)
find_package(Torch REQUIRED)

## googletest
set(GTEST_PREFIX ${PROJECT_BINARY_DIR}/thirdparty/googletest)
set(GTEST_INSTALL ${GTEST_PREFIX}/install)
set(GTEST_INCLUDE_DIRS ${GTEST_INSTALL}/include)
set(GTEST_LIBRARY_DIRS ${GTEST_INSTALL}/lib)
set(GTEST_LIBRARIES ${GTEST_INSTALL}/lib/libgtest.a)

ExternalProject_Add(
  googletest
  PREFIX ${GTEST_PREFIX}
  GIT_REPOSITORY https://github.com/google/googletest
  GIT_TAG release-1.10.0
  CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${GTEST_INSTALL} -DCMAKE_BUILD_TYPE=Release
  BUILD_BYPRODUCTS ${GTEST_LIB}
)

# Logging
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "Path to Vivado HLS: ${VIVADO_HLS_ROOT}")

# tests
add_subdirectory(tests)
