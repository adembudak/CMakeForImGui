# SPDX-License-Identifier: MIT

cmake_minimum_required(VERSION 3.40.0)

project(CMakeForImGui CXX)
include(CMakeDependentOption)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED 1)

if(NOT IMGUI_SOURCE_DIR)
  message(FATAL_ERROR "set IMGUI_SOURCE_DIR to proceed:\ne.g. cmake -DIMGUI_SOURCE_DIR=path.to.imgui ...")
endif()

set(_imdir ${IMGUI_SOURCE_DIR})

option(with_miscellaneous "" OFF)
option(allegro5 "" OFF)
option(glfw "" OFF)
option(glut "" OFF)
option(sdl2 "" OFF)
option(sdlrenderer2 "" OFF)
option(sdl3 "" OFF)
option(opengl2 "" OFF)
option(opengl3 "" OFF)
option(vulkan "" OFF)
option(examples "" OFF)
option(visualize_targets "" OFF)

if(WIN32)
  option(win32 "" OFF)
endif()

cmake_dependent_option(example_glfw_opengl2 "" OFF "examples AND glfw AND opengl2" OFF)
cmake_dependent_option(example_glfw_opengl3 "" OFF "examples AND glfw AND opengl3" OFF)
cmake_dependent_option(example_glfw_vulkan "" OFF "examples AND glfw AND vulkan" OFF)
cmake_dependent_option(example_glut_opengl2 "" OFF "examples AND glut AND opengl2" OFF)

cmake_dependent_option(example_sdl2_opengl2 "" OFF "examples AND sdl2 AND opengl2" OFF)
cmake_dependent_option(example_sdl2_opengl3 "" OFF "examples AND sdl2 AND opengl3" OFF)
cmake_dependent_option(example_sdl2_vulkan "" OFF "examples AND sdl2 AND vulkan" OFF)

cmake_dependent_option(example_sdl2_sdlrenderer2 "" OFF "examples AND sdlrenderer2 AND sdl2" OFF)

cmake_dependent_option(example_allegro5 "" OFF "examples AND allegro5" OFF)

cmake_dependent_option(example_sdl3_opengl3 "" OFF "examples AND sdl3 AND opengl3" OFF)
cmake_dependent_option(example_sdl3_vulkan "" OFF "examples AND sdl3 AND vulkan" OFF)

cmake_dependent_option(example_null "" OFF examples OFF)

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

install(
  EXPORT imgui_core_export
  DESTINATION lib/cmake
  NAMESPACE imgui::
  FILE ImGuiConfig.cmake)

install(
  TARGETS imgui_core
  EXPORT imgui_core_export
  FILE_SET imgui_core_set)

install(FILES ${_imdir}/LICENSE.txt TYPE DATA)

if(with_miscellaneous)
  add_library(imgui_misc_cpp)
  target_sources(imgui_misc_cpp PRIVATE ${_imdir}/misc/cpp/imgui_stdlib.cpp)
  target_sources(imgui_misc_cpp PRIVATE FILE_SET HEADERS BASE_DIRS ${_imdir} FILES ${_imdir}/imgui.h)
  target_link_libraries(imgui_core PUBLIC imgui_misc_cpp)
  target_sources(imgui_misc_cpp PUBLIC FILE_SET imgui_misc_cpp_set BASE_DIRS ${_imdir}/misc/cpp TYPE HEADERS FILES ${_imdir}/misc/cpp/imgui_stdlib.h)

  find_package(freetype CONFIG REQUIRED)

  add_library(imgui_misc_freetype)
  target_sources(imgui_misc_freetype PRIVATE ${_imdir}/misc/freetype/imgui_freetype.cpp)
  target_sources(imgui_misc_freetype PRIVATE FILE_SET HEADERS BASE_DIRS ${_imdir} FILES ${_imdir}/imgui.h)
  target_link_libraries(imgui_misc_freetype PRIVATE Freetype::Freetype)
  target_link_libraries(imgui_core PUBLIC imgui_misc_freetype)
  target_sources(imgui_misc_freetype PUBLIC FILE_SET imgui_misc_freetype_set BASE_DIRS ${_imdir}/misc/freetype TYPE HEADERS FILES ${_imdir}/misc/freetype/imgui_freetype.h)

  install(
    TARGETS imgui_misc_cpp imgui_misc_freetype
    EXPORT imgui_core_export
    FILE_SET imgui_misc_cpp_set FILE_SET imgui_misc_freetype_set)
endif()

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

if(vulkan)
  find_package(Vulkan REQUIRED)

  add_library(imgui_backend_vulkan)
  target_sources(imgui_backend_vulkan PRIVATE ${_imdir}/backends/imgui_impl_vulkan.cpp)
  target_link_libraries(imgui_backend_vulkan PUBLIC imgui_core)
  target_link_libraries(imgui_backend_vulkan PUBLIC Vulkan::Vulkan)
  target_sources(imgui_backend_vulkan PUBLIC FILE_SET imgui_backend_vulkan_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_vulkan.h)

  install(
    TARGETS imgui_backend_vulkan
    EXPORT imgui_core_export
    FILE_SET imgui_backend_vulkan_set)
endif()

if(allegro5)
  find_package(Allegro CONFIG REQUIRED)
  add_library(imgui_backend_allegro5)
  target_sources(imgui_backend_allegro5 PRIVATE ${_imdir}/backends/imgui_impl_allegro5.cpp)
  target_link_libraries(imgui_backend_allegro5 PUBLIC imgui_core)
  target_link_libraries(imgui_backend_allegro5 PUBLIC Allegro::allegro Allegro::allegro_primitives)
  target_sources(imgui_backend_allegro5 PUBLIC FILE_SET imgui_backend_allegro5_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_allegro5.h)

  install(
    TARGETS imgui_backend_allegro5
    EXPORT imgui_core_export
    FILE_SET imgui_backend_allegro5_set)

  if(example_allegro5)
    add_executable(example_allegro5)
    target_sources(example_allegro5 PRIVATE ${_imdir}/examples/example_allegro5/main.cpp)
    target_link_libraries(example_allegro5 PRIVATE imgui_backend_allegro5)
  endif()

endif()

if(glfw)
  find_package(glfw3 CONFIG REQUIRED)

  add_library(imgui_backend_glfw)
  target_sources(imgui_backend_glfw PRIVATE ${_imdir}/backends/imgui_impl_glfw.cpp)
  target_sources(imgui_backend_glfw PUBLIC FILE_SET imgui_backend_glfw_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_glfw.h)
  target_link_libraries(imgui_backend_glfw PUBLIC glfw)
  target_link_libraries(imgui_backend_glfw PUBLIC imgui_core imgui_backend_opengl3)

  install(
    TARGETS imgui_backend_glfw
    EXPORT imgui_core_export
    FILE_SET imgui_backend_glfw_set)

  if(example_glfw_opengl3)
    add_executable(glfw_example_opengl3)
    target_sources(glfw_example_opengl3 PRIVATE ${_imdir}/examples/example_glfw_opengl3/main.cpp)
    target_link_libraries(glfw_example_opengl3 PRIVATE imgui_backend_glfw)
  endif()

  if(example_glfw_opengl2)
    add_executable(glfw_example_opengl2)
    target_sources(glfw_example_opengl2 PRIVATE ${_imdir}/examples/example_glfw_opengl2/main.cpp)
    target_link_libraries(glfw_example_opengl2 PRIVATE imgui_backend_glfw imgui_backend_opengl2)
  endif()

  if(example_glfw_vulkan)
    add_executable(glfw_example_vulkan)
    target_sources(glfw_example_vulkan PRIVATE ${_imdir}/examples/example_glfw_vulkan/main.cpp)
    target_link_libraries(glfw_example_vulkan PRIVATE imgui_backend_glfw imgui_backend_vulkan)
  endif()
endif()

if(glut)
  find_package(FreeGLUT CONFIG REQUIRED)

  add_library(imgui_backend_glut)
  target_sources(imgui_backend_glut PRIVATE ${_imdir}/backends/imgui_impl_glut.cpp)
  target_sources(imgui_backend_glut PUBLIC FILE_SET imgui_backend_glut_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_glut.h)
  target_link_libraries(imgui_backend_glut PUBLIC FreeGLUT::freeglut)
  target_link_libraries(imgui_backend_glut PUBLIC imgui_core)

  install(
    TARGETS imgui_backend_glut
    EXPORT imgui_core_export
    FILE_SET imgui_backend_glut_set)

  if(example_glut_opengl2)
    add_executable(glut_example_opengl2)
    target_sources(glut_example_opengl2 PRIVATE ${_imdir}/examples/example_glut_opengl2/main.cpp)
    target_link_libraries(glut_example_opengl2 PRIVATE imgui_backend_glut imgui_backend_opengl2)
  endif()
endif()

if(sdl2)
  find_package(SDL2 CONFIG REQUIRED)

  add_library(imgui_backend_sdl2)
  target_sources(imgui_backend_sdl2 PRIVATE ${_imdir}/backends/imgui_impl_sdl2.cpp)
  target_sources(imgui_backend_sdl2 PUBLIC FILE_SET imgui_backend_sdl2_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_sdl2.h)
  target_link_libraries(imgui_backend_sdl2 PUBLIC SDL2::SDL2)
  target_link_libraries(imgui_backend_sdl2 PUBLIC imgui_core)

  install(
    TARGETS imgui_backend_sdl2
    EXPORT imgui_core_export
    FILE_SET imgui_backend_sdl2_set)

  if(example_sdl2_opengl2)
    add_executable(example_sdl2_opengl2)
    target_sources(example_sdl2_opengl2 PRIVATE ${_imdir}/examples/example_sdl2_opengl2/main.cpp)
    target_link_libraries(example_sdl2_opengl2 PRIVATE imgui_backend_sdl2 imgui_backend_opengl2)
  endif()

  if(example_sdl2_opengl3)
    add_executable(example_sdl2_opengl3)
    target_sources(example_sdl2_opengl3 PRIVATE ${_imdir}/examples/example_sdl2_opengl3/main.cpp)
    target_link_libraries(example_sdl2_opengl3 PRIVATE imgui_backend_sdl2 imgui_backend_opengl3)
  endif()

  if(example_sdl2_vulkan)
    add_executable(example_sdl2_vulkan)
    target_sources(example_sdl2_vulkan PRIVATE ${_imdir}/examples/example_sdl2_vulkan/main.cpp)
    target_link_libraries(example_sdl2_vulkan PRIVATE imgui_backend_sdl2 imgui_backend_vulkan)
  endif()
endif()

if(sdlrenderer2)
  find_package(SDL2 CONFIG REQUIRED)
  add_library(imgui_backend_sdlrenderer2)
  target_sources(imgui_backend_sdlrenderer2 PRIVATE ${_imdir}/backends/imgui_impl_sdlrenderer2.cpp)
  target_sources(imgui_backend_sdlrenderer2 PUBLIC FILE_SET imgui_backend_sdlrenderer2_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES
                                                   ${_imdir}/backends/imgui_impl_sdlrenderer2.h)
  target_link_libraries(imgui_backend_sdlrenderer2 PUBLIC SDL2::SDL2)
  target_link_libraries(imgui_backend_sdlrenderer2 PUBLIC imgui_core)

  install(
    TARGETS imgui_backend_sdlrenderer2
    EXPORT imgui_core_export
    FILE_SET imgui_backend_sdlrenderer2_set)

  if(example_sdl2_sdlrenderer2)
    add_executable(example_sdl2_sdlrenderer2)
    target_sources(example_sdl2_sdlrenderer2 PRIVATE ${_imdir}/examples/example_sdl2_sdlrenderer2/main.cpp)
    target_link_libraries(example_sdl2_sdlrenderer2 PRIVATE imgui_backend_sdlrenderer2 imgui_backend_sdl2)
  endif()
endif()

if(sdl3)
  find_package(SDL3 CONFIG REQUIRED)
  add_library(imgui_backend_sdl3)
  target_sources(imgui_backend_sdl3 PRIVATE ${_imdir}/backends/imgui_impl_sdl3.cpp)
  target_sources(imgui_backend_sdl3 PUBLIC FILE_SET imgui_backend_sdl3_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_sdl3.h)
  target_link_libraries(imgui_backend_sdl3 PUBLIC SDL3::SDL3)
  target_link_libraries(imgui_backend_sdl3 PUBLIC imgui_core)

  install(
    TARGETS imgui_backend_sdl3
    EXPORT imgui_core_export
    FILE_SET imgui_backend_sdl3_set)

  if(example_sdl3_opengl3)
    add_executable(example_sdl3_opengl3)
    target_sources(example_sdl3_opengl3 PRIVATE ${_imdir}/examples/example_sdl3_opengl3/main.cpp)
    target_link_libraries(example_sdl3_opengl3 PRIVATE imgui_backend_sdl3 imgui_backend_opengl3)
  endif()

  if(example_sdl3_vulkan)
    add_executable(example_sdl3_vulkan)
    target_sources(example_sdl3_vulkan PRIVATE ${_imdir}/examples/example_sdl3_vulkan/main.cpp)
    target_link_libraries(example_sdl3_vulkan PRIVATE imgui_backend_sdl3 imgui_backend_vulkan)
  endif()
endif()

if(win32)
   add_library(imgui_backend_win32)
   target_sources(imgui_backend_win32 PRIVATE ${_imdir}/backends/imgui_impl_win32.cpp)
   target_sources(imgui_backend_win32 PUBLIC FILE_SET imgui_backend_win32_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_win32.h)
   target_link_libraries(imgui_backend_win32 PUBLIC imgui_core)

   install(TARGETS imgui_backend_win32
    EXPORT imgui_core_export
    FILE_SET imgui_backend_win32_set)
endif()

if(example_null)
  add_executable(example_null)
  target_sources(example_null PRIVATE ${_imdir}/examples/example_null/main.cpp)
  target_link_libraries(example_null PRIVATE imgui_core)
endif()

if(visualize_targets)
  if(NOT EXISTS ${PROJECT_BINARY_DIR}/CMakeGraphVizOptions.cmake)
    file(
      WRITE ${PROJECT_BINARY_DIR}/CMakeGraphVizOptions.cmake
      "
set(GRAPHVIZ_GENERATE_PER_TARGET FALSE)
set(GRAPHVIZ_GENERATE_DEPENDERS FALSE)
")
  endif()

  find_program(DOT_COMMAND dot REQUIRED)

  add_custom_target(
    graph
    COMMAND ${CMAKE_COMMAND} --graphviz=targets.dot ${PROJECT_BINARY_DIR}
    COMMAND ${DOT_COMMAND} -Tpng targets.dot -o targets.png
    WORKING_DIRECTORY ${PROJECT_BINARY_DIR})
endif()
