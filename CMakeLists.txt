# SPDX-License-Identifier: MIT

cmake_minimum_required(VERSION 3.40.0)

project(CMakeForImGui CXX)
include(CMakeDependentOption)

if(NOT IMGUI_SOURCE_DIR)
  message(FATAL_ERROR "set IMGUI_SOURCE_DIR to proceed.\ne.g. cmake -DIMGUI_SOURCE_DIR=path.to.imgui -S . -B build")
endif()

set(_imdir ${IMGUI_SOURCE_DIR})

option(with_miscellaneous "" OFF)

option(glfw "" OFF)
option(opengl2 "" OFF)
option(opengl3 "" OFF)
option(vulkan "" OFF)
option(sdl3 "" OFF)

option(examples "" OFF)

cmake_dependent_option(opengl3 "" ON glfw ON)
cmake_dependent_option(opengl2 "" ON glfw ON)

cmake_dependent_option(example_glfw_metal "" OFF glfw;examples OFF)
cmake_dependent_option(example_glfw_opengl2 "" OFF glfw;examples OFF)
cmake_dependent_option(example_glfw_opengl3 "" OFF glfw;examples OFF)
cmake_dependent_option(example_glfw_vulkan "" OFF glfw;examples OFF)
cmake_dependent_option(example_glfw_wgpu "" OFF glfw;examples OFF)

cmake_dependent_option(example_null "" OFF examples OFF)

cmake_dependent_option(example_sdl3_opengl3 "" OFF sdl3;examples OFF)
option(visualize_targets "" ON)
add_library(imgui_core)
target_sources(imgui_core PRIVATE ${_imdir}/imgui.cpp ${_imdir}/imgui_demo.cpp ${_imdir}/imgui_draw.cpp ${_imdir}/imgui_tables.cpp ${_imdir}/imgui_widgets.cpp
                                  ${_imdir}/imgui_widgets.cpp)
target_sources(
  imgui_core
  PUBLIC FILE_SET
         imgui_core_set
         BASE_DIRS
         ${_imdir}
         TYPE
         HEADERS
         FILES
         ${_imdir}/imgui.h
         ${_imdir}/imconfig.h
         ${_imdir}/imgui_internal.h
         ${_imdir}/imstb_rectpack.h
         ${_imdir}/imstb_textedit.h
         ${_imdir}/imstb_truetype.h)

if(with_miscellaneous)
  add_library(imgui_misc_cpp)
  target_sources(imgui_misc_cpp PRIVATE ${_imdir}/misc/cpp/imgui_stdlib.cpp)
  target_sources(imgui_misc_cpp PUBLIC FILE_SET imgui_misc_cpp_set BASE_DIRS ${_imdir} TYPE HEADERS FILES ${_imdir}/misc/cpp/imgui_stdlib.h ${_imdir}/imgui.h)

  target_link_libraries(imgui_core PUBLIC imgui_misc_cpp)
endif()

install(
  EXPORT imgui_core_export
  DESTINATION lib/cmake
  NAMESPACE imgui::
  FILE ImGuiConfig.cmake)

install(
  TARGETS imgui_core imgui_misc_cpp
  EXPORT imgui_core_export
  FILE_SET imgui_core_set FILE_SET imgui_misc_cpp_set)

if(opengl2)
  find_package(OpenGL 2 REQUIRED)

  add_library(imgui_backend_opengl2)
  target_sources(imgui_backend_opengl2 PRIVATE ${_imdir}/backends/imgui_impl_opengl2.cpp)
  target_link_libraries(imgui_backend_opengl2 PUBLIC imgui_core)
  target_link_libraries(imgui_backend_opengl2 PUBLIC OpenGL::GL)
  target_sources(imgui_backend_opengl2 PUBLIC FILE_SET imgui_backend_opengl2_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_opengl2.h)

  install(
    TARGETS imgui_backend_opengl2
    EXPORT imgui_core_export
    FILE_SET imgui_backend_opengl2_set)
endif()

if(opengl3)
  find_package(OpenGL 3 REQUIRED)

  add_library(imgui_backend_opengl3)
  target_sources(imgui_backend_opengl3 PRIVATE ${_imdir}/backends/imgui_impl_opengl3.cpp)
  target_link_libraries(imgui_backend_opengl3 PUBLIC imgui_core)
  target_link_libraries(imgui_backend_opengl3 PUBLIC OpenGL::GL)
  target_sources(imgui_backend_opengl3 PUBLIC FILE_SET imgui_backend_opengl3_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_opengl3.h)

  install(
    TARGETS imgui_backend_opengl3
    EXPORT imgui_core_export
    FILE_SET imgui_backend_opengl3_set)

endif()

#[[
if(vulkan)
  find_package(Vulkan REQUIRED)

  add_library(imgui_backend_vulkan)
  target_sources(imgui_backend_vulkan PUBLIC backends/imgui_impl_vulkan.cpp)
  target_sources(imgui_backend_vulkan PUBLIC FILE_SET imgui_backend_vulkan_set BASE_DIRS backends TYPE HEADERS FILES backends/imgui_impl_vulkan.h)
  target_link_libraries(imgui_backend_vulkan PUBLIC Vulkan::Vulkan)
  target_link_libraries(imgui_backend_vulkan PUBLIC imgui_core)
  add_library(ImGui::backend::vulkan ALIAS imgui_backend_vulkan)
endif()

if(glfw)
  find_package(glfw3 CONFIG REQUIRED)

  add_library(imgui_backend_glfw)
  target_sources(imgui_backend_glfw PUBLIC backends/imgui_impl_glfw.cpp)
  target_sources(imgui_backend_glfw PUBLIC FILE_SET imgui_backend_glfw_set BASE_DIRS backends TYPE HEADERS FILES backends/imgui_impl_glfw.h)
  target_link_libraries(imgui_backend_glfw PUBLIC glfw)
  target_link_libraries(imgui_backend_glfw PUBLIC imgui_core imgui_backend_opengl3)
  add_library(ImGui::backend::glfw ALIAS imgui_backend_glfw)

  if(example_glfw_opengl3)
    add_executable(glfw_example_opengl3)
    target_sources(glfw_example_opengl3 PRIVATE examples/example_glfw_opengl3/main.cpp)
    target_link_libraries(glfw_example_opengl3 PRIVATE imgui_backend_glfw)
  endif()

  if(example_glfw_opengl2)
    add_executable(glfw_example_opengl2)
    target_sources(glfw_example_opengl2 PRIVATE examples/example_glfw_opengl2/main.cpp)
    target_link_libraries(glfw_example_opengl2 PRIVATE imgui_backend_glfw imgui_backend_opengl2)
  endif()

  if(example_glfw_vulkan)
    add_executable(glfw_example_vulkan)
    target_sources(glfw_example_vulkan PRIVATE examples/example_glfw_vulkan/main.cpp)
    target_link_libraries(glfw_example_vulkan PRIVATE imgui_backend_glfw imgui_backend_vulkan)
  endif()
endif()

if(sdl3)
  find_package(SDL3 REQUIRED)
  add_library(imgui_backend_sdl3)
  target_sources(imgui_backend_sdl3 PUBLIC backends/imgui_impl_sdl3.cpp)
  target_sources(imgui_backend_sdl3 PUBLIC FILE_SET imgui_backend_sdl3_set BASE_DIRS backends TYPE HEADERS FILES backends/imgui_impl_sdl3.h)
  target_link_libraries(imgui_backend_sdl3 PUBLIC SDL3::SDL3)
  target_link_libraries(imgui_backend_sdl3 PUBLIC imgui_core)
  add_library(ImGui::backend::sdl3 ALIAS imgui_backend_sdl3)

  if(example_sdl3_opengl3)
    add_executable(sdl3_example_opengl3)
    target_sources(sdl3_example_opengl3 PRIVATE examples/example_sdl3_opengl3/main.cpp)
    target_link_libraries(sdl3_example_opengl3 PRIVATE imgui_backend_sdl3 imgui_backend_opengl3)
  endif()

endif()

if(example_null)
  add_executable(example_null)
  target_sources(example_null PRIVATE examples/example_null/main.cpp)
  target_sources(example_null PRIVATE FILE_SET example_null_set TYPE HEADERS FILES ${IMGUI_SOURCE_DIR}/imgui.h)
  target_link_libraries(example_null PRIVATE imgui_core)
  target_include_directories(example_null PRIVATE ${IMGUI_SOURCE_DIR})
endif()

if(visualize_targets)
  include(CMakeGraphVizOptions)

  find_program(DOT_COMMAND dot REQUIRED)

  add_custom_target(
    graph ALL
    COMMAND ${CMAKE_COMMAND} --graphviz=tgt.dot ${PROJECT_BINARY_DIR}
    COMMAND ${DOT_COMMAND} -Tpng tgt.dot -o targets.png
    COMMAND feh targets.png
    WORKING_DIRECTORY ${PROJECT_BINARY_DIR})
endif()
]]
#
