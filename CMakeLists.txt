# SPDX-License-Identifier: MIT

cmake_minimum_required(VERSION 3.40.0)

project(CMakeForImGui CXX)

include(CMakeDependentOption)
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED 1)

if(NOT IMGUI_SOURCE_DIR)
  message(FATAL_ERROR "set IMGUI_SOURCE_DIR to proceed:\ne.g. cmake -DIMGUI_SOURCE_DIR=path.to.imgui ...")
endif()

set(_imdir ${IMGUI_SOURCE_DIR})

file(STRINGS ${_imdir}/imgui.h imgui_version_string REGEX "IMGUI_VERSION[ \t]+\"[0-9]+\\.[0-9]+\\.[0-9]+")
string(REGEX MATCH "[0-9]+\\.[0-9]+\\.[0-9]+" imgui_version ${imgui_version_string})

set(PROJECT_VERSION ${imgui_version})

option(with_miscellaneous "" OFF)
cmake_dependent_option(misc_cpp "" OFF "with_miscellaneous" OFF)
cmake_dependent_option(misc_freetype "" OFF "with_miscellaneous" OFF)
cmake_dependent_option(misc_debugger "" OFF "with_miscellaneous" OFF)
cmake_dependent_option(misc_fonts "" OFF "with_miscellaneous" OFF)

option(allegro5 "" OFF)
option(glfw "" OFF)
option(glut "" OFF)
option(sdl2 "" OFF)
option(sdlrenderer2 "" OFF)
option(sdl3 "" OFF)
option(sdlgpu3 "" OFF)
option(sdlrenderer3 "" OFF)
option(opengl2 "" OFF)
option(opengl3 "" OFF)
option(vulkan "" OFF)
option(examples "" OFF)
option(win32 "" OFF)
option(dx9 "" OFF)
option(dx10 "" OFF)
option(dx11 "" OFF)
option(dx12 "" OFF)

option(docs "" OFF)

option(uninstall "" ON)

option(visualize_targets "Visualize linkage graph" OFF)
mark_as_advanced(FORCE visualize_targets)

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

cmake_dependent_option(example_sdl3_sdlgpu3 "" OFF "examples AND sdlgpu3" OFF)

cmake_dependent_option(example_sdl3_sdlrenderer3 "" OFF "examples AND sdlrenderer3 AND sdl3" OFF)

cmake_dependent_option(example_win32_opengl3 "" OFF "examples AND win32 AND opengl3" OFF)
cmake_dependent_option(example_win32_vulkan "" OFF "examples AND win32 AND vulkan" OFF)
cmake_dependent_option(example_win32_directx9 "" OFF "examples AND win32 AND dx9" OFF)
cmake_dependent_option(example_win32_directx10 "" OFF "examples AND win32 AND dx10" OFF)
cmake_dependent_option(example_win32_directx11 "" OFF "examples AND win32 AND dx11" OFF)
cmake_dependent_option(example_win32_directx12 "" OFF "examples AND win32 AND dx12" OFF)

cmake_dependent_option(example_null "" OFF examples OFF)

add_library(imgui_core)
target_sources(imgui_core PRIVATE ${_imdir}/imgui.cpp ${_imdir}/imgui_demo.cpp ${_imdir}/imgui_draw.cpp ${_imdir}/imgui_tables.cpp ${_imdir}/imgui_widgets.cpp)
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
  EXPORT imgui_export
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/CMakeForImGui
  NAMESPACE Unofficial::DearImGui::
  FILE CMakeForImGuiTargets.cmake)

install(
  TARGETS imgui_core
  EXPORT imgui_export
  FILE_SET imgui_core_set)

install(FILES ${_imdir}/LICENSE.txt DESTINATION ${CMAKE_INSTALL_DATADIR}/imgui)

if(with_miscellaneous)
  if(misc_cpp)
    add_library(imgui_misc_cpp)
    target_sources(imgui_misc_cpp PRIVATE ${_imdir}/misc/cpp/imgui_stdlib.cpp)
    target_sources(imgui_misc_cpp PRIVATE FILE_SET HEADERS BASE_DIRS ${_imdir} FILES ${_imdir}/imgui.h)
    target_link_libraries(imgui_core PUBLIC imgui_misc_cpp)
    target_sources(imgui_misc_cpp PUBLIC FILE_SET imgui_misc_cpp_set BASE_DIRS ${_imdir}/misc/cpp TYPE HEADERS FILES ${_imdir}/misc/cpp/imgui_stdlib.h)

    install(
      TARGETS imgui_misc_cpp
      EXPORT imgui_export
      FILE_SET imgui_misc_cpp_set)
  endif()

  if(misc_freetype)
    find_package(freetype CONFIG REQUIRED)

    add_library(imgui_misc_freetype)
    target_sources(imgui_misc_freetype PRIVATE ${_imdir}/misc/freetype/imgui_freetype.cpp)
    target_sources(imgui_misc_freetype PRIVATE FILE_SET HEADERS BASE_DIRS ${_imdir} FILES ${_imdir}/imgui.h)
    target_link_libraries(imgui_misc_freetype PRIVATE Freetype::Freetype)
    target_link_libraries(imgui_core PUBLIC imgui_misc_freetype)
    target_sources(imgui_misc_freetype PUBLIC FILE_SET imgui_misc_freetype_set BASE_DIRS ${_imdir}/misc/freetype TYPE HEADERS FILES ${_imdir}/misc/freetype/imgui_freetype.h)

    install(
      TARGETS imgui_misc_freetype
      EXPORT imgui_export
      FILE_SET imgui_misc_freetype_set)
  endif()

  if(misc_debugger)
   if(UNIX)
     install(FILES ${_imdir}/misc/debuggers/imgui.gdb DESTINATION ${CMAKE_INSTALL_DATADIR}/imgui/debugger)
    endif()
    if(WIN32)
      install(FILES ${_imdir}/misc/debuggers/imgui.natvis ${_imdir}/misc/debuggers/imgui.natstepfilter DESTINATION ${CMAKE_INSTALL_DATADIR}/imgui/debugger)
    endif()
  endif()

  if(misc_fonts)
    add_executable(binary_to_compressed_c)
    target_sources(binary_to_compressed_c PRIVATE ${_imdir}/misc/fonts/binary_to_compressed_c.cpp)

    install(TARGETS binary_to_compressed_c DESTINATION ${CMAKE_INSTALL_DATADIR}/imgui/fonts)
    install(DIRECTORY ${_imdir}/misc/fonts DESTINATION ${CMAKE_INSTALL_DATADIR}/imgui FILES_MATCHING PATTERN "*.ttf")
  endif()
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
    EXPORT imgui_export
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
    EXPORT imgui_export
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
    EXPORT imgui_export
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
    EXPORT imgui_export
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
    EXPORT imgui_export
    FILE_SET imgui_backend_glfw_set)

  if(example_glfw_opengl3)
    add_executable(example_glfw_opengl3)
    target_sources(example_glfw_opengl3 PRIVATE ${_imdir}/examples/example_glfw_opengl3/main.cpp)
    target_link_libraries(example_glfw_opengl3 PRIVATE imgui_backend_glfw)
  endif()

  if(example_glfw_opengl2)
    add_executable(example_glfw_opengl2)
    target_sources(example_glfw_opengl2 PRIVATE ${_imdir}/examples/example_glfw_opengl2/main.cpp)
    target_link_libraries(example_glfw_opengl2 PRIVATE imgui_backend_glfw imgui_backend_opengl2)
  endif()

  if(example_glfw_vulkan)
    add_executable(example_glfw_vulkan)
    target_sources(example_glfw_vulkan PRIVATE ${_imdir}/examples/example_glfw_vulkan/main.cpp)
    target_link_libraries(example_glfw_vulkan PRIVATE imgui_backend_glfw imgui_backend_vulkan)
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
    EXPORT imgui_export
    FILE_SET imgui_backend_glut_set)

  if(example_glut_opengl2)
    add_executable(example_glut_opengl2)
    target_sources(example_glut_opengl2 PRIVATE ${_imdir}/examples/example_glut_opengl2/main.cpp)
    target_link_libraries(example_glut_opengl2 PRIVATE imgui_backend_glut imgui_backend_opengl2)
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
    EXPORT imgui_export
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
    EXPORT imgui_export
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
    EXPORT imgui_export
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

if(sdlgpu3)
  find_package(SDL3 CONFIG REQUIRED)

  add_library(imgui_backend_sdlgpu3)
  target_sources(imgui_backend_sdlgpu3 PRIVATE ${_imdir}/backends/imgui_impl_sdlgpu3.cpp)
  target_sources(imgui_backend_sdlgpu3 PUBLIC FILE_SET imgui_backend_sdlgpu3_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_sdlgpu3.h ${_imdir}/backends/imgui_impl_sdlgpu3_shaders.h)
  target_link_libraries(imgui_backend_sdlgpu3 PUBLIC SDL3::SDL3)
  target_link_libraries(imgui_backend_sdlgpu3 PUBLIC imgui_core)

  install(
    TARGETS imgui_backend_sdlgpu3
    EXPORT imgui_export
    FILE_SET imgui_backend_sdlgpu3_set)

  if(example_sdl3_sdlgpu3)
    add_executable(example_sdl3_sdlgpu3)
    target_sources(example_sdl3_sdlgpu3 PRIVATE ${_imdir}/examples/example_sdl3_sdlgpu3/main.cpp)
    target_link_libraries(example_sdl3_sdlgpu3 PRIVATE imgui_backend_sdlgpu3 imgui_backend_sdl3)
  endif()
endif()

if(sdlrenderer3)
  find_package(SDL3 CONFIG REQUIRED)

  add_library(imgui_backend_sdlrenderer3)
  target_sources(imgui_backend_sdlrenderer3 PRIVATE ${_imdir}/backends/imgui_impl_sdlrenderer3.cpp)
  target_sources(imgui_backend_sdlrenderer3 PUBLIC FILE_SET imgui_backend_sdlrenderer3_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_sdlrenderer3.h)

  target_link_libraries(imgui_backend_sdlrenderer3 PUBLIC SDL3::SDL3)
  target_link_libraries(imgui_backend_sdlrenderer3 PUBLIC imgui_core imgui_backend_sdl3)

  install(
    TARGETS imgui_backend_sdlrenderer3
    EXPORT imgui_export
    FILE_SET imgui_backend_sdlrenderer3_set)

  if(example_sdl3_sdlrenderer3)
    add_executable(example_sdl3_sdlrenderer3)
    target_sources(example_sdl3_sdlrenderer3 PRIVATE ${_imdir}/examples/example_sdl3_sdlrenderer3/main.cpp)
    target_link_libraries(example_sdl3_sdlrenderer3 PRIVATE imgui_backend_sdlrenderer3 imgui_backend_sdl3)
  endif()
endif()

if(win32)
   add_library(imgui_backend_win32)
   target_sources(imgui_backend_win32 PRIVATE ${_imdir}/backends/imgui_impl_win32.cpp)
   target_sources(imgui_backend_win32 PUBLIC FILE_SET imgui_backend_win32_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_win32.h)
   target_link_libraries(imgui_backend_win32 PUBLIC imgui_core)

   install(TARGETS imgui_backend_win32
    EXPORT imgui_export
    FILE_SET imgui_backend_win32_set)

   if(example_win32_opengl3)
     add_executable(example_win32_opengl3)
     target_sources(example_win32_opengl3 PRIVATE ${_imdir}/examples/example_win32_opengl3/main.cpp)
     target_link_libraries(example_win32_opengl3 PRIVATE imgui_backend_win32 imgui_backend_opengl3)
   endif()

   if(example_win32_vulkan)
     add_executable(example_win32_vulkan)
     target_sources(example_win32_vulkan PRIVATE ${_imdir}/examples/example_win32_vulkan/main.cpp)
     target_link_libraries(example_win32_vulkan PRIVATE imgui_backend_win32 imgui_backend_vulkan)
   endif()

   if(example_win32_directx9)
     add_executable(example_win32_directx9)
     target_sources(example_win32_directx9 PRIVATE ${_imdir}/examples/example_win32_directx9/main.cpp)
     target_link_libraries(example_win32_directx9 PRIVATE imgui_backend_win32 imgui_backend_dx9)
   endif()

  if(example_win32_directx10)
    add_executable(example_win32_directx10)
    target_sources(example_win32_directx10 PRIVATE ${_imdir}/examples/example_win32_directx10/main.cpp)
    target_link_libraries(example_win32_directx10 PRIVATE imgui_backend_win32 imgui_backend_dx10)
  endif()

  if(example_win32_directx11)
    add_executable(example_win32_directx11)
    target_sources(example_win32_directx11 PRIVATE ${_imdir}/examples/example_win32_directx11/main.cpp)
    target_link_libraries(example_win32_directx11 PRIVATE imgui_backend_win32 imgui_backend_dx11)
  endif()

  if(example_win32_directx12)
    add_executable(example_win32_directx12)
    target_sources(example_win32_directx12 PRIVATE ${_imdir}/examples/example_win32_directx12/main.cpp)
    target_link_libraries(example_win32_directx12 PRIVATE imgui_backend_win32 imgui_backend_dx12)
  endif()
endif()

if(NOT WIN32 AND (win32 OR dx9 OR dx10 OR dx11 OR dx12))
  message(FATAL_ERROR "Win32 and DirectX targets are only available on Windows")
endif()

if(dx9 OR dx10 OR dx11 OR dx12)
  set(sdk_version ${CMAKE_VS_WINDOWS_TARGET_PLATFORM_VERSION})

  find_path(Windows_SDK_INCLUDE_DIR d3d9.h PATHS "[HKLM/Software/Microsoft/Windows Kits/Installed Roots;KitsRoot10]" "C:/Program Files (x86)/Windows Kits/10"
    PATH_SUFFIXES "Include/${sdk_version}/shared"
    NO_CACHE REQUIRED)

  find_path(Windows_SDK_INCLUDE_DIR_1 d3d10.h d3d10_1.h d3d11.h d3d12.h d3dcompiler.h
    PATHS "[HKLM/Software/Microsoft/Windows Kits/Installed Roots;KitsRoot10]" "C:/Program Files (x86)/Windows Kits/10"
    PATH_SUFFIXES "Include/${sdk_version}/um"
    NO_CACHE REQUIRED)

  set(_arch ${CMAKE_VS_PLATFORM_NAME})
  if("${_arch}" STREQUAL "Win32")
    set(arch x86)
  elseif("${_arch}" STREQUAL "x64")
    set(arch x64)
  elseif("${_arch}" STREQUAL "ARM")
    # ARM32 targets available SDKs before version 10.0.26100.0
    # https://github.com/microsoft/WindowsAppSDK/discussions/406
    if(${sdk_version} VERSION_LESS 10.0.26100.0)
      set(arch arm)
    endif()
  elseif("${_arch}" STREQUAL "ARM64")
    set(arch arm64)
  endif()

  if(NOT arch)
    message(FATAL_ERROR "-- Unsupported architecture: ${_arch}")
  endif()

  find_path(Windows_SDK_LIB_DIR NAMES d3d9.lib d3d10.lib d3d10_1.lib d3d11.lib d3d12.lib d3dcompiler.lib
    PATHS "[HKLM/SOFTWARE/Microsoft/Windows Kits/Installed Roots;KitsRoot10]" PATH_SUFFIXES "Lib/${sdk_version}/um/${arch}"
          "C:/Program Files (x86)/Windows Kits/10/Lib/${sdk_version}/um/${arch}"
    REGISTRY_VIEW HOST NO_CACHE REQUIRED)
endif()

if(dx9)
  add_library(imgui_backend_dx9)
  target_sources(imgui_backend_dx9 PRIVATE ${_imdir}/backends/imgui_impl_dx9.cpp)
  target_sources(imgui_backend_dx9 PUBLIC FILE_SET imgui_backend_dx9_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_dx9.h)
  target_link_directories(imgui_backend_dx9 PRIVATE ${Windows_SDK_LIB_DIR})
  target_link_libraries(imgui_backend_dx9 PUBLIC d3d9.lib)
  target_link_libraries(imgui_backend_dx9 PUBLIC imgui_core)
  target_sources(imgui_backend_dx9 PRIVATE FILE_SET HEADERS BASE_DIRS ${Windows_SDK_INCLUDE_DIR} FILES ${Windows_SDK_INCLUDE_DIR}/d3d9.h)

  install(TARGETS imgui_backend_dx9
    EXPORT imgui_export
    FILE_SET imgui_backend_dx9_set)
endif()

if(dx10)
  add_library(imgui_backend_dx10)
  target_sources(imgui_backend_dx10 PRIVATE ${_imdir}/backends/imgui_impl_dx10.cpp)
  target_sources(imgui_backend_dx10 PUBLIC FILE_SET imgui_backend_dx10_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_dx10.h)
  target_link_directories(imgui_backend_dx10 PRIVATE ${Windows_SDK_LIB_DIR})
  target_link_libraries(imgui_backend_dx10 PUBLIC d3d10_1.lib d3d10.lib d3dcompiler.lib)
  target_link_libraries(imgui_backend_dx10 PUBLIC imgui_core)
  target_sources(imgui_backend_dx10 PRIVATE FILE_SET HEADERS BASE_DIRS ${Windows_SDK_INCLUDE_DIR_1} FILES ${Windows_SDK_INCLUDE_DIR_1}/d3d10_1.h ${Windows_SDK_INCLUDE_DIR_1}/d3d10.h ${Windows_SDK_INCLUDE_DIR_1}/d3dcompiler.h)

  install(TARGETS imgui_backend_dx10
    EXPORT imgui_export
    FILE_SET imgui_backend_dx10_set)
endif()

if(dx11)
  add_library(imgui_backend_dx11)
  target_sources(imgui_backend_dx11 PRIVATE ${_imdir}/backends/imgui_impl_dx11.cpp)
  target_sources(imgui_backend_dx11 PUBLIC FILE_SET imgui_backend_dx11_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_dx11.h)
  target_link_directories(imgui_backend_dx11 PRIVATE ${Windows_SDK_LIB_DIR})
  target_link_libraries(imgui_backend_dx11 PUBLIC d3d11.lib d3dcompiler.lib)
  target_link_libraries(imgui_backend_dx11 PUBLIC imgui_core)
  target_sources(imgui_backend_dx11 PRIVATE FILE_SET HEADERS BASE_DIRS ${Windows_SDK_INCLUDE_DIR_1} FILES ${Windows_SDK_INCLUDE_DIR_1}/d3d11.h ${Windows_SDK_INCLUDE_DIR_1}/d3dcompiler.h)

  install(TARGETS imgui_backend_dx11
    EXPORT imgui_export
    FILE_SET imgui_backend_dx11_set)
endif()

if(dx12)
  add_library(imgui_backend_dx12)
  target_sources(imgui_backend_dx12 PRIVATE ${_imdir}/backends/imgui_impl_dx12.cpp)
  target_sources(imgui_backend_dx12 PUBLIC FILE_SET imgui_backend_dx12_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_dx12.h)
  target_link_directories(imgui_backend_dx12 PRIVATE ${Windows_SDK_LIB_DIR})
  target_link_libraries(imgui_backend_dx12 PUBLIC d3d12.lib dxgi.lib d3dcompiler.lib)
  target_link_libraries(imgui_backend_dx12 PUBLIC imgui_core)
  target_sources(imgui_backend_dx12 PRIVATE FILE_SET HEADERS BASE_DIRS ${Windows_SDK_INCLUDE_DIR_1} FILES ${Windows_SDK_INCLUDE_DIR_1}/d3d12.h ${Windows_SDK_INCLUDE_DIR_1}/d3dcompiler.h)
  target_sources(imgui_backend_dx12 PRIVATE FILE_SET HEADERS BASE_DIRS ${Windows_SDK_INCLUDE_DIR} FILES ${Windows_SDK_INCLUDE_DIR}/dxgi1_4.h)

  install(TARGETS imgui_backend_dx12
    EXPORT imgui_export
    FILE_SET imgui_backend_dx12_set)
endif()

if(docs)
  install(DIRECTORY ${_imdir}/docs DESTINATION ${CMAKE_INSTALL_DATADIR}/imgui FILES_MATCHING PATTERN "docs/*")
endif()

if(uninstall)
  file(CONFIGURE OUTPUT cmake_uninstall.cmake
       CONTENT
[=[
if(NOT EXISTS @PROJECT_BINARY_DIR@/install_manifest.txt)
  message(FATAL_ERROR "Couldn't find @PROJECT_BINARY_DIR@/install_manifest.txt")
endif()

file(STRINGS @PROJECT_BINARY_DIR@/install_manifest.txt files_to_uninstall)

foreach(file ${files_to_uninstall})
  execute_process(COMMAND @CMAKE_COMMAND@ -E rm $ENV{DESTDIR}${file} COMMAND_ECHO STDOUT)
endforeach()
]=] @ONLY)

  add_custom_target(uninstall
    COMMAND ${CMAKE_COMMAND} -P ${PROJECT_BINARY_DIR}/cmake_uninstall.cmake
    WORKING_DIRECTORY ${PROJECT_BINARY_DIR})
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

configure_package_config_file(
  CMakeForImGuiConfig.cmake.in
  CMakeForImGuiConfig.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/ImGui)

write_basic_package_version_file(
  ${PROJECT_BINARY_DIR}/CMakeForImGuiConfigVersion.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY AnyNewerVersion)

install(FILES
  ${PROJECT_BINARY_DIR}/CMakeForImGuiConfig.cmake
  ${PROJECT_BINARY_DIR}/CMakeForImGuiConfigVersion.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/CMakeForImGui)

