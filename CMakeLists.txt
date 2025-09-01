# SPDX-License-Identifier: MIT

cmake_minimum_required(VERSION 4.0.0)

project(CMakeForImGui
  VERSION 0.1.0.0
  DESCRIPTION "CMake build support for Dear ImGui"
  HOMEPAGE_URL "https://github.com/adembudak/CMakeForImGui"
  LANGUAGES CXX)

include(CMakeDependentOption)
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)
include(CMakePrintHelpers)
include(FeatureSummary)

if(NOT IMGUI_SOURCE_DIR)
  message(FATAL_ERROR "set IMGUI_SOURCE_DIR to proceed:\ne.g. cmake -DIMGUI_SOURCE_DIR=path.to.imgui ...")
endif()

cmake_path(SET _imgui_dir ${IMGUI_SOURCE_DIR})
cmake_path(APPEND _imgui_backends_dir ${_imgui_dir} backends)
cmake_path(APPEND _imgui_examples_dir ${_imgui_dir} examples)
cmake_path(APPEND _imgui_misc_dir ${_imgui_dir} misc)
cmake_path(APPEND _imgui_docs_dir ${_imgui_dir} docs)

file(STRINGS ${_imgui_dir}/imgui.h imgui_version_string REGEX "#define[ \t]+IMGUI_VERSION_NUM[ \t]+[0-9]+")
string(REGEX MATCH "[0-9]+" imgui_version ${imgui_version_string})

math(EXPR IMGUI_VERSION_MAJOR "(${imgui_version} / 10000)")
math(EXPR IMGUI_VERSION_MINOR "(${imgui_version} / 100) % 100")
math(EXPR IMGUI_VERSION_PATCH "(${imgui_version} / 10) % 10")
math(EXPR IMGUI_VERSION_TWEAK "(${imgui_version} % 10)")

set(IMGUI_VERSION "${IMGUI_VERSION_MAJOR}.${IMGUI_VERSION_MINOR}.${IMGUI_VERSION_PATCH}.${IMGUI_VERSION_TWEAK}")
message(STATUS "ImGui version ${IMGUI_VERSION}")

get_property(is_generator_multi_config GLOBAL PROPERTY GENERATOR_IS_MULTI_CONFIG)
if(NOT is_generator_multi_config)
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS Debug Release RelWithDebInfo MinSizeRel)
endif()

option(misc "" OFF)
cmake_dependent_option(misc_cpp "" OFF misc OFF)
cmake_dependent_option(misc_freetype "" OFF misc OFF)
cmake_dependent_option(misc_debugger "" OFF "misc AND (${IMGUI_VERSION} VERSION_GREATER 1.81)" OFF)
cmake_dependent_option(misc_fonts "" OFF misc OFF)

if(CMAKE_SYSTEM_NAME STREQUAL "Android")
  option(DearImGui_Backend_Android "" OFF)
endif()

option(DearImGui_Backend_Allegro5 "" OFF)
option(DearImGui_Backend_GLFW "" OFF)
option(DearImGui_Backend_FreeGLUT "" OFF)
option(DearImGui_Backend_SDL2 "" OFF)
option(DearImGui_Backend_SDLRenderer2 "" OFF)
option(DearImGui_Backend_SDL3 "" OFF)
option(DearImGui_Backend_SDLGPU3 "" OFF)
option(DearImGui_Backend_SDLRenderer3 "" OFF)
option(DearImGui_Backend_OpenGL2 "" OFF)
option(DearImGui_Backend_OpenGL3 "" OFF)
option(DearImGui_Backend_Vulkan "" OFF)
option(DearImGui_Backend_WebGPU "" OFF)
option(DearImGui_Examples "" OFF)

if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
  option(DearImGui_Backend_Win32 "" OFF)
  option(DearImGui_Backend_DirectX9 "" OFF)
  option(DearImGui_Backend_DirectX10 "" OFF)
  option(DearImGui_Backend_DirectX11 "" OFF)
  option(DearImGui_Backend_DirectX12 "" OFF)
endif()

if(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
  option(DearImGui_Backend_Metal "" OFF)
  option(DearImGui_Backend_OSX "" OFF)
endif()

option(thirdparty "" OFF)
cmake_dependent_option(imgui_test_engine "" OFF "thirdparty AND (${IMGUI_VERSION} VERSION_GREATER_EQUAL 1.89)" OFF)
cmake_dependent_option(imgui_club "" OFF thirdparty OFF)
cmake_dependent_option(imgui_markdown "" OFF thirdparty OFF)
cmake_dependent_option(implot "" OFF "thirdparty AND (${IMGUI_VERSION} VERSION_GREATER 1.89.0)" OFF)
cmake_dependent_option(implot3d "" OFF "thirdparty AND (${IMGUI_VERSION} VERSION_GREATER 1.90.0)" OFF)
cmake_dependent_option(imguifiledialog "" OFF "thirdparty AND (${IMGUI_VERSION} VERSION_GREATER 1.92.0)" OFF)

cmake_dependent_option(imgui_memory_editor "" OFF imgui_club OFF)
cmake_dependent_option(imgui_multicontext_compositor "" OFF "imgui_club AND (${IMGUI_VERSION} VERSION_GREATER 1.90.8)" OFF)
cmake_dependent_option(imgui_threaded_rendering "" OFF imgui_club OFF)

option(BUILD_SHARED_LIBS "" OFF)

option(pkg-config "Generate cmakeforimgui.pc file" OFF)

option(install "" ON)
option(uninstall "" ON)
cmake_dependent_option(install_examples "" OFF install OFF)

option(visualize_targets "Visualize linkage graph" OFF)
mark_as_advanced(FORCE visualize_targets)

cmake_dependent_option(DearImGui_Backend_SDLRenderer2 "" OFF "(${IMGUI_VERSION} VERSION_GREATER 1.84.2)" OFF)
cmake_dependent_option(DearImGui_Backend_SDL3 "" OFF "(${IMGUI_VERSION} VERSION_GREATER 1.91.0)" OFF)
cmake_dependent_option(DearImGui_Backend_SDLRenderer3 "" OFF "(${IMGUI_VERSION} VERSION_GREATER 1.91.0)" OFF)
cmake_dependent_option(DearImGui_Backend_SDLGPU3 "" OFF "(${IMGUI_VERSION} VERSION_GREATER 1.91.6)" OFF)
cmake_dependent_option(DearImGui_Backend_WebGPU "" OFF "(${IMGUI_VERSION} VERSION_GREATER 1.80.0)" OFF)

cmake_dependent_option(example_android_opengl3 "" OFF "DearImGui_Examples AND android" OFF)

cmake_dependent_option(example_glfw_opengl2 "" OFF "DearImGui_Examples AND DearImGui_Backend_GLFW AND DearImGui_Backend_OpenGL2" OFF)
cmake_dependent_option(example_glfw_opengl3 "" OFF "DearImGui_Examples AND DearImGui_Backend_GLFW AND DearImGui_Backend_OpenGL3" OFF)
cmake_dependent_option(example_glfw_vulkan "" OFF "DearImGui_Examples AND DearImGui_Backend_GLFW AND DearImGui_Backend_Vulkan" OFF)
cmake_dependent_option(example_glfw_metal "" OFF "DearImGui_Examples AND DearImGui_Backend_GLFW AND DearImGui_Backend_Metal" OFF)
cmake_dependent_option(example_glfw_wgpu "" OFF "DearImGui_Examples AND DearImGui_Backend_GLFW AND DearImGui_Backend_WebGPU" OFF)

cmake_dependent_option(example_glut_opengl2 "" OFF "DearImGui_Examples AND DearImGui_Backend_FreeGLUT AND DearImGui_Backend_OpenGL2" OFF)

cmake_dependent_option(example_sdl2_opengl2 "" OFF "DearImGui_Examples AND DearImGui_Backend_SDL2 AND DearImGui_Backend_OpenGL2" OFF)
cmake_dependent_option(example_sdl2_opengl3 "" OFF "DearImGui_Examples AND DearImGui_Backend_SDL2 AND DearImGui_Backend_OpenGL3" OFF)
cmake_dependent_option(example_sdl2_vulkan "" OFF "DearImGui_Examples AND DearImGui_Backend_SDL2 AND DearImGui_Backend_Vulkan" OFF)
cmake_dependent_option(example_sdl2_directx11 "" OFF "DearImGui_Examples AND DearImGui_Backend_SDL2 AND DearImGui_Backend_DirectX11" OFF)
cmake_dependent_option(example_sdl2_metal "" OFF "DearImGui_Examples AND DearImGui_Backend_SDL2 AND DearImGui_Backend_Metal" OFF)

cmake_dependent_option(example_sdl2_sdlrenderer2 "" OFF "DearImGui_Examples AND DearImGui_Backend_SDLRenderer2 AND DearImGui_Backend_SDL2" OFF)

cmake_dependent_option(example_allegro5 "" OFF "DearImGui_Examples AND DearImGui_Backend_Allegro5" OFF)

cmake_dependent_option(example_sdl3_opengl3 "" OFF "DearImGui_Examples AND DearImGui_Backend_SDL3 AND DearImGui_Backend_OpenGL3" OFF)
cmake_dependent_option(example_sdl3_vulkan "" OFF "DearImGui_Examples AND DearImGui_Backend_SDL3 AND DearImGui_Backend_Vulkan AND (${IMGUI_VERSION} VERSION_GREATER 1.91.4)" OFF)
cmake_dependent_option(example_sdl3_metal "" OFF "DearImGui_Examples AND DearImGui_Backend_SDL3 AND DearImGui_Backend_Metal AND (${IMGUI_VERSION} VERSION_GREATER 1.92.1)" OFF)

cmake_dependent_option(example_sdl3_sdlgpu3 "" OFF "DearImGui_Examples AND DearImGui_Backend_SDLGPU3" OFF)

cmake_dependent_option(example_sdl3_sdlrenderer3 "" OFF "DearImGui_Examples AND DearImGui_Backend_SDLRenderer3 AND DearImGui_Backend_SDL3" OFF)

cmake_dependent_option(example_win32_opengl3 "" OFF "DearImGui_Examples AND DearImGui_Backend_Win32 AND DearImGui_Backend_OpenGL3 AND (${IMGUI_VERSION} VERSION_GREATER 1.89.5)" OFF)
cmake_dependent_option(example_win32_vulkan "" OFF "DearImGui_Examples AND DearImGui_Backend_Win32 AND DearImGui_Backend_Vulkan AND (${IMGUI_VERSION} VERSION_GREATER 1.91.6)" OFF)
cmake_dependent_option(example_win32_directx9 "" OFF "DearImGui_Examples AND DearImGui_Backend_Win32 AND DearImGui_Backend_DirectX9" OFF)
cmake_dependent_option(example_win32_directx10 "" OFF "DearImGui_Examples AND DearImGui_Backend_Win32 AND DearImGui_Backend_DirectX10" OFF)
cmake_dependent_option(example_win32_directx11 "" OFF "DearImGui_Examples AND DearImGui_Backend_Win32 AND DearImGui_Backend_DirectX11" OFF)
cmake_dependent_option(example_win32_directx12 "" OFF "DearImGui_Examples AND DearImGui_Backend_Win32 AND DearImGui_Backend_DirectX12" OFF)

cmake_dependent_option(example_apple_opengl2 "" OFF "DearImGui_Examples AND DearImGui_Backend_OSX AND DearImGui_Backend_OpenGL2" OFF)
cmake_dependent_option(example_apple_metal "" OFF "DearImGui_Examples AND DearImGui_Backend_OSX AND DearImGui_Backend_Metal" OFF)

cmake_dependent_option(example_null "" OFF DearImGui_Examples OFF)

set(imgui_backends_list)
set(imgui_examples_list)
set(imgui_thirdparty_list)
set(pkg_config_requires_list)

cmake_path(SET imgui_include_dir ${_imgui_dir})
cmake_path(ABSOLUTE_PATH imgui_include_dir NORMALIZE)

if(CMAKE_CROSSCOMPILING)
  set(configuration_information_return_value 15) # can't run try_run() on Android, just enable them all
else()
  try_run(configuration_information_return_value compilation_result SOURCE_FROM_CONTENT check_configuration_information_in_imconfig_h.cpp
[=[
#include <imconfig.h>

int main() {
// pack some compile-time configuration definitions related to build on an integer
  unsigned result = 0;

#if defined(IMGUI_ENABLE_TEST_ENGINE)
  result |= (0b1 << 0);
#endif

#if defined(IMGUI_ENABLE_FREETYPE)
  result |= (0b1 << 1);
#endif

#if defined(IMGUI_ENABLE_FREETYPE_LUNASVG)
  result |= (0b1 << 2);
#endif

#if defined(IMGUI_ENABLE_FREETYPE_PLUTOSVG)
  result |= (0b1 << 3);
#endif
  return result;
}
]=] COMPILE_DEFINITIONS "-I${imgui_include_dir}" NO_CACHE NO_LOG)

 # unpack them in here
  math(EXPR is_imgui_test_engine_enabled_in_imconfig "((${configuration_information_return_value} >> 0) & 1)")
  math(EXPR is_imgui_freetype_enabled_in_imconfig "((${configuration_information_return_value} >> 1) & 1)")
  math(EXPR is_imgui_freetype_lunasvg_enabled_in_imconfig "((${configuration_information_return_value} >> 2) & 1)")
  math(EXPR is_imgui_freetype_plutosvg_enabled_in_imconfig "((${configuration_information_return_value} >> 3) & 1)")
endif()

add_library(imgui_core)
target_sources(imgui_core PRIVATE ${_imgui_dir}/imgui.cpp ${_imgui_dir}/imgui_draw.cpp ${_imgui_dir}/imgui_tables.cpp ${_imgui_dir}/imgui_widgets.cpp ${_imgui_dir}/imgui_demo.cpp)
target_sources(imgui_core PUBLIC
  FILE_SET imgui_core_set
  BASE_DIRS ${_imgui_dir}
  TYPE HEADERS
  FILES ${_imgui_dir}/imgui.h ${_imgui_dir}/imconfig.h ${_imgui_dir}/imgui_internal.h ${_imgui_dir}/imstb_rectpack.h ${_imgui_dir}/imstb_textedit.h ${_imgui_dir}/imstb_truetype.h)
target_compile_features(imgui_core PUBLIC cxx_std_11)

if(misc)
  if(misc_cpp)
    target_sources(imgui_core PRIVATE ${_imgui_misc_dir}/cpp/imgui_stdlib.cpp)
    target_sources(imgui_core PUBLIC FILE_SET imgui_misc_cpp_set BASE_DIRS ${_imgui_dir}/misc/cpp TYPE HEADERS FILES ${_imgui_misc_dir}/cpp/imgui_stdlib.h)
  endif()

  if(misc_freetype)
    if(NOT is_imgui_freetype_enabled_in_imconfig)
      message(WARNING "misc_freetype option is enabled but IMGUI_ENABLE_FREETYPE is not defined on imconfig.h. FreeType support feature will not be available.")
      set(misc_freetype OFF)
    else()
      find_package(freetype CONFIG REQUIRED)

      target_sources(imgui_core PRIVATE ${_imgui_misc_dir}/freetype/imgui_freetype.cpp)
      target_link_libraries(imgui_core PUBLIC Freetype::Freetype)
      target_sources(imgui_core PUBLIC FILE_SET imgui_misc_freetype_set BASE_DIRS ${_imgui_dir}/misc/freetype TYPE HEADERS FILES ${_imgui_misc_dir}/freetype/imgui_freetype.h)

      if(is_imgui_freetype_lunasvg_enabled_in_imconfig)
        if(${IMGUI_VERSION} VERSION_LESS 1.92.0)
          find_package(lunasvg 2.4.3 EXACT CONFIG REQUIRED)
        else()
          find_package(lunasvg CONFIG REQUIRED)
        endif()
        target_link_libraries(imgui_core PUBLIC lunasvg::lunasvg)
      elseif(is_imgui_freetype_plutosvg_enabled_in_imconfig)
        find_package(plutosvg CONFIG REQUIRED)
        target_link_libraries(imgui_core PUBLIC plutosvg::plutosvg)
      endif()

      list(APPEND pkg_config_requires_list freetype2)
    endif()
  endif()

  if(misc_fonts)
    add_executable(binary_to_compressed_c)
    target_sources(binary_to_compressed_c PRIVATE ${_imgui_misc_dir}/fonts/binary_to_compressed_c.cpp)
    target_compile_features(binary_to_compressed_c PRIVATE cxx_std_11)
  endif()
endif()

if(DearImGui_Backend_OpenGL2)
  if(CMAKE_SYSTEM_NAME STREQUAL "Windows" AND CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
  else()
    find_package(OpenGL 2 QUIET REQUIRED)
  endif()

  add_library(imgui_backend_opengl2)
  target_sources(imgui_backend_opengl2 PRIVATE ${_imgui_backends_dir}/imgui_impl_opengl2.cpp)
  target_link_libraries(imgui_backend_opengl2 PUBLIC imgui_core)
  target_link_libraries(imgui_backend_opengl2 PUBLIC $<IF:$<AND:$<PLATFORM_ID:Windows>,$<CXX_COMPILER_ID:GNU,Clang>>,opengl32,OpenGL::GL>)
  target_sources(imgui_backend_opengl2 PUBLIC FILE_SET imgui_backend_opengl2_set BASE_DIRS ${_imgui_dir}/backends TYPE HEADERS FILES ${_imgui_backends_dir}/imgui_impl_opengl2.h)

  list(APPEND imgui_backends_list imgui_backend_opengl2)
  list(APPEND pkg_config_requires_list opengl)
endif()

if(DearImGui_Backend_OpenGL3)
  if(CMAKE_SYSTEM_NAME STREQUAL "Windows" AND CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
  else()
    include(FindOpenGL)
  endif()

  if(${IMGUI_VERSION} VERSION_LESS 1.84)
    find_package(GLEW REQUIRED)
  endif()

  add_library(imgui_backend_opengl3)
  target_sources(imgui_backend_opengl3 PRIVATE ${_imgui_backends_dir}/imgui_impl_opengl3.cpp)
  target_link_libraries(imgui_backend_opengl3 PUBLIC imgui_core)
  target_link_libraries(imgui_backend_opengl3 PUBLIC $<IF:$<AND:$<PLATFORM_ID:Windows>,$<CXX_COMPILER_ID:GNU,Clang>>,opengl32,$<IF:$<TARGET_EXISTS:imgui_backend_android>,EGL,OpenGL::GL>>)
  target_sources(imgui_backend_opengl3 PUBLIC FILE_SET imgui_backend_opengl3_set BASE_DIRS ${_imgui_dir}/backends TYPE HEADERS FILES ${_imgui_backends_dir}/imgui_impl_opengl3.h)

  target_link_libraries(imgui_backend_opengl3 PUBLIC $<$<VERSION_LESS:${IMGUI_VERSION},1.84>:GLEW::GLEW>)
  target_compile_definitions(imgui_backend_opengl3 PUBLIC $<$<VERSION_LESS:${IMGUI_VERSION},1.84>:IMGUI_IMPL_OPENGL_LOADER_GLEW>)

  list(APPEND imgui_backends_list imgui_backend_opengl3)
  list(APPEND pkg_config_requires_list opengl)
endif()

if(DearImGui_Backend_Vulkan)
  if(CMAKE_SYSTEM_NAME STREQUAL "Windows" AND CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    list(APPEND CMAKE_FIND_LIBRARY_SUFFIXES .dll)

    if(NOT VULKAN_RUNTIME_COMPONENTS_DIR)
      message(FATAL_ERROR "set VULKAN_RUNTIME_COMPONENTS_DIR to proceed:\ne.g. cmake -DVULKAN_RUNTIME_COMPONENTS_DIR=VulkanRT-X64-x.y.zzz-Components...")
    endif()

    if(CMAKE_SIZEOF_VOID_P EQUAL 4)
      set(suffix x86)
    endif()
    if(CMAKE_SIZEOF_VOID_P EQUAL 8)
      set(suffix x64)
    endif()

    find_library(Vulkan_Libraries vulkan-1 PATHS ${VULKAN_RUNTIME_COMPONENTS_DIR} PATH_SUFFIXES ${suffix} NO_CACHE REQUIRED NO_DEFAULT_PATH NO_CMAKE_SYSTEM_PATH NO_CMAKE_FIND_ROOT_PATH)
    find_package(VulkanHeaders REQUIRED)
  else()
    find_package(Vulkan REQUIRED)
  endif()

  add_library(imgui_backend_vulkan)
  target_sources(imgui_backend_vulkan PRIVATE ${_imgui_backends_dir}/imgui_impl_vulkan.cpp)
  target_link_libraries(imgui_backend_vulkan PUBLIC imgui_core)
  target_link_libraries(imgui_backend_vulkan PUBLIC $<IF:$<AND:$<PLATFORM_ID:Windows>,$<CXX_COMPILER_ID:GNU,Clang>>,Vulkan::Headers ${Vulkan_Libraries},Vulkan::Vulkan>)
  target_sources(imgui_backend_vulkan PUBLIC FILE_SET imgui_backend_vulkan_set BASE_DIRS ${_imgui_dir}/backends TYPE HEADERS FILES ${_imgui_backends_dir}/imgui_impl_vulkan.h)

  list(APPEND imgui_backends_list imgui_backend_vulkan)
  list(APPEND pkg_config_requires_list vulkan)
endif()

if(DearImGui_Backend_WebGPU)
  find_package(Threads REQUIRED)
  find_package(Dawn REQUIRED)

  add_library(imgui_backend_wgpu)
  target_sources(imgui_backend_wgpu PRIVATE ${_imgui_backends_dir}/imgui_impl_wgpu.cpp)
  target_sources(imgui_backend_wgpu PUBLIC FILE_SET imgui_backend_wgpu_set BASE_DIRS ${_imgui_dir}/backends TYPE HEADERS FILES ${_imgui_backends_dir}/imgui_impl_wgpu.h)
  target_link_libraries(imgui_backend_wgpu PUBLIC imgui_core)
  target_link_libraries(imgui_backend_wgpu PUBLIC dawn::webgpu_dawn dawn::dawn_public_config Threads::Threads)
  target_compile_definitions(imgui_backend_wgpu PUBLIC IMGUI_IMPL_WEBGPU_BACKEND_DAWN)

  list(APPEND imgui_backends_list imgui_backend_wgpu)
endif()

if(DearImGui_Backend_Android)
  if(NOT DEFINED CMAKE_ANDROID_NDK)
    message(FATAL_ERROR
[=[
Android NDK toolchain is not used
  Try to utilize Android toolchain file when running cmake: --toolchain $ANDROID_NDK/build/cmake/android.toolchain.cmake
  https://developer.android.com/ndk/guides/cmake#the_new_toolchain_file
]=])
  endif()

  cmake_print_variables(CMAKE_SYSTEM_NAME CMAKE_ANDROID_NDK CMAKE_TOOLCHAIN_FILE ANDROID_TOOLCHAIN ANDROID_TOOLCHAIN_NAME $ENV{ANDROID_HOME} $ENV{ANDROID_NDK} ANDROID_ABI ANDROID_STL ANDROID_PIE ANDROID_CPP_FEATURES ANDROID_ALLOW_UNDEFINED_SYMBOLS ANDROID_ARM_MODE ANDROID_PLATFORM ANDROID_NATIVE_API_LEVEL)

  if(CMAKE_ANDROID_ARCH_ABI STREQUAL "armeabi-v7a" OR CMAKE_ANDROID_ARCH_ABI STREQUAL "armeabi-v7a with NEON")
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

  cmake_host_system_information(RESULT os_name QUERY OS_NAME)
  if(os_name STREQUAL "Darwin")
    set(host_tag darwin-x86_64)
  elseif(os_name STREQUAL "Linux")
    set(host_tag linux-x86_64)
  elseif(os_name STREQUAL "Windows")
    set(host_tag windows-x86_64)
  endif()

  set(Android_NDK_INCLUDE_DIR $ENV{ANDROID_NDK}/toolchains/llvm/prebuilt/${host_tag}/sysroot/usr/include)
  set(Android_NDK_LIB_DIR $ENV{ANDROID_NDK}/toolchains/llvm/prebuilt/${host_tag}/sysroot/usr/lib/${triplet}/${api_level})

  add_library(imgui_backend_android)
  target_sources(imgui_backend_android PRIVATE ${_imgui_backends_dir}/imgui_impl_android.cpp)
  target_link_libraries(imgui_backend_android PUBLIC imgui_core)
  target_sources(imgui_backend_android PUBLIC FILE_SET imgui_backend_android_set BASE_DIRS ${_imgui_dir}/backends TYPE HEADERS FILES ${_imgui_backends_dir}/imgui_impl_android.h)
  target_link_directories(imgui_backend_android PUBLIC ${Android_NDK_LIB_DIR})
  target_link_libraries(imgui_backend_android PUBLIC log android)
  target_sources(imgui_backend_android PRIVATE FILE_SET HEADERS BASE_DIRS ${Android_NDK_INCLUDE_DIR} FILES ${Android_NDK_INCLUDE_DIR}/android/native_window.h ${Android_NDK_INCLUDE_DIR}/android/input.h ${Android_NDK_INCLUDE_DIR}/android/keycodes.h ${Android_NDK_INCLUDE_DIR}/android/log.h)

  list(APPEND imgui_backends_list imgui_backend_android)

  if(example_android_opengl3)
    find_program(GRADLE_COMMAND gradle REQUIRED)

    cmake_path(SET android_example_output_path ${_imgui_examples_dir}/example_android_opengl3/android/app/build/outputs/apk)
    cmake_path(ABSOLUTE_PATH android_example_output_path NORMALIZE)

    add_custom_command(TARGET imgui_backend_android POST_BUILD
      COMMAND ${GRADLE_COMMAND} --project-dir ${_imgui_examples_dir}/example_android_opengl3/android assembleRelease
      WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
      COMMENT "Building .apk files. Output will be on: ${android_example_output_path}"
      VERBATIM)
  endif()
endif()

if(DearImGui_Backend_Allegro5)
  find_package(Allegro CONFIG REQUIRED)

  add_library(imgui_backend_allegro5)
  target_sources(imgui_backend_allegro5 PRIVATE ${_imgui_backends_dir}/imgui_impl_allegro5.cpp)
  target_link_libraries(imgui_backend_allegro5 PUBLIC imgui_core)
  target_link_libraries(imgui_backend_allegro5 PUBLIC Allegro::allegro Allegro::allegro_primitives INTERFACE Allegro::allegro_main)
  target_sources(imgui_backend_allegro5 PUBLIC FILE_SET imgui_backend_allegro5_set BASE_DIRS ${_imgui_dir}/backends TYPE HEADERS FILES ${_imgui_backends_dir}/imgui_impl_allegro5.h)

  list(APPEND imgui_backends_list imgui_backend_allegro5)
  list(APPEND pkg_config_requires_list allegro-5 allegro_primitives-5)

  if(example_allegro5)
    add_executable(example_allegro5)
    target_sources(example_allegro5 PRIVATE ${_imgui_examples_dir}/example_allegro5/main.cpp)
    target_link_libraries(example_allegro5 PRIVATE imgui_backend_allegro5)
    target_link_libraries(example_allegro5 PRIVATE "$<$<PLATFORM_ID:Darwin>:-framework OpenGL -framework Carbon -framework ApplicationServices>")

    list(APPEND imgui_examples_list example_allegro5)
  endif()
endif()

if(DearImGui_Backend_GLFW)
  find_package(glfw3 CONFIG REQUIRED)

  add_library(imgui_backend_glfw)
  target_sources(imgui_backend_glfw PRIVATE ${_imgui_backends_dir}/imgui_impl_glfw.cpp)
  target_sources(imgui_backend_glfw PUBLIC FILE_SET imgui_backend_glfw_set BASE_DIRS ${_imgui_dir}/backends TYPE HEADERS FILES ${_imgui_backends_dir}/imgui_impl_glfw.h)
  target_link_libraries(imgui_backend_glfw PUBLIC glfw)
  target_link_libraries(imgui_backend_glfw PUBLIC imgui_core)

  list(APPEND imgui_backends_list imgui_backend_glfw)
  list(APPEND pkg_config_requires_list glfw3)

  if(example_glfw_opengl3)
    add_executable(example_glfw_opengl3)
    target_sources(example_glfw_opengl3 PRIVATE ${_imgui_examples_dir}/example_glfw_opengl3/main.cpp)
    target_link_libraries(example_glfw_opengl3 PRIVATE imgui_backend_glfw imgui_backend_opengl3)

    list(APPEND imgui_examples_list example_glfw_opengl3)
  endif()

  if(example_glfw_opengl2)
    add_executable(example_glfw_opengl2)
    target_sources(example_glfw_opengl2 PRIVATE ${_imgui_examples_dir}/example_glfw_opengl2/main.cpp)
    target_link_libraries(example_glfw_opengl2 PRIVATE imgui_backend_glfw imgui_backend_opengl2)

    list(APPEND imgui_examples_list example_glfw_opengl2)
  endif()

  if(example_glfw_vulkan)
    add_executable(example_glfw_vulkan)
    target_sources(example_glfw_vulkan PRIVATE ${_imgui_examples_dir}/example_glfw_vulkan/main.cpp)
    target_link_libraries(example_glfw_vulkan PRIVATE imgui_backend_glfw imgui_backend_vulkan)

    list(APPEND imgui_examples_list example_glfw_vulkan)
  endif()

  if(example_glfw_metal)
    add_executable(example_glfw_metal)
    target_sources(example_glfw_metal PRIVATE ${_imgui_examples_dir}/example_glfw_metal/main.mm)
    target_link_libraries(example_glfw_metal PRIVATE imgui_backend_glfw imgui_backend_metal)
    target_link_libraries(example_glfw_metal PRIVATE "$<$<PLATFORM_ID:Darwin>:-framework QuartzCore>")

    list(APPEND imgui_examples_list example_glfw_metal)
  endif()
endif()

if(DearImGui_Backend_FreeGLUT)
  find_package(FreeGLUT CONFIG REQUIRED)

  add_library(imgui_backend_glut)
  target_sources(imgui_backend_glut PRIVATE ${_imgui_backends_dir}/imgui_impl_glut.cpp)
  target_sources(imgui_backend_glut PUBLIC FILE_SET imgui_backend_glut_set BASE_DIRS ${_imgui_dir}/backends TYPE HEADERS FILES ${_imgui_backends_dir}/imgui_impl_glut.h)
  target_link_libraries(imgui_backend_glut PUBLIC $<IF:$<TARGET_EXISTS:FreeGLUT::freeglut>,FreeGLUT::freeglut,FreeGLUT::freeglut_static>)
  target_link_libraries(imgui_backend_glut PUBLIC imgui_core)

  list(APPEND imgui_backends_list imgui_backend_glut)
  list(APPEND pkg_config_requires_list glut)

  if(example_glut_opengl2)
    add_executable(example_glut_opengl2)
    target_sources(example_glut_opengl2 PRIVATE ${_imgui_examples_dir}/example_glut_opengl2/main.cpp)
    target_link_libraries(example_glut_opengl2 PRIVATE imgui_backend_glut imgui_backend_opengl2)

    list(APPEND imgui_examples_list example_glut_opengl2)
  endif()
endif()

if(DearImGui_Backend_SDL2)
  find_package(SDL2 CONFIG REQUIRED)

  add_library(imgui_backend_sdl2)
  target_sources(imgui_backend_sdl2 PRIVATE $<IF:$<VERSION_GREATER:${IMGUI_VERSION},1.89.2>,${_imgui_backends_dir}/imgui_impl_sdl2.cpp,${_imgui_backends_dir}/imgui_impl_sdl.cpp>)
  target_sources(imgui_backend_sdl2 PUBLIC FILE_SET imgui_backend_sdl2_set BASE_DIRS ${_imgui_dir}/backends TYPE HEADERS FILES $<IF:$<VERSION_GREATER:${IMGUI_VERSION},1.89.2>,${_imgui_backends_dir}/imgui_impl_sdl2.h,${_imgui_backends_dir}/imgui_impl_sdl.h>)
  target_link_libraries(imgui_backend_sdl2 PUBLIC $<IF:$<TARGET_EXISTS:SDL2::SDL2>,SDL2::SDL2,SDL2::SDL2-static>)
  target_link_libraries(imgui_backend_sdl2 PUBLIC imgui_core)

  list(APPEND imgui_backends_list imgui_backend_sdl2)
  list(APPEND pkg_config_requires_list sdl2)

  if(example_sdl2_opengl2)
    add_executable(example_sdl2_opengl2)
    target_sources(example_sdl2_opengl2 PRIVATE $<IF:$<VERSION_GREATER:${IMGUI_VERSION},1.89.2>,${_imgui_dir}/examples/example_sdl2_opengl2/main.cpp,${_imgui_dir}/examples/example_sdl_opengl2/main.cpp>)
    target_link_libraries(example_sdl2_opengl2 PRIVATE $<$<PLATFORM_ID:Windows>:SDL2::SDL2main> imgui_backend_sdl2 imgui_backend_opengl2)

    list(APPEND imgui_examples_list example_sdl2_opengl2)
  endif()

  if(example_sdl2_opengl3)
    add_executable(example_sdl2_opengl3)
    target_sources(example_sdl2_opengl3 PRIVATE $<IF:$<VERSION_GREATER:${IMGUI_VERSION},1.89.2>,${_imgui_dir}/examples/example_sdl2_opengl3/main.cpp,${_imgui_dir}/examples/example_sdl_opengl3/main.cpp>)
    target_link_libraries(example_sdl2_opengl3 PRIVATE $<$<PLATFORM_ID:Windows>:SDL2::SDL2main> imgui_backend_sdl2 imgui_backend_opengl3)
    if(CMAKE_SYSTEM_NAME STREQUAL "Emscripten")
      set_target_properties(example_sdl2_opengl3 PROPERTIES SUFFIX ".html")
    endif()

    list(APPEND imgui_examples_list example_sdl2_opengl3)
  endif()

  if(example_sdl2_vulkan)
    add_executable(example_sdl2_vulkan)
    target_sources(example_sdl2_vulkan PRIVATE $<IF:$<VERSION_GREATER:${IMGUI_VERSION},1.89.2>,${_imgui_dir}/examples/example_sdl2_vulkan/main.cpp,${_imgui_dir}/examples/example_sdl_vulkan/main.cpp>)
    target_link_libraries(example_sdl2_vulkan PRIVATE $<$<PLATFORM_ID:Windows>:SDL2::SDL2main> imgui_backend_sdl2 imgui_backend_vulkan)

    list(APPEND imgui_examples_list example_sdl2_vulkan)
  endif()

  if(example_sdl2_directx11)
    add_executable(example_sdl2_directx11)
    target_sources(example_sdl2_directx11 PRIVATE $<IF:$<VERSION_GREATER:${IMGUI_VERSION},1.89.2>,${_imgui_dir}/examples/example_sdl2_directx11/main.cpp,${_imgui_dir}/examples/example_sdl_directx11/main.cpp>)
    target_link_libraries(example_sdl2_directx11 PRIVATE $<$<PLATFORM_ID:Windows>:SDL2::SDL2main> imgui_backend_sdl2 imgui_backend_dx11)

    list(APPEND imgui_examples_list example_sdl2_directx11)
  endif()

  if(example_sdl2_metal)
    add_executable(example_sdl2_metal)
    target_sources(example_sdl2_metal PRIVATE $<IF:$<VERSION_GREATER:${IMGUI_VERSION},1.89.2>,${_imgui_dir}/examples/example_sdl2_metal/main.mm,${_imgui_dir}/examples/example_sdl_metal/main.mm>)
    target_link_libraries(example_sdl2_metal PRIVATE imgui_core)
    target_link_libraries(example_sdl2_metal PRIVATE imgui_backend_sdl2 imgui_backend_metal)

    list(APPEND imgui_examples_list example_sdl2_metal)
  endif()
endif()

if(DearImGui_Backend_SDLRenderer2)
  find_package(SDL2 CONFIG REQUIRED)

  add_library(imgui_backend_sdlrenderer2)
  target_sources(imgui_backend_sdlrenderer2 PRIVATE $<IF:$<VERSION_GREATER:${IMGUI_VERSION},1.89.5>,${_imgui_backends_dir}/imgui_impl_sdlrenderer2.cpp,${_imgui_backends_dir}/imgui_impl_sdlrenderer.cpp>)
  target_sources(imgui_backend_sdlrenderer2 PUBLIC FILE_SET imgui_backend_sdlrenderer2_set BASE_DIRS ${_imgui_dir}/backends TYPE HEADERS FILES $<IF:$<VERSION_GREATER:${IMGUI_VERSION},1.89.5>,${_imgui_backends_dir}/imgui_impl_sdlrenderer2.h,${_imgui_backends_dir}/imgui_impl_sdlrenderer.h>)
  target_link_libraries(imgui_backend_sdlrenderer2 PUBLIC $<IF:$<TARGET_EXISTS:SDL2::SDL2>,SDL2::SDL2,SDL2::SDL2-static>)
  target_link_libraries(imgui_backend_sdlrenderer2 PUBLIC imgui_core)

  list(APPEND imgui_backends_list imgui_backend_sdlrenderer2)
  list(APPEND pkg_config_requires_list sdl2)

  if(example_sdl2_sdlrenderer2)
    add_executable(example_sdl2_sdlrenderer2)
    target_sources(example_sdl2_sdlrenderer2 PRIVATE $<IF:$<VERSION_GREATER:${IMGUI_VERSION},1.89.5>,${_imgui_examples_dir}/example_sdl2_sdlrenderer2/main.cpp,$<IF:$<VERSION_GREATER:${IMGUI_VERSION},1.89.2>,${_imgui_examples_dir}/example_sdl2_sdlrenderer/main.cpp,${_imgui_examples_dir}/example_sdl_sdlrenderer/main.cpp>>)
    target_link_libraries(example_sdl2_sdlrenderer2 PRIVATE $<$<PLATFORM_ID:Windows>:SDL2::SDL2main> imgui_backend_sdlrenderer2 imgui_backend_sdl2)

    list(APPEND imgui_examples_list example_sdl2_sdlrenderer2)
  endif()
endif()

if(DearImGui_Backend_SDL3)
  find_package(SDL3 CONFIG REQUIRED)

  add_library(imgui_backend_sdl3)
  target_sources(imgui_backend_sdl3 PRIVATE ${_imgui_backends_dir}/imgui_impl_sdl3.cpp)
  target_sources(imgui_backend_sdl3 PUBLIC FILE_SET imgui_backend_sdl3_set BASE_DIRS ${_imgui_dir}/backends TYPE HEADERS FILES ${_imgui_backends_dir}/imgui_impl_sdl3.h)
  target_link_libraries(imgui_backend_sdl3 PUBLIC SDL3::SDL3)
  target_link_libraries(imgui_backend_sdl3 PUBLIC imgui_core)
  target_compile_definitions(imgui_backend_sdl3 PUBLIC $<$<VERSION_LESS:${IMGUI_VERSION},1.91.3>:SDL_ENABLE_OLD_NAMES>)

  list(APPEND imgui_backends_list imgui_backend_sdl3)
  list(APPEND pkg_config_requires_list sdl3)

  if(example_sdl3_opengl3)
    add_executable(example_sdl3_opengl3)
    target_sources(example_sdl3_opengl3 PRIVATE ${_imgui_examples_dir}/example_sdl3_opengl3/main.cpp)
    target_link_libraries(example_sdl3_opengl3 PRIVATE imgui_backend_sdl3 imgui_backend_opengl3)
    target_compile_definitions(example_sdl3_opengl3 PRIVATE $<$<VERSION_LESS:${IMGUI_VERSION},1.91.3>:SDL_INIT_TIMER=0x00000001u>)

    list(APPEND imgui_examples_list example_sdl3_opengl3)
  endif()

  if(example_sdl3_vulkan)
    add_executable(example_sdl3_vulkan)
    target_sources(example_sdl3_vulkan PRIVATE ${_imgui_examples_dir}/example_sdl3_vulkan/main.cpp)
    target_link_libraries(example_sdl3_vulkan PRIVATE imgui_backend_sdl3 imgui_backend_vulkan)

    list(APPEND imgui_examples_list example_sdl3_vulkan)
  endif()

  if(example_sdl3_metal)
    add_executable(example_sdl3_metal)
    target_sources(example_sdl3_metal PRIVATE ${_imgui_examples_dir}/example_sdl3_metal/main.mm)
    target_link_libraries(example_sdl3_metal PRIVATE imgui_backend_sdl3 imgui_backend_metal)

    list(APPEND imgui_examples_list example_sdl3_metal)
  endif()
endif()

if(DearImGui_Backend_SDLGPU3)
  find_package(SDL3 CONFIG REQUIRED)

  add_library(imgui_backend_sdlgpu3)
  target_sources(imgui_backend_sdlgpu3 PRIVATE ${_imgui_backends_dir}/imgui_impl_sdlgpu3.cpp)
  target_sources(imgui_backend_sdlgpu3 PUBLIC FILE_SET imgui_backend_sdlgpu3_set BASE_DIRS ${_imgui_dir}/backends TYPE HEADERS FILES ${_imgui_backends_dir}/imgui_impl_sdlgpu3.h
                                              ${_imgui_backends_dir}/imgui_impl_sdlgpu3_shaders.h)
  target_link_libraries(imgui_backend_sdlgpu3 PUBLIC SDL3::SDL3)
  target_link_libraries(imgui_backend_sdlgpu3 PUBLIC imgui_core)

  list(APPEND imgui_backends_list imgui_backend_sdlgpu3)
  list(APPEND pkg_config_requires_list sdl3)

  if(example_sdl3_sdlgpu3)
    add_executable(example_sdl3_sdlgpu3)
    target_sources(example_sdl3_sdlgpu3 PRIVATE ${_imgui_examples_dir}/example_sdl3_sdlgpu3/main.cpp)
    target_link_libraries(example_sdl3_sdlgpu3 PRIVATE imgui_backend_sdlgpu3 imgui_backend_sdl3)

    list(APPEND imgui_examples_list example_sdl3_sdlgpu3)
  endif()
endif()

if(DearImGui_Backend_SDLRenderer3)
  find_package(SDL3 CONFIG REQUIRED)

  add_library(imgui_backend_sdlrenderer3)
  target_sources(imgui_backend_sdlrenderer3 PRIVATE ${_imgui_backends_dir}/imgui_impl_sdlrenderer3.cpp)
  target_sources(imgui_backend_sdlrenderer3 PUBLIC FILE_SET imgui_backend_sdlrenderer3_set BASE_DIRS ${_imgui_dir}/backends TYPE HEADERS FILES
                                                   ${_imgui_backends_dir}/imgui_impl_sdlrenderer3.h)
  target_link_libraries(imgui_backend_sdlrenderer3 PUBLIC SDL3::SDL3)
  target_link_libraries(imgui_backend_sdlrenderer3 PUBLIC imgui_core imgui_backend_sdl3)

  list(APPEND imgui_backends_list imgui_backend_sdlrenderer3)
  list(APPEND pkg_config_requires_list sdl3)

  if(example_sdl3_sdlrenderer3)
    add_executable(example_sdl3_sdlrenderer3)
    target_sources(example_sdl3_sdlrenderer3 PRIVATE ${_imgui_examples_dir}/example_sdl3_sdlrenderer3/main.cpp)
    target_link_libraries(example_sdl3_sdlrenderer3 PRIVATE imgui_backend_sdlrenderer3 imgui_backend_sdl3)
    target_compile_definitions(example_sdl3_sdlrenderer3 PRIVATE $<$<VERSION_LESS:${IMGUI_VERSION},1.91.3>:SDL_INIT_TIMER=0x00000001u>)

    list(APPEND imgui_examples_list example_sdl3_sdlrenderer3)
  endif()
endif()

if(DearImGui_Backend_Win32)
  add_library(imgui_backend_win32)
  target_sources(imgui_backend_win32 PRIVATE ${_imgui_backends_dir}/imgui_impl_win32.cpp)
  target_sources(imgui_backend_win32 PUBLIC FILE_SET imgui_backend_win32_set BASE_DIRS ${_imgui_dir}/backends TYPE HEADERS FILES ${_imgui_backends_dir}/imgui_impl_win32.h)
  target_link_libraries(imgui_backend_win32 PUBLIC $<$<AND:$<PLATFORM_ID:Windows>,$<CXX_COMPILER_ID:GNU,Clang>>:dwmapi>)
  target_link_libraries(imgui_backend_win32 PUBLIC imgui_core)

  list(APPEND imgui_backends_list imgui_backend_win32)

  if(example_win32_opengl3)
    add_executable(example_win32_opengl3)
    target_sources(example_win32_opengl3 PRIVATE ${_imgui_examples_dir}/example_win32_opengl3/main.cpp)
    target_link_libraries(example_win32_opengl3 PRIVATE imgui_backend_win32 imgui_backend_opengl3)

    list(APPEND imgui_examples_list example_win32_opengl3)
  endif()

  if(example_win32_vulkan)
    add_executable(example_win32_vulkan)
    target_sources(example_win32_vulkan PRIVATE ${_imgui_examples_dir}/example_win32_vulkan/main.cpp)
    target_link_libraries(example_win32_vulkan PRIVATE imgui_backend_win32 imgui_backend_vulkan)

    list(APPEND imgui_examples_list example_win32_vulkan)
  endif()

  if(example_win32_directx9)
    add_executable(example_win32_directx9)
    target_sources(example_win32_directx9 PRIVATE ${_imgui_examples_dir}/example_win32_directx9/main.cpp)
    target_link_libraries(example_win32_directx9 PRIVATE imgui_backend_win32 imgui_backend_dx9)

    list(APPEND imgui_examples_list example_win32_directx9)
  endif()

  if(example_win32_directx10)
    add_executable(example_win32_directx10)
    target_sources(example_win32_directx10 PRIVATE ${_imgui_examples_dir}/example_win32_directx10/main.cpp)
    target_link_libraries(example_win32_directx10 PRIVATE imgui_backend_win32 imgui_backend_dx10)

    list(APPEND imgui_examples_list example_win32_directx10)
  endif()

  if(example_win32_directx11)
    add_executable(example_win32_directx11)
    target_sources(example_win32_directx11 PRIVATE ${_imgui_examples_dir}/example_win32_directx11/main.cpp)
    target_link_libraries(example_win32_directx11 PRIVATE imgui_backend_win32 imgui_backend_dx11)

    list(APPEND imgui_examples_list example_win32_directx11)
  endif()

  if(example_win32_directx12)
    add_executable(example_win32_directx12)
    target_sources(example_win32_directx12 PRIVATE ${_imgui_examples_dir}/example_win32_directx12/main.cpp)
    target_link_libraries(example_win32_directx12 PRIVATE imgui_backend_win32 imgui_backend_dx12)

    list(APPEND imgui_examples_list example_win32_directx12)
  endif()
endif()

if(example_null)
  add_executable(example_null)
  target_sources(example_null PRIVATE ${_imgui_examples_dir}/example_null/main.cpp)
  target_link_libraries(example_null PRIVATE imgui_core)

  list(APPEND imgui_examples_list example_null)
endif()

if((DearImGui_Backend_DirectX9 OR DearImGui_Backend_DirectX10 OR DearImGui_Backend_DirectX11 OR DearImGui_Backend_DirectX12) AND (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC"))
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

if(DearImGui_Backend_DirectX9)
  add_library(imgui_backend_dx9)
  target_sources(imgui_backend_dx9 PRIVATE ${_imgui_backends_dir}/imgui_impl_dx9.cpp)
  target_sources(imgui_backend_dx9 PUBLIC FILE_SET imgui_backend_dx9_set BASE_DIRS ${_imgui_dir}/backends TYPE HEADERS FILES ${_imgui_backends_dir}/imgui_impl_dx9.h)
  if(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    target_link_directories(imgui_backend_dx9 PRIVATE ${Windows_SDK_LIB_DIR})
    target_link_libraries(imgui_backend_dx9 PUBLIC d3d9.lib)
    target_sources(imgui_backend_dx9 PRIVATE FILE_SET HEADERS BASE_DIRS ${Windows_SDK_INCLUDE_DIR} FILES ${Windows_SDK_INCLUDE_DIR}/d3d9.h)
  elseif(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang") # cross compiling with mingw or msys2
    target_link_libraries(imgui_backend_dx9 PUBLIC d3d9.lib)
  endif()

  target_link_libraries(imgui_backend_dx9 PUBLIC imgui_core)

  list(APPEND imgui_backends_list imgui_backend_dx9)
endif()

if(DearImGui_Backend_DirectX10)
  add_library(imgui_backend_dx10)
  target_sources(imgui_backend_dx10 PRIVATE ${_imgui_backends_dir}/imgui_impl_dx10.cpp)
  target_sources(imgui_backend_dx10 PUBLIC FILE_SET imgui_backend_dx10_set BASE_DIRS ${_imgui_dir}/backends TYPE HEADERS FILES ${_imgui_backends_dir}/imgui_impl_dx10.h)
  if(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    target_link_directories(imgui_backend_dx10 PRIVATE ${Windows_SDK_LIB_DIR})
    target_link_libraries(imgui_backend_dx10 PUBLIC d3d10_1.lib d3d10.lib d3dcompiler.lib)
    target_sources(imgui_backend_dx10 PRIVATE FILE_SET HEADERS BASE_DIRS ${Windows_SDK_INCLUDE_DIR_1} FILES ${Windows_SDK_INCLUDE_DIR_1}/d3d10_1.h
                                            ${Windows_SDK_INCLUDE_DIR_1}/d3d10.h ${Windows_SDK_INCLUDE_DIR_1}/d3dcompiler.h)
  elseif(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_link_libraries(imgui_backend_dx10 PUBLIC d3d10.lib d3dcompiler.lib)
  endif()
  target_link_libraries(imgui_backend_dx10 PUBLIC imgui_core)

  list(APPEND imgui_backends_list imgui_backend_dx10)
endif()

if(DearImGui_Backend_DirectX11)
  add_library(imgui_backend_dx11)
  target_sources(imgui_backend_dx11 PRIVATE ${_imgui_backends_dir}/imgui_impl_dx11.cpp)
  target_sources(imgui_backend_dx11 PUBLIC FILE_SET imgui_backend_dx11_set BASE_DIRS ${_imgui_dir}/backends TYPE HEADERS FILES ${_imgui_backends_dir}/imgui_impl_dx11.h)
  if(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    target_link_directories(imgui_backend_dx11 PRIVATE ${Windows_SDK_LIB_DIR})
    target_link_libraries(imgui_backend_dx11 PUBLIC d3d11.lib d3dcompiler.lib)
    target_sources(imgui_backend_dx11 PRIVATE FILE_SET HEADERS BASE_DIRS ${Windows_SDK_INCLUDE_DIR_1} FILES ${Windows_SDK_INCLUDE_DIR_1}/d3d11.h
                                            ${Windows_SDK_INCLUDE_DIR_1}/d3dcompiler.h)
  elseif(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_link_libraries(imgui_backend_dx11 PUBLIC d3d11.lib d3dcompiler.lib)
  endif()
  target_link_libraries(imgui_backend_dx11 PUBLIC imgui_core)

  list(APPEND imgui_backends_list imgui_backend_dx11)
endif()

if(DearImGui_Backend_DirectX12)
  add_library(imgui_backend_dx12)
  target_sources(imgui_backend_dx12 PRIVATE ${_imgui_backends_dir}/imgui_impl_dx12.cpp)
  target_sources(imgui_backend_dx12 PUBLIC FILE_SET imgui_backend_dx12_set BASE_DIRS ${_imgui_dir}/backends TYPE HEADERS FILES ${_imgui_backends_dir}/imgui_impl_dx12.h)
  if(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    target_link_directories(imgui_backend_dx12 PRIVATE ${Windows_SDK_LIB_DIR})
    target_link_libraries(imgui_backend_dx12 PUBLIC d3d12.lib dxgi.lib d3dcompiler.lib)
    target_sources(imgui_backend_dx12 PRIVATE FILE_SET HEADERS BASE_DIRS ${Windows_SDK_INCLUDE_DIR_1} FILES ${Windows_SDK_INCLUDE_DIR_1}/d3d12.h
                                            ${Windows_SDK_INCLUDE_DIR_1}/d3dcompiler.h)
    target_sources(imgui_backend_dx12 PRIVATE FILE_SET HEADERS BASE_DIRS ${Windows_SDK_INCLUDE_DIR} FILES ${Windows_SDK_INCLUDE_DIR}/dxgi1_4.h)
  elseif(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_link_libraries(imgui_backend_dx12 PUBLIC d3d12.lib dxgi.lib d3dcompiler.lib)
  endif()
  target_link_libraries(imgui_backend_dx12 PUBLIC imgui_core)

  list(APPEND imgui_backends_list imgui_backend_dx12)
endif()

if(DearImGui_Backend_Metal)
  add_library(imgui_backend_metal)
  target_sources(imgui_backend_metal PRIVATE ${_imgui_backends_dir}/imgui_impl_metal.mm)
  target_sources(imgui_backend_metal PUBLIC FILE_SET imgui_backend_metal_set BASE_DIRS ${_imgui_dir}/backends TYPE HEADERS FILES ${_imgui_backends_dir}/imgui_impl_metal.h)
  target_link_libraries(imgui_backend_metal PUBLIC imgui_core)
  target_link_libraries(imgui_backend_metal PUBLIC "-framework Metal" INTERFACE "-framework MetalKit")
  target_compile_options(imgui_backend_metal PUBLIC $<$<VERSION_LESS:${IMGUI_VERSION},1.89.4>:-fobjc-arc>)

  list(APPEND imgui_backends_list imgui_backend_metal)
endif()

if(DearImGui_Backend_OSX)
  add_library(imgui_backend_osx)
  target_sources(imgui_backend_osx PRIVATE ${_imgui_backends_dir}/imgui_impl_osx.mm)
  target_sources(imgui_backend_osx PUBLIC FILE_SET imgui_backend_osx_set BASE_DIRS ${_imgui_dir}/backends TYPE HEADERS FILES ${_imgui_backends_dir}/imgui_impl_osx.h)
  target_link_libraries(imgui_backend_osx PUBLIC imgui_core)
  target_link_libraries(imgui_backend_osx PUBLIC "-framework AppKit" "-framework Carbon" "-framework GameController")

  list(APPEND imgui_backends_list imgui_backend_osx)

  if(example_apple_opengl2)
    add_executable(example_apple_opengl2 MACOSX_BUNDLE)
    target_sources(example_apple_opengl2 PRIVATE ${_imgui_examples_dir}/example_apple_opengl2/main.mm)
    target_link_libraries(example_apple_opengl2 PRIVATE imgui_backend_osx imgui_backend_opengl2)

    list(APPEND imgui_examples_list example_apple_opengl2)
  endif()

  if(example_apple_metal)
    add_executable(example_apple_metal MACOSX_BUNDLE)
    target_sources(example_apple_metal PRIVATE ${_imgui_examples_dir}/example_apple_metal/main.mm)
    target_link_libraries(example_apple_metal PRIVATE imgui_backend_osx imgui_backend_metal)

    list(APPEND imgui_examples_list example_apple_metal)
  endif()
endif()

if(imgui_test_engine)
  if(NOT IMGUI_TEST_ENGINE_SOURCE_DIR)
    message(FATAL_ERROR "set IMGUI_TEST_ENGINE_SOURCE_DIR to proceed:\ne.g. cmake -DIMGUI_TEST_ENGINE_SOURCE_DIR=path.to.imgui_test_engine...")
  endif()
  set(_imgui_te_dir ${IMGUI_TEST_ENGINE_SOURCE_DIR})

  if(NOT is_imgui_test_engine_enabled_in_imconfig)
    message(WARNING "imgui_test_engine option is enabled but IMGUI_ENABLE_TEST_ENGINE is not defined on imconfig.h. ImGui Test Engine is not configurated.")
    set(imgui_test_engine OFF)
  else()

    add_library(imgui_test_engine)
    target_sources(imgui_test_engine PRIVATE
      ${_imgui_te_dir}/imgui_test_engine/imgui_te_context.cpp
      ${_imgui_te_dir}/imgui_test_engine/imgui_te_coroutine.cpp
      ${_imgui_te_dir}/imgui_test_engine/imgui_te_engine.cpp
      ${_imgui_te_dir}/imgui_test_engine/imgui_te_exporters.cpp
      ${_imgui_te_dir}/imgui_test_engine/imgui_te_perftool.cpp
      ${_imgui_te_dir}/imgui_test_engine/imgui_te_ui.cpp
      ${_imgui_te_dir}/imgui_test_engine/imgui_te_utils.cpp
      ${_imgui_te_dir}/imgui_test_engine/imgui_capture_tool.cpp)

    target_sources(imgui_test_engine PUBLIC FILE_SET imgui_test_engine_set BASE_DIRS ${_imgui_te_dir}/imgui_test_engine
      TYPE HEADERS
      FILES
        ${_imgui_te_dir}/imgui_test_engine/imgui_te_context.h
        ${_imgui_te_dir}/imgui_test_engine/imgui_te_coroutine.h
        ${_imgui_te_dir}/imgui_test_engine/imgui_te_engine.h
        ${_imgui_te_dir}/imgui_test_engine/imgui_te_exporters.h
        ${_imgui_te_dir}/imgui_test_engine/imgui_te_imconfig.h
        ${_imgui_te_dir}/imgui_test_engine/imgui_te_internal.h
        ${_imgui_te_dir}/imgui_test_engine/imgui_te_perftool.h
        ${_imgui_te_dir}/imgui_test_engine/imgui_te_ui.h
        ${_imgui_te_dir}/imgui_test_engine/imgui_te_utils.h
        ${_imgui_te_dir}/imgui_test_engine/imgui_capture_tool.h
        ${_imgui_te_dir}/imgui_test_engine/thirdparty/stb/imstb_image_write.h
        ${_imgui_te_dir}/imgui_test_engine/thirdparty/Str/Str.h)
    target_compile_features(imgui_test_engine PUBLIC cxx_std_14)

    target_link_libraries(imgui_test_engine PRIVATE imgui_core)
    target_link_libraries(imgui_core INTERFACE imgui_test_engine)

    cmake_path(SET imgui_te_dir ${_imgui_te_dir})
    cmake_path(ABSOLUTE_PATH imgui_te_dir NORMALIZE)

    try_run(return_value compilation_result SOURCE_FROM_CONTENT check_imgui_test_engine_performans_viewer_feature.cpp
[=[
#include <imgui_test_suite_imconfig.h>

int main() {
  unsigned result = 0;
#if IMGUI_TEST_ENGINE_ENABLE_IMPLOT == 1
  result |= (0b1 << 0);
#endif
  return result;
}
]=] COMPILE_DEFINITIONS "-I${imgui_te_dir}/imgui_test_suite -I ${imgui_include_dir}" NO_CACHE NO_LOG)

    math(EXPR is_imgui_test_engine_implot_enabled "((${return_value} >> 0) & 1)")

    if(is_imgui_test_engine_implot_enabled AND NOT implot)
      message(WARNING "In imgui_te_imconfig.h or imgui_test_suite_imconfig.h IMGUI_TEST_ENGINE_ENABLE_IMPLOT is set but ImPlot is not configurated. Performance viewer feature will not be available. Try with 'cmake -Dimplot=ON' ...")
    endif()
    target_link_libraries(imgui_test_engine INTERFACE $<$<BOOL:${is_imgui_test_engine_implot_enabled}>:implot>)

    try_run(return_value compilation_result SOURCE_FROM_CONTENT check_imgui_app_backends.cpp
[=[
#include <imgui_app.h>

int main() {
  unsigned result = 0;

#if defined(IMGUI_APP_WIN32_DX11)
  result |= (0b1 << 0);
#elif defined(IMGUI_APP_SDL2_GL2)
  result |= (0b1 << 1);
#elif defined(IMGUI_APP_SDL2_GL3)
  result |= (0b1 << 2);
#elif defined(IMGUI_APP_GLFW_GL2)
  result |= (0b1 << 3);
#elif defined(IMGUI_APP_GLFW_GL3)
  result |= (0b1 << 4);
#else
  // Null backend
#endif

  return result;
}
]=] COMPILE_DEFINITIONS "-I${imgui_te_dir}/shared -I${imgui_include_dir}" NO_CACHE NO_LOG)

    add_library(imgui_app)
    target_sources(imgui_app PRIVATE ${_imgui_te_dir}/shared/imgui_app.cpp)
    target_sources(imgui_app PUBLIC FILE_SET imgui_app_set BASE_DIRS ${_imgui_te_dir} TYPE HEADERS FILES ${_imgui_te_dir}/shared/imgui_app.h)

    if(${return_value} EQUAL 1)
      target_link_libraries(imgui_app PRIVATE imgui_backend_win32 imgui_backend_dx11)
    elseif(${return_value} EQUAL 2)
      target_link_libraries(imgui_app PRIVATE imgui_backend_sdl2 imgui_backend_opengl2)
    elseif(${return_value} EQUAL 4)
      target_link_libraries(imgui_app PRIVATE imgui_backend_sdl2 imgui_backend_opengl3)
    elseif(${return_value} EQUAL 8)
      target_link_libraries(imgui_app PRIVATE imgui_backend_glfw imgui_backend_opengl2)
    elseif(${return_value} EQUAL 16)
      target_link_libraries(imgui_app PRIVATE imgui_backend_glfw imgui_backend_opengl3)
    else()
      target_link_libraries(imgui_app PRIVATE imgui_core)
    endif()

    add_executable(app_minimal)
    target_sources(app_minimal PRIVATE ${_imgui_te_dir}/app_minimal/app_minimal_main.cpp ${_imgui_te_dir}/app_minimal/app_minimal_tests.cpp)
    target_link_libraries(app_minimal PRIVATE imgui_app imgui_test_engine imgui_core)

    add_executable(imgui_test_suite)
    target_sources(imgui_test_suite PRIVATE
      ${_imgui_te_dir}/imgui_test_suite/imgui_tests_core.cpp
      ${_imgui_te_dir}/imgui_test_suite/imgui_tests_inputs.cpp
      ${_imgui_te_dir}/imgui_test_suite/imgui_tests_perf.cpp
      ${_imgui_te_dir}/imgui_test_suite/imgui_test_suite.cpp
      ${_imgui_te_dir}/imgui_test_suite/imgui_tests_widgets.cpp
      ${_imgui_te_dir}/imgui_test_suite/imgui_tests_docking.cpp
      ${_imgui_te_dir}/imgui_test_suite/imgui_tests_nav.cpp
      ${_imgui_te_dir}/imgui_test_suite/imgui_tests_tables.cpp
      ${_imgui_te_dir}/imgui_test_suite/imgui_tests_viewports.cpp
      ${_imgui_te_dir}/imgui_test_suite/imgui_tests_widgets_inputtext.cpp)
    target_compile_definitions(imgui_test_suite PRIVATE IMGUI_USER_CONFIG="imgui_test_suite/imgui_test_suite_imconfig.h")

    target_sources(imgui_test_suite PUBLIC FILE_SET HEADERS BASE_DIRS ${_imgui_te_dir}
      FILES
        ${_imgui_te_dir}/imgui_test_suite/imgui_test_suite.h
        ${_imgui_te_dir}/imgui_test_suite/imgui_test_suite_imconfig.h)
    target_link_libraries(imgui_test_suite PRIVATE imgui_test_engine imgui_app imgui_core)
  endif()
endif()

if(imgui_club)
  if(NOT IMGUI_CLUB_SOURCE_DIR)
    message(FATAL_ERROR "set IMGUI_CLUB_SOURCE_DIR to proceed:\ne.g. cmake -DIMGUI_CLUB_SOURCE_DIR=path.to.imgui_club ...")
  endif()
  cmake_path(SET _imclub_dir ${IMGUI_CLUB_SOURCE_DIR})

  if(imgui_memory_editor)
    add_library(imgui_memory_editor INTERFACE)
    target_sources(imgui_memory_editor INTERFACE FILE_SET imgui_memory_editor_set BASE_DIRS ${_imclub_dir}/imgui_memory_editor TYPE HEADERS FILES ${_imclub_dir}/imgui_memory_editor/imgui_memory_editor.h)
  endif()

  if(imgui_multicontext_compositor)
    add_library(imgui_multicontext_compositor)
    target_sources(imgui_multicontext_compositor PRIVATE ${_imclub_dir}/imgui_multicontext_compositor/imgui_multicontext_compositor.cpp)
    target_sources(imgui_multicontext_compositor PUBLIC FILE_SET imgui_multicontext_compositor_set BASE_DIRS ${_imclub_dir}/imgui_multicontext_compositor TYPE HEADERS FILES ${_imclub_dir}/imgui_multicontext_compositor/imgui_multicontext_compositor.h)
    target_link_libraries(imgui_multicontext_compositor PUBLIC imgui_core)

    list(APPEND imgui_thirdparty_list imgui_multicontext_compositor)
 endif()

  if(imgui_threaded_rendering)
    add_library(imgui_threaded_rendering INTERFACE)
    target_sources(imgui_threaded_rendering INTERFACE FILE_SET imgui_threaded_rendering_set BASE_DIRS ${_imclub_dir}/imgui_threaded_rendering TYPE HEADERS FILES ${_imclub_dir}/imgui_threaded_rendering/imgui_threaded_rendering.h)
  endif()
endif()

if(imgui_markdown)
  if(NOT IMGUI_MARKDOWN_SOURCE_DIR)
    message(FATAL_ERROR "set IMGUI_MARKDOWN_SOURCE_DIR to proceed:\ne.g. cmake -DIMGUI_MARKDOWN_SOURCE_DIR=path.to.imgui_markdown...")
  endif()
  cmake_path(SET _immarkdown_dir ${IMGUI_MARKDOWN_SOURCE_DIR})

  add_library(imgui_markdown INTERFACE)
  target_sources(imgui_markdown INTERFACE FILE_SET imgui_markdown_set BASE_DIRS ${_immarkdown_dir} TYPE HEADERS FILES ${_immarkdown_dir}/imgui_markdown.h)
endif()

if(implot)
  if(NOT IMPLOT_SOURCE_DIR)
    message(FATAL_ERROR "set IMPLOT_SOURCE_DIR to proceed:\ne.g. cmake -DIMPLOT_SOURCE_DIR=path.to.implot ...")
  endif()
  cmake_path(SET _implot_dir ${IMPLOT_SOURCE_DIR})

  add_library(implot)
  target_sources(implot PRIVATE ${_implot_dir}/implot.cpp ${_implot_dir}/implot_items.cpp ${_implot_dir}/implot_demo.cpp)
  target_sources(implot PUBLIC FILE_SET implot_set BASE_DIRS ${_implot_dir} TYPE HEADERS FILES ${_implot_dir}/implot.h ${_implot_dir}/implot_internal.h)
  target_link_libraries(implot PUBLIC imgui_core)

  list(APPEND imgui_thirdparty_list implot)
endif()

if(implot3d)
  if(NOT IMPLOT3D_SOURCE_DIR)
    message(FATAL_ERROR "set IMPLOT3D_SOURCE_DIR to proceed:\ne.g. cmake -DIMPLOT3D_SOURCE_DIR=path.to.implot3D ...")
  endif()
  cmake_path(SET _implot3d_dir ${IMPLOT3D_SOURCE_DIR})

  add_library(implot3d)
  target_sources(implot3d PRIVATE ${_implot3d_dir}/implot3d.cpp ${_implot3d_dir}/implot3d_items.cpp ${_implot3d_dir}/implot3d_meshes.cpp ${_implot3d_dir}/implot3d_demo.cpp)
  target_sources(implot3d PUBLIC FILE_SET implot3d_set BASE_DIRS ${_implot3d_dir} TYPE HEADERS FILES ${_implot3d_dir}/implot3d.h ${_implot3d_dir}/implot3d_internal.h)
  target_link_libraries(implot3d PUBLIC imgui_core)

  list(APPEND imgui_thirdparty_list implot3d)
endif()

if(imguifiledialog)
  if(NOT IMGUIFILEDIALOG_SOURCE_DIR)
    message(FATAL_ERROR "set IMGUIFILEDIALOG_SOURCE_DIR to proceed:\ne.g. cmake -DIMGUIFILEDIALOG_SOURCE_DIR=path.to.imguifiledialog...")
  endif()
  cmake_path(SET _imfiledialog_dir ${IMGUIFILEDIALOG_SOURCE_DIR})

  find_path(Dirent_PATH dirent.h PATHS ${_imfiledialog_dir} PATH_SUFFIXES dirent DOC "https://github.com/tronkko/dirent" NO_CACHE REQUIRED NO_DEFAULT_PATH NO_CMAKE_SYSTEM_PATH NO_CMAKE_FIND_ROOT_PATH)
  find_path(stb_PATH std_image.h stb_image_resize2.h PATHS ${_imfiledialog_dir} PATH_SUFFIXES stb DOC "https://github.com/nothings/stb" NO_CACHE REQUIRED NO_DEFAULT_PATH NO_CMAKE_SYSTEM_PATH NO_CMAKE_FIND_ROOT_PATH)

  add_library(imguifiledialog)
  target_sources(imguifiledialog PRIVATE ${_imfiledialog_dir}/ImGuiFileDialog.cpp)
  target_compile_features(imguifiledialog PRIVATE cxx_std_17)
  target_compile_definitions(imguifiledialog PRIVATE USE_STD_FILESYSTEM)
  target_sources(imguifiledialog PUBLIC FILE_SET imguifiledialog_set BASE_DIRS ${_imfiledialog_dir} TYPE HEADERS FILES ${_imfiledialog_dir}/ImGuiFileDialog.h ${_imfiledialog_dir}/ImGuiFileDialogConfig.h)
  target_sources(imguifiledialog PRIVATE FILE_SET HEADERS BASE_DIRS ${stb_PATH} FILES ${stb_PATH}/stb_image.h ${stb_PATH}/stb_image_resize2.h)
  target_sources(imguifiledialog PRIVATE FILE_SET HEADERS BASE_DIRS $<$<PLATFORM_ID:Windows>:${Dirent_PATH}> FILES $<$<PLATFORM_ID:Windows>:${Dirent_PATH}/dirent.h>)
  target_link_libraries(imguifiledialog PUBLIC imgui_core)
endif()

if(pkg-config)
  if(pkg_config_requires_list)
    list(SORT pkg_config_requires_list)
    list(REMOVE_DUPLICATES pkg_config_requires_list)
    string(JOIN " " pkg_config_requires_string ${pkg_config_requires_list})
  endif()

  if(imgui_backends_list)
    set(pkg_config_imgui_backends_list ${imgui_backends_list})
    list(TRANSFORM pkg_config_imgui_backends_list PREPEND "-l")
    string(JOIN " " pkg_config_imgui_backends_string ${pkg_config_imgui_backends_list})
  endif()

  if(imgui_thirdparty_list)
    list(TRANSFORM imgui_thirdparty_list PREPEND "-l")
    string(JOIN " " imgui_thirdparty_string ${imgui_thirdparty_list})
  endif()

  file(CONFIGURE OUTPUT ${PROJECT_BINARY_DIR}/cmakeforimgui.pc CONTENT
[=[
# SPDX-License-Identifier: MIT

prefix=@CMAKE_INSTALL_PREFIX@
exec_prefix=${prefix}
libdir=@CMAKE_INSTALL_FULL_LIBDIR@
includedir=@CMAKE_INSTALL_FULL_INCLUDEDIR@

Name: @PROJECT_NAME@
Description: @PROJECT_DESCRIPTION@
Version: @PROJECT_VERSION@
URL: @PROJECT_HOMEPAGE_URL@

Requires: @pkg_config_requires_string@
Conflicts:
Libs: -L${libdir} -limgui_core @pkg_config_imgui_backends_string@ @imgui_thirdparty_string@
Libs.private:
Cflags: -I${includedir}
]=] @ONLY)

endif()

if(install)
  set_property(GLOBAL PROPERTY INSTALL_PARALLEL ON)

  install(DIRECTORY ${_imgui_dir}/docs DESTINATION ${CMAKE_INSTALL_DATADIR}/Dear\ ImGui)
  install(FILES ${_imgui_dir}/LICENSE.txt DESTINATION ${CMAKE_INSTALL_DATADIR}/Dear\ ImGui)
  install(EXPORT imgui_export DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/CMakeForImGui NAMESPACE Unofficial::DearImGui:: FILE CMakeForImGuiTargets.cmake)
  install(TARGETS imgui_core EXPORT imgui_export FILE_SET imgui_core_set FILE_SET imgui_misc_cpp_set FILE_SET imgui_misc_freetype_set)

  if(misc)
    if(misc_debugger)
      install(DIRECTORY ${_imgui_misc_dir}/debuggers DESTINATION ${CMAKE_INSTALL_DATADIR}/Dear\ ImGui)
    endif()

    if(misc_fonts)
      install(TARGETS binary_to_compressed_c DESTINATION ${CMAKE_INSTALL_DATADIR}/Dear\ ImGui/fonts)
      install(DIRECTORY ${_imgui_dir}/misc/fonts DESTINATION ${CMAKE_INSTALL_DATADIR}/Dear\ ImGui FILES_MATCHING PATTERN "*.ttf")
    endif()
  endif()

  foreach(backend IN LISTS imgui_backends_list)
    get_property(name TARGET ${backend} PROPERTY NAME)
    get_property(file_set TARGET ${backend} PROPERTY HEADER_SETS)
    list(REMOVE_ITEM file_set HEADERS)

    install(TARGETS ${name} EXPORT imgui_export FILE_SET ${file_set})
  endforeach()

  if(install_examples)
    if(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
      set_target_properties(${imgui_examples_list} PROPERTIES MACOSX_BUNDLE TRUE)
    endif()

    install(TARGETS ${imgui_examples_list} BUNDLE DESTINATION Applications RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
  endif()

  if(imgui_test_engine)
    install(EXPORT imgui_test_engine_export DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/CMakeForImGui NAMESPACE Unofficial::imgui_test_engine:: FILE CMakeForImGui-imgui_test_engine-Targets.cmake)

    install(TARGETS imgui_test_engine EXPORT imgui_test_engine_export FILE_SET imgui_test_engine_set)
    install(TARGETS imgui_app EXPORT imgui_test_engine_export FILE_SET imgui_app_set)
    install(TARGETS imgui_test_suite RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

    install(DIRECTORY ${_imgui_te_dir}/docs DESTINATION ${CMAKE_INSTALL_DATADIR}/imgui_test_engine)
    install(DIRECTORY ${_imgui_te_dir}/build_scripts DESTINATION ${CMAKE_INSTALL_DATADIR}/imgui_test_engine)
  endif()

  if(imgui_club)
    install(EXPORT imgui_club_export DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/CMakeForImGui NAMESPACE Unofficial::imgui_club:: FILE CMakeForImGui-imgui_club-Targets.cmake)

    if(imgui_threaded_rendering)
      install(TARGETS imgui_threaded_rendering EXPORT imgui_club_export FILE_SET imgui_threaded_rendering_set)
    endif()

    if(imgui_memory_editor)
      install(TARGETS imgui_memory_editor EXPORT imgui_club_export FILE_SET imgui_memory_editor_set)
    endif()

    if(imgui_multicontext_compositor)
      install(TARGETS imgui_multicontext_compositor EXPORT imgui_club_export FILE_SET imgui_multicontext_compositor_set)
    endif()

    install(FILES ${_imclub_dir}/LICENSE.txt DESTINATION ${CMAKE_INSTALL_DATADIR}/imgui_club)
  endif()

  if(imgui_markdown)
    install(EXPORT imgui_markdown_export DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/CMakeForImGui NAMESPACE Unofficial::imgui_markdown:: FILE CMakeForImGui-imgui_markdown-Targets.cmake)
    install(TARGETS imgui_markdown EXPORT imgui_markdown_export FILE_SET imgui_markdown_set)
    install(FILES ${_immarkdown_dir}/License.txt DESTINATION ${CMAKE_INSTALL_DATADIR}/imgui_markdown)
  endif()

  if(implot)
    install(EXPORT implot_export DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/CMakeForImGui NAMESPACE Unofficial::ImPlot:: FILE CMakeForImGui-ImPlot-Targets.cmake)
    install(TARGETS implot EXPORT implot_export FILE_SET implot_set)
    install(FILES ${_implot_dir}/LICENSE DESTINATION ${CMAKE_INSTALL_DATADIR}/ImPlot)
  endif()

  if(implot3d)
    install(EXPORT implot3d_export DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/CMakeForImGui NAMESPACE Unofficial::ImPlot3D:: FILE CMakeForImGui-ImPlot3D-Targets.cmake)
    install(TARGETS implot3d EXPORT implot3d_export FILE_SET implot3d_set)
    install(FILES ${_implot3d_dir}/LICENSE DESTINATION ${CMAKE_INSTALL_DATADIR}/ImPlot3D)
  endif()

  if(imguifiledialog)
    install(EXPORT imguifiledialog_export DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/CMakeForImGui NAMESPACE Unofficial::ImGuiFileDialog:: FILE CMakeForImGui-ImGuiFileDialog-Targets.cmake)
    install(TARGETS imguifiledialog EXPORT imguifiledialog_export FILE_SET imguifiledialog_set)
    install(FILES ${_imfiledialog_dir}/LICENSE DESTINATION ${CMAKE_INSTALL_DATADIR}/ImGuiFileDialog)
    install(FILES ${_imfiledialog_dir}/Documentation.md DESTINATION ${CMAKE_INSTALL_DATADIR}/ImGuiFileDialog/docs)
  endif()

  if(pkg-config)
    install(FILES ${PROJECT_BINARY_DIR}/cmakeforimgui.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
  endif()

  configure_package_config_file(CMakeForImGuiConfig.cmake.in ${PROJECT_BINARY_DIR}/CMakeForImGuiConfig.cmake INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/CMakeForImGui)
  write_basic_package_version_file(${PROJECT_BINARY_DIR}/CMakeForImGuiConfigVersion.cmake VERSION ${IMGUI_VERSION} COMPATIBILITY ExactVersion)

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

foreach(file IN LISTS files_to_uninstall)
  execute_process(COMMAND @CMAKE_COMMAND@ -E rm $ENV{DESTDIR}${file} COMMAND_ECHO STDOUT)
endforeach()
]=] @ONLY)

  add_custom_target(
    uninstall
    COMMAND ${CMAKE_COMMAND} -P ${PROJECT_BINARY_DIR}/cmake_uninstall.cmake
    WORKING_DIRECTORY ${PROJECT_BINARY_DIR})
endif()

if(visualize_targets)
  if(NOT EXISTS ${PROJECT_BINARY_DIR}/CMakeGraphVizOptions.cmake)
    file(
      WRITE ${PROJECT_BINARY_DIR}/CMakeGraphVizOptions.cmake
[=[
set(GRAPHVIZ_GENERATE_PER_TARGET FALSE)
set(GRAPHVIZ_GENERATE_DEPENDERS FALSE)
]=])
  endif()

  find_program(DOT_COMMAND dot REQUIRED)

  add_custom_target(
    graph
    COMMAND ${CMAKE_COMMAND} --graphviz=targets.dot ${PROJECT_BINARY_DIR}
    COMMAND ${DOT_COMMAND} -Tpng targets.dot -o targets.png
    WORKING_DIRECTORY ${PROJECT_BINARY_DIR}
    VERBATIM)
endif()

set(CPACK_GENERATOR ZIP)
set(CPACK_PACKAGE_FILE_NAME ${PROJECT_NAME}_${PROJECT_VERSION}-DearImGui_${IMGUI_VERSION}-${CMAKE_SYSTEM})
set(CPACK_PACKAGE_DIRECTORY ${PROJECT_BINARY_DIR}/package)

include(CPack)

feature_summary(WHAT ALL)

set(options DearImGui_Backend_Android DearImGui_Backend_Allegro5 DearImGui_Backend_GLFW DearImGui_Backend_FreeGLUT DearImGui_Backend_SDL2 DearImGui_Backend_SDLRenderer2 DearImGui_Backend_SDL3 DearImGui_Backend_SDLGPU3 DearImGui_Backend_SDLRenderer3 DearImGui_Backend_OpenGL2 DearImGui_Backend_OpenGL3 DearImGui_Backend_Vulkan DearImGui_Backend_WebGPU DearImGui_Backend_Win32 DearImGui_Backend_DirectX9 DearImGui_Backend_DirectX10 DearImGui_Backend_DirectX11 DearImGui_Backend_DirectX12 DearImGui_Backend_OSX DearImGui_Backend_Metal
misc
misc_cpp misc_freetype misc_debugger misc_fonts
thirdparty
imgui_test_engine imgui_club imgui_markdown implot implot3d imguifiledialog
DearImGui_Examples
example_null
example_allegro5 example_android_opengl3
example_glfw_opengl2 example_glfw_opengl3 example_glfw_vulkan example_glfw_metal example_glfw_wgpu
example_glut_opengl2
example_sdl2_directx11 example_sdl2_opengl2 example_sdl2_opengl3 example_sdl2_vulkan example_sdl2_metal example_sdl2_sdlrenderer2
example_sdl3_opengl3 example_sdl3_sdlgpu3 example_sdl3_sdlrenderer3 example_sdl3_vulkan example_sdl3_metal
example_win32_directx10 example_win32_directx11 example_win32_directx12 example_win32_directx9 example_win32_opengl3 example_win32_vulkan
example_apple_metal example_apple_opengl2
BUILD_SHARED_LIBS pkg-config install install_examples uninstall visualize_targets)
list(SORT options CASE INSENSITIVE)

message(STATUS "Configuration summary:\n")
foreach(o IN LISTS options)
  message(" * ${o}: ${${o}}")
endforeach()
