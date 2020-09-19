# Prevent CMake from generating in-tree builds
if(CMAKE_SOURCE_DIR STREQUAL CMAKE_BINARY_DIR)
  message(FATAL_ERROR "Prevented in-tree build. Please specify another directory outside of the source root.")
endif()

cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
project(Vulkan-Engine VERSION 0.1.0 LANGUAGES CXX)

message(NOTICE "----------------------")
message(NOTICE "Building Vulkan-Engine")
message(NOTICE "----------------------")
message(NOTICE "Source Dir: ${PROJECT_SOURCE_DIR}")
message(NOTICE "Binary Dir: ${PROJECT_BINARY_DIR}")

# Download the CMake wrapper for the Conan C and C++ package manager
if(NOT EXISTS "${CMAKE_BINARY_DIR}/conan.cmake")
    message(STATUS "Downloading conan.cmake from https://github.com/conan-io/cmake-conan")
    file(DOWNLOAD "https://github.com/conan-io/cmake-conan/raw/v0.15/conan.cmake" "${CMAKE_BINARY_DIR}/conan.cmake")
endif()
include(${CMAKE_BINARY_DIR}/conan.cmake)
conan_check(VERSION 1.28.1 REQUIRED)

# Run Conan and retrieve dependencies
conan_cmake_run(
    CONANFILE conanfile.txt
    BASIC_SETUP
    CMAKE_TARGETS
    BUILD missing
)

# Find Vulkan package
find_package(Vulkan REQUIRED)

# Require a C++17 compatible compiler
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Configure header file
configure_file(${PROJECT_SOURCE_DIR}/Include/Config.hpp.in Config.hpp @ONLY)

set(
    SOURCE_FILES
    Source/Main.cpp
    Source/Application.cpp
)

set(
    INCLUDE_FILES
    Include/Application.hpp
)

add_executable(Vulkan-Engine ${SOURCE_FILES} ${INCLUDE_FILES})

target_include_directories(
    Vulkan-Engine
    PUBLIC
    ${PROJECT_SOURCE_DIR}/Include
    ${PROJECT_BINARY_DIR}
)

target_link_libraries(
    Vulkan-Engine
    PUBLIC
    CONAN_PKG::sdl2
    Vulkan::Vulkan
)