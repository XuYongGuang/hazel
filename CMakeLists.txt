cmake_minimum_required(VERSION 3.0.0)

# Name our project
project(hazel)

# Use C++17
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Create compile_commands.json for clangd
set(CMAKE_EXPORT_COMPILE_COMMANDS 1)

# Enable warnings
add_compile_options(-Wall -Wextra -Wpedantic)

# Default to a release build
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

#
# Hazel
#

# Define the hazel static library
file(GLOB_RECURSE hazel_sources RELATIVE ${PROJECT_SOURCE_DIR} src/hazel/*.cpp)
add_library(hazel STATIC
  ${hazel_sources}

  vendor/stb_image/stb_image.cpp
)

target_compile_definitions(hazel PRIVATE HZ_PLATFORM_MAC)

# Set defines for build type
set(CMAKE_CXX_FLAGS_DEBUG -DHZ_DEBUG)
set(CMAKE_CXX_FLAGS_RELEASE -DHZ_RELEASE)
set(CMAKE_CXX_FLAGS_DIST -DHZ_DIST)

# Include directories for hazel
target_include_directories(hazel PUBLIC
  "${PROJECT_SOURCE_DIR}/src"
  "${PROJECT_SOURCE_DIR}/vendor/glad/include"
  "${PROJECT_SOURCE_DIR}/vendor/glm"
  "${PROJECT_SOURCE_DIR}/vendor/imgui"
  "${PROJECT_SOURCE_DIR}/vendor/spdlog/include"
  "${PROJECT_SOURCE_DIR}/vendor/stb"
)

# glfw (brew install glfw)
find_package(glfw3 3.3 REQUIRED)
target_link_libraries(hazel PRIVATE glfw)
target_compile_options(hazel PRIVATE -DGLFW_INCLUDE_NONE)

# glad
add_subdirectory(vendor/glad)
target_link_libraries(hazel PRIVATE glad)

# imgui
add_subdirectory(vendor/imgui)
target_link_libraries(hazel PRIVATE imgui)

# Precompiled header
set(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/vendor/cotire/CMake")
include(cotire)
set_target_properties(hazel PROPERTIES COTIRE_CXX_PREFIX_HEADER_INIT src/hazel/core/core.hpp)
set_target_properties(hazel PROPERTIES COTIRE_ADD_UNITY_BUILD FALSE)
cotire(hazel)

#
# Sandbox
#

# Define sandbox application
file(GLOB sandbox_sources src/sandbox/*.cpp)
add_executable(sandbox ${sandbox_sources})

# Include directories for the sandbox application
target_include_directories(sandbox PUBLIC "${PROJECT_SOURCE_DIR}/src")

# Link sandbox with the hazel library
target_link_libraries(sandbox PUBLIC hazel)
