# SPDX-License-Identifier: MIT

cmake_minimum_required(VERSION 3.40.0)

project(ImGui CXX)
include(CMakeDependentOption)
include(CMakeGraphVizOptions)

set(IMGUI_SOURCE_DIR ${PROJECT_SOURCE_DIR})

option(glfw "" OFF)

cmake_dependent_option(opengl3 "" ON glfw ON)

cmake_dependent_option(example_glfw_metal "" OFF glfw OFF)
cmake_dependent_option(example_glfw_opengl2 "" OFF glfw OFF)
cmake_dependent_option(example_glfw_opengl3 "" OFF glfw OFF)
cmake_dependent_option(example_glfw_vulkan "" OFF glfw OFF)
cmake_dependent_option(example_glfw_wgpu "" OFF glfw OFF)

option(visualize_targets "" ON)

add_library(core)
target_sources(core PUBLIC imgui.cpp imgui_demo.cpp imgui_draw.cpp imgui_tables.cpp imgui_widgets.cpp imgui_widgets.cpp)
target_include_directories(core PUBLIC ${IMGUI_SOURCE_DIR})
add_library(ImGui::core ALIAS core)

if(opengl3)
  find_package(OpenGL 3 REQUIRED)
  add_library(imgui_backend_opengl3)
  target_sources(imgui_backend_opengl3 PUBLIC backends/imgui_impl_opengl3.cpp)
  target_include_directories(imgui_backend_opengl3 PUBLIC ${IMGUI_SOURCE_DIR} ${IMGUI_SOURCE_DIR}/backends)
  target_link_libraries(imgui_backend_opengl3 PUBLIC ImGui::core)
  target_link_libraries(imgui_backend_opengl3 PUBLIC OpenGL::GL)
  add_library(ImGui::backend::opengl3 ALIAS imgui_backend_opengl3)
endif()

if(glfw)
  find_package(glfw3 CONFIG REQUIRED)

  add_library(imgui_backend_glfw)
  target_sources(imgui_backend_glfw PUBLIC backends/imgui_impl_glfw.cpp)
  target_include_directories(imgui_backend_glfw PUBLIC ${IMGUI_SOURCE_DIR} ${IMGUI_SOURCE_DIR}/backends)
  target_link_libraries(imgui_backend_glfw PUBLIC ImGui::backend::opengl3)
  target_link_libraries(imgui_backend_glfw PUBLIC glfw)
  add_library(ImGui::backend::glfw ALIAS core)

  if(example_glfw_opengl3)
    add_executable(glfw_example_opengl3)
    target_sources(glfw_example_opengl3 PUBLIC examples/example_glfw_opengl3/main.cpp)
    target_link_libraries(glfw_example_opengl3 PUBLIC imgui_backend_glfw)
  endif()

endif()

if(visualize_targets)
  find_program(DOT_COMMAND dot REQUIRED)

  add_custom_target(
    graph ALL
    COMMAND ${CMAKE_COMMAND} --graphviz=tgt.dot ${PROJECT_BINARY_DIR}
    COMMAND ${DOT_COMMAND} -Tpng tgt.dot -o targets.png
    COMMAND feh targets.png
    WORKING_DIRECTORY ${PROJECT_BINARY_DIR})
endif()
