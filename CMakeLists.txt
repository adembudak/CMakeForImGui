# SPDX-License-Identifier: MIT

cmake_minimum_required(VERSION 3.40.0)

project(CMakeForImGui CXX)

include(CMakeDependentOption)
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED 1)
set(CMAKE_CXX_EXTENSIONS 0)

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

option(android "" OFF)
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

option(imgui_club "" OFF)
cmake_dependent_option(imgui_memory_editor "" OFF imgui_club OFF)
cmake_dependent_option(imgui_multicontext_compositor "" OFF imgui_club OFF)
cmake_dependent_option(imgui_threaded_rendering "" OFF imgui_club OFF)

option(implot "" OFF)

option(BUILD_SHARED_LIBS "" OFF)

option(install "" ON)
option(uninstall "" ON)

option(visualize_targets "Visualize linkage graph" OFF)
mark_as_advanced(FORCE visualize_targets)

cmake_dependent_option(example_android_opengl3 "" OFF "examples AND android" OFF)

cmake_dependent_option(example_glfw_opengl2 "" OFF "examples AND glfw AND opengl2" OFF)
cmake_dependent_option(example_glfw_opengl3 "" OFF "examples AND glfw AND opengl3" OFF)
cmake_dependent_option(example_glfw_vulkan "" OFF "examples AND glfw AND vulkan" OFF)
cmake_dependent_option(example_glut_opengl2 "" OFF "examples AND glut AND opengl2" OFF)

cmake_dependent_option(example_sdl2_opengl2 "" OFF "examples AND sdl2 AND opengl2" OFF)
cmake_dependent_option(example_sdl2_opengl3 "" OFF "examples AND sdl2 AND opengl3" OFF)
cmake_dependent_option(example_sdl2_vulkan "" OFF "examples AND sdl2 AND vulkan" OFF)
cmake_dependent_option(example_sdl2_directx11 "" OFF "examples AND sdl2 AND dx11" OFF)

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

set(imgui_backend_list)

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
    target_compile_definitions(imgui_core PUBLIC IMGUI_ENABLE_FREETYPE)

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
    install(
      DIRECTORY ${_imdir}/misc/fonts
      DESTINATION ${CMAKE_INSTALL_DATADIR}/imgui
      FILES_MATCHING
      PATTERN "*.ttf")
  endif()
endif()

if(opengl2)
  find_package(OpenGL 2 REQUIRED)

  add_library(imgui_backend_opengl2)
  target_sources(imgui_backend_opengl2 PRIVATE ${_imdir}/backends/imgui_impl_opengl2.cpp)
  target_link_libraries(imgui_backend_opengl2 PUBLIC imgui_core)
  target_link_libraries(imgui_backend_opengl2 PUBLIC OpenGL::GL)
  target_sources(imgui_backend_opengl2 PUBLIC FILE_SET imgui_backend_opengl2_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_opengl2.h)

  list(APPEND imgui_backend_list imgui_backend_opengl2)
endif()

if(opengl3)
  include(FindOpenGL)

  add_library(imgui_backend_opengl3)
  target_sources(imgui_backend_opengl3 PRIVATE ${_imdir}/backends/imgui_impl_opengl3.cpp)
  target_link_libraries(imgui_backend_opengl3 PUBLIC imgui_core)
  target_link_libraries(imgui_backend_opengl3 PUBLIC $<IF:$<TARGET_EXISTS:imgui_backend_android>,EGL,OpenGL::GL>)
  target_sources(imgui_backend_opengl3 PUBLIC FILE_SET imgui_backend_opengl3_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_opengl3.h)

  list(APPEND imgui_backend_list imgui_backend_opengl3)
endif()

if(vulkan)
  find_package(Vulkan REQUIRED)

  add_library(imgui_backend_vulkan)
  target_sources(imgui_backend_vulkan PRIVATE ${_imdir}/backends/imgui_impl_vulkan.cpp)
  target_link_libraries(imgui_backend_vulkan PUBLIC imgui_core)
  target_link_libraries(imgui_backend_vulkan PUBLIC Vulkan::Vulkan)
  target_sources(imgui_backend_vulkan PUBLIC FILE_SET imgui_backend_vulkan_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_vulkan.h)

  list(APPEND imgui_backend_list imgui_backend_vulkan)
endif()

if(android)
  if(NOT DEFINED CMAKE_ANDROID_NDK AND NOT DEFINED ENV{ANDROID_NDK})
    message(FATAL_ERROR "Android NDK toolchain not found")
  endif()

  include(CMakePrintHelpers)
  cmake_print_variables(CMAKE_SYSTEM_NAME CMAKE_ANDROID_NDK CMAKE_TOOLCHAIN_FILE ANDROID_TOOLCHAIN ANDROID_TOOLCHAIN_NAME $ENV{ANDROID_HOME} $ENV{ANDROID_NDK} ANDROID_ABI ANDROID_PLATFORM ANDROID_STL ANDROID_PIE ANDROID_CPP_FEATURES ANDROID_ALLOW_UNDEFINED_SYMBOLS ANDROID_ARM_MODE ANDROID_NATIVE_API_LEVEL)

  if(CMAKE_ANDROID_ARCH_ABI STREQUAL "armeabi-v7a")
    set(triplet arm-linux-androideabi)
  elseif(CMAKE_ANDROID_ARCH_ABI STREQUAL "arm64-v8a")
    set(triplet aarch64-linux-android)
  elseif(CMAKE_ANDROID_ARCH_ABI STREQUAL "x86")
    set(triplet i686-linux-android)
  elseif(CMAKE_ANDROID_ARCH_ABI STREQUAL "x86_64")
    set(triplet x86_64-linux-android)
  elseif(CMAKE_ANDROID_ARCH_ABI STREQUAL "risv64")
    set(triplet riscv64-linux-android)
  else()
    set(triplet arm-linux-androideabi)
  endif()

  set(api_level ${ANDROID_NATIVE_API_LEVEL})

  set(Android_NDK_INCLUDE_DIR $ENV{ANDROID_NDK}/toolchains/llvm/prebuilt/windows-x86_64/sysroot/usr/include)
  set(Android_NDK_LIB_DIR $ENV{ANDROID_NDK}/toolchains/llvm/prebuilt/windows-x86_64/sysroot/usr/lib/${triplet}/${api_level})

  add_library(imgui_backend_android)
  target_sources(imgui_backend_android PRIVATE ${_imdir}/backends/imgui_impl_android.cpp)
  target_link_libraries(imgui_backend_android PUBLIC imgui_core)
  target_sources(imgui_backend_android PUBLIC FILE_SET imgui_backend_android_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_android.h)
  target_link_directories(imgui_backend_android PUBLIC ${Android_NDK_LIB_DIR})
  target_link_libraries(imgui_backend_android PUBLIC log android)
  target_sources(imgui_backend_android PRIVATE FILE_SET HEADERS BASE_DIRS ${Android_NDK_INCLUDE_DIR} FILES ${Android_NDK_INCLUDE_DIR}/android/native_window.h ${Android_NDK_INCLUDE_DIR}/android/input.h ${Android_NDK_INCLUDE_DIR}/android/keycodes.h ${Android_NDK_INCLUDE_DIR}/android/log.h)

  list(APPEND imgui_backend_list imgui_backend_android)

  if(example_android_opengl3)
    include(CMakePushCheckState)
    cmake_push_check_state(RESET)

    enable_language(C)
    add_library(android_native_app_glue STATIC)
    target_sources(android_native_app_glue PRIVATE $ENV{ANDROID_NDK}/sources/android/native_app_glue/android_native_app_glue.c)
    target_sources(android_native_app_glue PUBLIC FILE_SET HEADERS BASE_DIRS $ENV{ANDROID_NDK}/sources/android/native_app_glue FILES $ENV{ANDROID_NDK}/sources/android/native_app_glue/android_native_app_glue.h)
    set_target_properties(android_native_app_glue PROPERTIES LINKER_LANGUAGE C)

    cmake_pop_check_state()

    add_library(example_android_opengl3 SHARED)
    target_sources(example_android_opengl3 PRIVATE ${_imdir}/examples/example_android_opengl3/main.cpp)
    target_link_libraries(example_android_opengl3 PRIVATE imgui_backend_android imgui_backend_opengl3)
    target_link_libraries(example_android_opengl3 PRIVATE android_native_app_glue android EGL GLESv3 log)
    target_compile_definitions(example_android_opengl3 PRIVATE IMGUI_IMPL_OPENGL_ES3)
    target_link_options(example_android_opengl3 PRIVATE "-u ANativeActivity_onCreate")

    find_program(GRADLE_COMMAND gradle)

    if(DEFINED ENV{ANDROID_HOME} AND DEFINED ENV{ANDROID_NDK} AND GRADLE_COMMAND)
      add_custom_command(TARGET example_android_opengl3 POST_BUILD
        COMMAND ${GRADLE_COMMAND} --project-dir ${_imdir}/examples/example_android_opengl3/android --warning-mode=none :app:assemble
        WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
        COMMENT "Building .apk files\n${_imdir}/examples/example_android_opengl3/android/app/build/outputs/apk")
    endif()
  endif()
endif()

if(allegro5)
  find_package(Allegro CONFIG REQUIRED)
  add_library(imgui_backend_allegro5)
  target_sources(imgui_backend_allegro5 PRIVATE ${_imdir}/backends/imgui_impl_allegro5.cpp)
  target_link_libraries(imgui_backend_allegro5 PUBLIC imgui_core)
  target_link_libraries(imgui_backend_allegro5 PUBLIC Allegro::allegro Allegro::allegro_primitives)
  target_sources(imgui_backend_allegro5 PUBLIC FILE_SET imgui_backend_allegro5_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_allegro5.h)

  list(APPEND imgui_backend_list imgui_backend_allegro5)

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

  list(APPEND imgui_backend_list imgui_backend_glfw)

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

  list(APPEND imgui_backend_list imgui_backend_glut)

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
  target_link_libraries(imgui_backend_sdl2 PUBLIC $<IF:$<BOOL:${BUILD_SHARED_LIBS}>,SDL2::SDL2,SDL2::SDL2-static>)
  target_link_libraries(imgui_backend_sdl2 PUBLIC imgui_core)

  list(APPEND imgui_backend_list imgui_backend_sdl2)

  if(example_sdl2_opengl2)
    add_executable(example_sdl2_opengl2)
    target_sources(example_sdl2_opengl2 PRIVATE ${_imdir}/examples/example_sdl2_opengl2/main.cpp)
    target_link_libraries(example_sdl2_opengl2 PRIVATE $<$<PLATFORM_ID:Windows>:SDL2::SDL2main> imgui_backend_sdl2 imgui_backend_opengl2)
  endif()

  if(example_sdl2_opengl3)
    add_executable(example_sdl2_opengl3)
    target_sources(example_sdl2_opengl3 PRIVATE ${_imdir}/examples/example_sdl2_opengl3/main.cpp)
    target_link_libraries(example_sdl2_opengl3 PRIVATE $<$<PLATFORM_ID:Windows>:SDL2::SDL2main> imgui_backend_sdl2 imgui_backend_opengl3)
  endif()

  if(example_sdl2_vulkan)
    add_executable(example_sdl2_vulkan)
    target_sources(example_sdl2_vulkan PRIVATE ${_imdir}/examples/example_sdl2_vulkan/main.cpp)
    target_link_libraries(example_sdl2_vulkan PRIVATE $<$<PLATFORM_ID:Windows>:SDL2::SDL2main> imgui_backend_sdl2 imgui_backend_vulkan)
  endif()

  if(example_sdl2_directx11)
    add_executable(example_sdl2_directx11)
    target_sources(example_sdl2_directx11 PRIVATE ${_imdir}/examples/example_sdl2_directx11/main.cpp)
    target_link_libraries(example_sdl2_directx11 PRIVATE $<$<PLATFORM_ID:Windows>:SDL2::SDL2main> imgui_backend_sdl2 imgui_backend_dx11)
  endif()
endif()

if(sdlrenderer2)
  find_package(SDL2 CONFIG REQUIRED)
  add_library(imgui_backend_sdlrenderer2)
  target_sources(imgui_backend_sdlrenderer2 PRIVATE ${_imdir}/backends/imgui_impl_sdlrenderer2.cpp)
  target_sources(imgui_backend_sdlrenderer2 PUBLIC FILE_SET imgui_backend_sdlrenderer2_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES
                                                   ${_imdir}/backends/imgui_impl_sdlrenderer2.h)
  target_link_libraries(imgui_backend_sdlrenderer2 PUBLIC $<IF:$<BOOL:${BUILD_SHARED_LIBS}>,SDL2::SDL2,SDL2::SDL2-static>)
  target_link_libraries(imgui_backend_sdlrenderer2 PUBLIC imgui_core)

  list(APPEND imgui_backend_list imgui_backend_sdlrenderer2)

  if(example_sdl2_sdlrenderer2)
    add_executable(example_sdl2_sdlrenderer2)
    target_sources(example_sdl2_sdlrenderer2 PRIVATE ${_imdir}/examples/example_sdl2_sdlrenderer2/main.cpp)
    target_link_libraries(example_sdl2_sdlrenderer2 PRIVATE $<$<PLATFORM_ID:Windows>:SDL2::SDL2main> imgui_backend_sdlrenderer2 imgui_backend_sdl2)
  endif()
endif()

if(sdl3)
  find_package(SDL3 CONFIG REQUIRED)

  add_library(imgui_backend_sdl3)
  target_sources(imgui_backend_sdl3 PRIVATE ${_imdir}/backends/imgui_impl_sdl3.cpp)
  target_sources(imgui_backend_sdl3 PUBLIC FILE_SET imgui_backend_sdl3_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_sdl3.h)
  target_link_libraries(imgui_backend_sdl3 PUBLIC SDL3::SDL3)
  target_link_libraries(imgui_backend_sdl3 PUBLIC imgui_core)

  list(APPEND imgui_backend_list imgui_backend_sdl3)

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
  target_sources(imgui_backend_sdlgpu3 PUBLIC FILE_SET imgui_backend_sdlgpu3_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_sdlgpu3.h
                                              ${_imdir}/backends/imgui_impl_sdlgpu3_shaders.h)
  target_link_libraries(imgui_backend_sdlgpu3 PUBLIC SDL3::SDL3)
  target_link_libraries(imgui_backend_sdlgpu3 PUBLIC imgui_core)

  list(APPEND imgui_backend_list imgui_backend_sdlgpu3)

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
  target_sources(imgui_backend_sdlrenderer3 PUBLIC FILE_SET imgui_backend_sdlrenderer3_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES
                                                   ${_imdir}/backends/imgui_impl_sdlrenderer3.h)
  target_link_libraries(imgui_backend_sdlrenderer3 PUBLIC SDL3::SDL3)
  target_link_libraries(imgui_backend_sdlrenderer3 PUBLIC imgui_core imgui_backend_sdl3)

  list(APPEND imgui_backend_list imgui_backend_sdlrenderer3)

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

  list(APPEND imgui_backend_list imgui_backend_win32)

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

if(example_null)
  add_executable(example_null)
  target_sources(example_null PRIVATE ${_imdir}/examples/example_null/main.cpp)
  target_link_libraries(example_null PRIVATE imgui_core)
endif()

if(NOT WIN32 AND (win32 OR dx9 OR dx10 OR dx11 OR dx12))
  message(FATAL_ERROR "Win32 and DirectX targets are only available on Windows")
endif()

if(dx9 OR dx10 OR dx11 OR dx12)
  set(sdk_version ${CMAKE_VS_WINDOWS_TARGET_PLATFORM_VERSION})

  find_path(
    Windows_SDK_INCLUDE_DIR d3d9.h
    PATHS "[HKLM/Software/Microsoft/Windows Kits/Installed Roots;KitsRoot10]" "C:/Program Files (x86)/Windows Kits/10"
    PATH_SUFFIXES "Include/${sdk_version}/shared" NO_CACHE REQUIRED)

  find_path(
    Windows_SDK_INCLUDE_DIR_1 d3d10.h d3d10_1.h d3d11.h d3d12.h d3dcompiler.h
    PATHS "[HKLM/Software/Microsoft/Windows Kits/Installed Roots;KitsRoot10]" "C:/Program Files (x86)/Windows Kits/10"
    PATH_SUFFIXES "Include/${sdk_version}/um" NO_CACHE REQUIRED)

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

  find_path(
    Windows_SDK_LIB_DIR
    NAMES d3d9.lib d3d10.lib d3d10_1.lib d3d11.lib d3d12.lib d3dcompiler.lib
    PATHS "[HKLM/SOFTWARE/Microsoft/Windows Kits/Installed Roots;KitsRoot10]"
    PATH_SUFFIXES "Lib/${sdk_version}/um/${arch}" "C:/Program Files (x86)/Windows Kits/10/Lib/${sdk_version}/um/${arch}" REGISTRY_VIEW HOST NO_CACHE REQUIRED)
endif()

if(dx9)
  add_library(imgui_backend_dx9)
  target_sources(imgui_backend_dx9 PRIVATE ${_imdir}/backends/imgui_impl_dx9.cpp)
  target_sources(imgui_backend_dx9 PUBLIC FILE_SET imgui_backend_dx9_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_dx9.h)
  target_link_directories(imgui_backend_dx9 PRIVATE ${Windows_SDK_LIB_DIR})
  target_link_libraries(imgui_backend_dx9 PUBLIC d3d9.lib)
  target_link_libraries(imgui_backend_dx9 PUBLIC imgui_core)
  target_sources(imgui_backend_dx9 PRIVATE FILE_SET HEADERS BASE_DIRS ${Windows_SDK_INCLUDE_DIR} FILES ${Windows_SDK_INCLUDE_DIR}/d3d9.h)

  list(APPEND imgui_backend_list imgui_backend_dx9)
endif()

if(dx10)
  add_library(imgui_backend_dx10)
  target_sources(imgui_backend_dx10 PRIVATE ${_imdir}/backends/imgui_impl_dx10.cpp)
  target_sources(imgui_backend_dx10 PUBLIC FILE_SET imgui_backend_dx10_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_dx10.h)
  target_link_directories(imgui_backend_dx10 PRIVATE ${Windows_SDK_LIB_DIR})
  target_link_libraries(imgui_backend_dx10 PUBLIC d3d10_1.lib d3d10.lib d3dcompiler.lib)
  target_link_libraries(imgui_backend_dx10 PUBLIC imgui_core)
  target_sources(imgui_backend_dx10 PRIVATE FILE_SET HEADERS BASE_DIRS ${Windows_SDK_INCLUDE_DIR_1} FILES ${Windows_SDK_INCLUDE_DIR_1}/d3d10_1.h
                                            ${Windows_SDK_INCLUDE_DIR_1}/d3d10.h ${Windows_SDK_INCLUDE_DIR_1}/d3dcompiler.h)

  list(APPEND imgui_backend_list imgui_backend_dx10)
endif()

if(dx11)
  add_library(imgui_backend_dx11)
  target_sources(imgui_backend_dx11 PRIVATE ${_imdir}/backends/imgui_impl_dx11.cpp)
  target_sources(imgui_backend_dx11 PUBLIC FILE_SET imgui_backend_dx11_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_dx11.h)
  target_link_directories(imgui_backend_dx11 PRIVATE ${Windows_SDK_LIB_DIR})
  target_link_libraries(imgui_backend_dx11 PUBLIC d3d11.lib d3dcompiler.lib)
  target_link_libraries(imgui_backend_dx11 PUBLIC imgui_core)
  target_sources(imgui_backend_dx11 PRIVATE FILE_SET HEADERS BASE_DIRS ${Windows_SDK_INCLUDE_DIR_1} FILES ${Windows_SDK_INCLUDE_DIR_1}/d3d11.h
                                            ${Windows_SDK_INCLUDE_DIR_1}/d3dcompiler.h)

  list(APPEND imgui_backend_list imgui_backend_dx11)
endif()

if(dx12)
  add_library(imgui_backend_dx12)
  target_sources(imgui_backend_dx12 PRIVATE ${_imdir}/backends/imgui_impl_dx12.cpp)
  target_sources(imgui_backend_dx12 PUBLIC FILE_SET imgui_backend_dx12_set BASE_DIRS ${_imdir}/backends TYPE HEADERS FILES ${_imdir}/backends/imgui_impl_dx12.h)
  target_link_directories(imgui_backend_dx12 PRIVATE ${Windows_SDK_LIB_DIR})
  target_link_libraries(imgui_backend_dx12 PUBLIC d3d12.lib dxgi.lib d3dcompiler.lib)
  target_link_libraries(imgui_backend_dx12 PUBLIC imgui_core)
  target_sources(imgui_backend_dx12 PRIVATE FILE_SET HEADERS BASE_DIRS ${Windows_SDK_INCLUDE_DIR_1} FILES ${Windows_SDK_INCLUDE_DIR_1}/d3d12.h
                                            ${Windows_SDK_INCLUDE_DIR_1}/d3dcompiler.h)
  target_sources(imgui_backend_dx12 PRIVATE FILE_SET HEADERS BASE_DIRS ${Windows_SDK_INCLUDE_DIR} FILES ${Windows_SDK_INCLUDE_DIR}/dxgi1_4.h)

  list(APPEND imgui_backend_list imgui_backend_dx12)
endif()

if(imgui_club)
  if(NOT IMGUI_CLUB_SOURCE_DIR)
    message(FATAL_ERROR "set IMGUI_CLUB_SOURCE_DIR to proceed:\ne.g. cmake -DIMGUI_CLUB_SOURCE_DIR=path.to.imgui_club ...")
  endif()
  set(_imclub_dir ${IMGUI_CLUB_SOURCE_DIR})

  if(imgui_memory_editor)
    add_library(imgui_memory_editor INTERFACE)
    target_sources(imgui_memory_editor INTERFACE FILE_SET imgui_memory_editor_set BASE_DIRS ${_imclub_dir}/imgui_memory_editor TYPE HEADERS FILES ${_imclub_dir}/imgui_memory_editor/imgui_memory_editor.h)
  endif()

  if(imgui_multicontext_compositor)
    add_library(imgui_multicontext_compositor)
    target_sources(imgui_multicontext_compositor PRIVATE ${_imclub_dir}/imgui_multicontext_compositor/imgui_multicontext_compositor.cpp)
    target_sources(imgui_multicontext_compositor PUBLIC FILE_SET imgui_multicontext_compositor_set BASE_DIRS ${_imclub_dir}/imgui_multicontext_compositor TYPE HEADERS FILES ${_imclub_dir}/imgui_multicontext_compositor/imgui_multicontext_compositor.h)
    target_link_libraries(imgui_multicontext_compositor PUBLIC imgui_core)
 endif()

  if(imgui_threaded_rendering)
    add_library(imgui_threaded_rendering INTERFACE)
    target_sources(imgui_threaded_rendering INTERFACE FILE_SET imgui_threaded_rendering_set BASE_DIRS ${_imclub_dir}/imgui_threaded_rendering TYPE HEADERS FILES ${_imclub_dir}/imgui_threaded_rendering/imgui_threaded_rendering.h)
  endif()
endif()

if(implot)
  if(NOT IMPLOT_SOURCE_DIR)
    message(FATAL_ERROR "set IMPLOT_SOURCE_DIR to proceed:\ne.g. cmake -DIMPLOT_SOURCE_DIR=path.to.implot ...")
  endif()
  set(_implot_dir ${IMPLOT_SOURCE_DIR})

  add_library(implot)
  target_sources(implot PRIVATE ${_implot_dir}/implot.cpp ${_implot_dir}/implot_items.cpp)
  target_sources(implot PUBLIC FILE_SET implot_set BASE_DIRS ${_implot_dir} TYPE HEADERS FILES ${_implot_dir}/implot.h ${_implot_dir}/implot_internal.h)
  target_link_libraries(implot PUBLIC imgui_core)

  add_library(implot_demo)
  target_sources(implot_demo PRIVATE ${_implot_dir}/implot_demo.cpp)
  target_link_libraries(implot_demo PRIVATE implot)
endif()

if(install)
  install(
    DIRECTORY ${_imdir}/docs
    DESTINATION ${CMAKE_INSTALL_DATADIR}/imgui FILES_MATCHING PATTERN "docs/*")

  install(FILES ${_imdir}/LICENSE.txt DESTINATION ${CMAKE_INSTALL_DATADIR}/imgui)

  install(
    EXPORT imgui_export
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/CMakeForImGui
    NAMESPACE Unofficial::DearImGui::
    FILE CMakeForImGuiTargets.cmake)

  install(
    TARGETS imgui_core
    EXPORT imgui_export
    FILE_SET imgui_core_set)

  foreach(backend IN LISTS imgui_backend_list)
    get_property(name TARGET ${backend} PROPERTY NAME)
    get_property(file_set TARGET ${backend} PROPERTY HEADER_SETS)
    list(REMOVE_ITEM file_set HEADERS)

    install(TARGETS ${name}
	    EXPORT imgui_export
	    FILE_SET ${file_set})
  endforeach()

  if(imgui_club)
    install(
      EXPORT imgui_club_export
      DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/CMakeForImGui
      NAMESPACE Unofficial::imgui_club::
      FILE CMakeForImGui-imgui_club-Targets.cmake)

    install(TARGETS imgui_threaded_rendering EXPORT imgui_club_export FILE_SET imgui_threaded_rendering_set)
    install(TARGETS imgui_memory_editor EXPORT imgui_club_export FILE_SET imgui_memory_editor_set)
    install(TARGETS imgui_multicontext_compositor EXPORT imgui_club_export FILE_SET imgui_multicontext_compositor_set)
  endif()

  if(implot)
    install(
      EXPORT implot_export
      DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/CMakeForImGui
      NAMESPACE Unofficial::ImPlot::
      FILE CMakeForImGui-ImPlot-Targets.cmake)

    install(TARGETS implot implot_demo EXPORT implot_export FILE_SET implot_set)
  endif()

  configure_package_config_file(CMakeForImGuiConfig.cmake.in CMakeForImGuiConfig.cmake INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/ImGui)

  write_basic_package_version_file(
    ${PROJECT_BINARY_DIR}/CMakeForImGuiConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion)

  install(FILES ${PROJECT_BINARY_DIR}/CMakeForImGuiConfig.cmake ${PROJECT_BINARY_DIR}/CMakeForImGuiConfigVersion.cmake DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/CMakeForImGui)
endif()

if(uninstall)
  file(CONFIGURE
    OUTPUT cmake_uninstall.cmake
    CONTENT
    [=[
if(NOT EXISTS @PROJECT_BINARY_DIR@/install_manifest.txt)
  message(FATAL_ERROR "Couldn't find @PROJECT_BINARY_DIR@/install_manifest.txt")
endif()

file(STRINGS @PROJECT_BINARY_DIR@/install_manifest.txt files_to_uninstall)

foreach(file ${files_to_uninstall})
  execute_process(COMMAND @CMAKE_COMMAND@ -E rm $ENV{DESTDIR}${file} COMMAND_ECHO STDOUT)
endforeach()
]=]
    @ONLY)

  add_custom_target(
    uninstall
    COMMAND ${CMAKE_COMMAND} -P ${PROJECT_BINARY_DIR}/cmake_uninstall.cmake
    WORKING_DIRECTORY ${PROJECT_BINARY_DIR})
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
