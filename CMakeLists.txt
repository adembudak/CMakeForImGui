cmake_minimum_required(VERSION 4.0.0)

project(CMakeForImGui
  VERSION 1.0.0.1
  DESCRIPTION "CMake build support for Dear ImGui"
  HOMEPAGE_URL "https://github.com/adembudak/CMakeForImGui"
  LANGUAGES CXX)

include(CMakeDependentOption)
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)
include(CMakePrintHelpers)
include(FeatureSummary)

get_property(is_generator_multi_config GLOBAL PROPERTY GENERATOR_IS_MULTI_CONFIG)
if(NOT is_generator_multi_config)
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS Debug Release RelWithDebInfo MinSizeRel)
endif()

set(IMGUI_SOURCE_DIR "" CACHE PATH "Path to Dear ImGui source tree")
if(NOT IMGUI_SOURCE_DIR)
  message(FATAL_ERROR "set IMGUI_SOURCE_DIR to proceed:\ne.g. cmake -DIMGUI_SOURCE_DIR=path.to.imgui ...")
endif()

cmake_path(SET _imgui_dir ${IMGUI_SOURCE_DIR})

file(STRINGS ${_imgui_dir}/imgui.h imgui_version_string REGEX "#define[ \t]+IMGUI_VERSION_NUM[ \t]+[0-9]+")
string(REGEX MATCH "[0-9]+" IMGUI_VERSION ${imgui_version_string})

message(STATUS "ImGui version ${IMGUI_VERSION}")

file(READ ${_imgui_dir}/imconfig.h imconfig_h_string)
string(REGEX REPLACE  "//#define (IMGUI_(ENABLE|DISABLE|USER|USE|DEFINE|DEBUG|INCLUDE)[_A-Z0-9]*)" "#cmakedefine \\1" imconfig_h_in_string ${imconfig_h_string})
file(WRITE ${PROJECT_BINARY_DIR}/imconfig.h.in ${imconfig_h_in_string})
configure_file(${PROJECT_BINARY_DIR}/imconfig.h.in ${PROJECT_BINARY_DIR}/imconfig.h)

if(CMAKE_SYSTEM_NAME STREQUAL "Android")
  option(DearImGui_Backend_Android "" OFF)
endif()

option(DearImGui_Backend_Allegro5 "" OFF)
option(DearImGui_Backend_GLFW "" OFF)
option(DearImGui_Backend_FreeGLUT "" OFF)
option(DearImGui_Backend_SDL2 "" OFF)
cmake_dependent_option(DearImGui_Backend_SDLRenderer2 "" OFF "(${IMGUI_VERSION} VERSION_GREATER 18420)" OFF)
cmake_dependent_option(DearImGui_Backend_SDL3 "" OFF "(${IMGUI_VERSION} VERSION_GREATER 19100)" OFF)
cmake_dependent_option(DearImGui_Backend_SDLGPU3 "" OFF "(${IMGUI_VERSION} VERSION_GREATER 19160)" OFF)
cmake_dependent_option(DearImGui_Backend_SDLRenderer3 "" OFF "(${IMGUI_VERSION} VERSION_GREATER 19100)" OFF)
option(DearImGui_Backend_OpenGL2 "" OFF)
option(DearImGui_Backend_OpenGL3 "" OFF)
option(DearImGui_Backend_Vulkan "" OFF)
cmake_dependent_option(DearImGui_Backend_WebGPU "" OFF "(${IMGUI_VERSION} VERSION_GREATER 18000)" OFF)

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

option(BUILD_SHARED_LIBS "" OFF)

option(Pkg-config "Generate dearimgui.pc file" OFF)

option(Install "Install Dear ImGui" ON)
option(Uninstall "A custom target 'uninstall' is generated to undo latest install step" ON)
cmake_dependent_option(Install_examples "Install Dear ImGui examples" OFF Install OFF)

option(visualize_targets "A custom target 'graph' that shows linkage graph enabled to generate a targets.png on build directory" OFF)
mark_as_advanced(FORCE visualize_targets)

cmake_dependent_option(Example_Android_OpenGL3 "" OFF "DearImGui_Backend_Android AND DearImGui_Backend_OpenGL3" OFF)

cmake_dependent_option(Example_GLFW_OpenGL2 "" OFF "DearImGui_Backend_GLFW AND DearImGui_Backend_OpenGL2" OFF)
cmake_dependent_option(Example_GLFW_OpenGL3 "" OFF "DearImGui_Backend_GLFW AND DearImGui_Backend_OpenGL3" OFF)
cmake_dependent_option(Example_GLFW_Vulkan "" OFF "DearImGui_Backend_GLFW AND DearImGui_Backend_Vulkan" OFF)
cmake_dependent_option(Example_GLFW_Metal "" OFF "DearImGui_Backend_GLFW AND DearImGui_Backend_Metal" OFF)
cmake_dependent_option(Example_GLFW_WebGPU "" OFF "DearImGui_Backend_GLFW AND DearImGui_Backend_WebGPU" OFF)

cmake_dependent_option(Example_FreeGLUT_OpenGL2 "" OFF "DearImGui_Backend_FreeGLUT AND DearImGui_Backend_OpenGL2" OFF)

cmake_dependent_option(Example_SDL2_OpenGL2 "" OFF "DearImGui_Backend_SDL2 AND DearImGui_Backend_OpenGL2" OFF)
cmake_dependent_option(Example_SDL2_OpenGL3 "" OFF "DearImGui_Backend_SDL2 AND DearImGui_Backend_OpenGL3" OFF)
cmake_dependent_option(Example_SDL2_Vulkan "" OFF "DearImGui_Backend_SDL2 AND DearImGui_Backend_Vulkan" OFF)
cmake_dependent_option(Example_SDL2_DirectX11 "" OFF "DearImGui_Backend_SDL2 AND DearImGui_Backend_DirectX11" OFF)
cmake_dependent_option(Example_SDL2_Metal "" OFF "DearImGui_Backend_SDL2 AND DearImGui_Backend_Metal" OFF)
cmake_dependent_option(Example_SDL2_SDLRenderer2 "" OFF "DearImGui_Backend_SDLRenderer2 AND DearImGui_Backend_SDL2" OFF)

cmake_dependent_option(Example_Allegro5 "" OFF DearImGui_Backend_Allegro5 OFF)

cmake_dependent_option(Example_SDL3_OpenGL3 "" OFF "DearImGui_Backend_SDL3 AND DearImGui_Backend_OpenGL3" OFF)
cmake_dependent_option(Example_SDL3_Vulkan "" OFF "DearImGui_Backend_SDL3 AND DearImGui_Backend_Vulkan AND (${IMGUI_VERSION} VERSION_GREATER 19140)" OFF)
cmake_dependent_option(Example_SDL3_Metal "" OFF "DearImGui_Backend_SDL3 AND DearImGui_Backend_Metal AND (${IMGUI_VERSION} VERSION_GREATER 19210)" OFF)
cmake_dependent_option(Example_SDL3_SDLGPU3 "" OFF DearImGui_Backend_SDLGPU3 OFF)
cmake_dependent_option(Example_SDL3_SDLRenderer3 "" OFF "DearImGui_Backend_SDLRenderer3 AND DearImGui_Backend_SDL3" OFF)

cmake_dependent_option(Example_Win32_OpenGL3 "" OFF "DearImGui_Backend_Win32 AND DearImGui_Backend_OpenGL3 AND (${IMGUI_VERSION} VERSION_GREATER 18950)" OFF)
cmake_dependent_option(Example_Win32_Vulkan "" OFF "DearImGui_Backend_Win32 AND DearImGui_Backend_Vulkan AND (${IMGUI_VERSION} VERSION_GREATER 19160)" OFF)
cmake_dependent_option(Example_Win32_DirectX9 "" OFF "DearImGui_Backend_Win32 AND DearImGui_Backend_DirectX9" OFF)
cmake_dependent_option(Example_Win32_DirectX10 "" OFF "DearImGui_Backend_Win32 AND DearImGui_Backend_DirectX10" OFF)
cmake_dependent_option(Example_Win32_DirectX11 "" OFF "DearImGui_Backend_Win32 AND DearImGui_Backend_DirectX11" OFF)
cmake_dependent_option(Example_Win32_DirectX12 "" OFF "DearImGui_Backend_Win32 AND DearImGui_Backend_DirectX12" OFF)

cmake_dependent_option(Example_Apple_OpenGL2 "" OFF "DearImGui_Backend_OSX AND DearImGui_Backend_OpenGL2" OFF)
cmake_dependent_option(Example_Apple_Metal "" OFF "DearImGui_Backend_OSX AND DearImGui_Backend_Metal" OFF)
option(Example_NULL "" OFF)

set(imgui_backends_list)
set(imgui_examples_list)
set(pkg_config_requires_list)

set(IMGUI_USER_CONFIG "" CACHE FILEPATH "File path to user specified configuration file (relative to imgui/ folder)")
set(IMGUI_USER_H_FILENAME "" CACHE FILEPATH "")
set(IMGUI_STB_TRUETYPE_FILENAME "" CACHE FILEPATH "")
set(IMGUI_STB_RECT_PACK_FILENAME "" CACHE FILEPATH "")
set(IMGUI_STB_SPRINTF_FILENAME "" CACHE FILEPATH "")

add_library(imgui_core)
target_sources(imgui_core
  PRIVATE ${_imgui_dir}/imgui.cpp ${_imgui_dir}/imgui_draw.cpp ${_imgui_dir}/imgui_tables.cpp ${_imgui_dir}/imgui_widgets.cpp ${_imgui_dir}/imgui_demo.cpp
  PUBLIC FILE_SET imgui_core_set
  BASE_DIRS ${_imgui_dir}
  TYPE HEADERS
  FILES ${_imgui_dir}/imgui.h ${_imgui_dir}/imgui_internal.h ${_imgui_dir}/imstb_rectpack.h ${_imgui_dir}/imstb_textedit.h ${_imgui_dir}/imstb_truetype.h
  PUBLIC FILE_SET HEADERS
  BASE_DIRS ${PROJECT_BINARY_DIR}
  FILES ${PROJECT_BINARY_DIR}/imconfig.h
  PRIVATE ${_imgui_dir}/misc/cpp/imgui_stdlib.cpp
  PUBLIC FILE_SET imgui_misc_cpp_set
  BASE_DIRS ${_imgui_dir}/misc/cpp
  TYPE HEADERS
  FILES ${_imgui_dir}/misc/cpp/imgui_stdlib.h)
target_link_libraries(imgui_core PRIVATE "$<$<AND:$<PLATFORM_ID:Darwin>,$<BOOL:${IMGUI_ENABLE_OSX_DEFAULT_CLIPBOARD_FUNCTIONS}>>:-framework ApplicationServices>")
target_compile_features(imgui_core PUBLIC cxx_std_11)

if(IMGUI_USER_CONFIG) # this and imgui_config_FILENAME file paths below must be relative to imgui/ folder
  target_compile_definitions(imgui_core PUBLIC IMGUI_USER_CONFIG="${IMGUI_USER_CONFIG}")
endif()

if(IMGUI_USER_H_FILENAME)
  target_compile_definitions(imgui_core PRIVATE IMGUI_INCLUDE_IMGUI_USER_H IMGUI_USER_H_FILENAME="${IMGUI_USER_H_FILENAME}")
endif()

if(IMGUI_STB_TRUETYPE_FILENAME)
  target_compile_definitions(imgui_core PRIVATE IMGUI_STB_TRUETYPE_FILENAME="${IMGUI_STB_TRUETYPE_FILENAME}")
endif()

if(IMGUI_STB_RECT_PACK_FILENAME)
  target_compile_definitions(imgui_core PRIVATE IMGUI_STB_RECT_PACK_FILENAME="${IMGUI_STB_RECT_PACK_FILENAME}")
endif()

if(IMGUI_STB_SPRINTF_FILENAME)
  target_compile_definitions(imgui_core PRIVATE IMGUI_STB_SPRINTF_FILENAME="${IMGUI_STB_SPRINTF_FILENAME}")
endif()

if(IMGUI_ENABLE_FREETYPE)
  find_package(freetype CONFIG REQUIRED)

  target_sources(imgui_core
    PRIVATE ${_imgui_dir}/misc/freetype/imgui_freetype.cpp
    PUBLIC FILE_SET imgui_misc_freetype_set
    BASE_DIRS ${_imgui_dir}/misc/freetype
    TYPE HEADERS
    FILES ${_imgui_dir}/misc/freetype/imgui_freetype.h)
  target_link_libraries(imgui_core PUBLIC Freetype::Freetype)

  if(IMGUI_ENABLE_FREETYPE_LUNASVG)
    if(${IMGUI_VERSION} VERSION_LESS 19200)
      find_package(lunasvg 2.4.3 EXACT CONFIG REQUIRED)
    else()
      find_package(lunasvg CONFIG REQUIRED)
    endif()
    target_link_libraries(imgui_core PUBLIC lunasvg::lunasvg)
  elseif(IMGUI_ENABLE_FREETYPE_PLUTOSVG)
    find_package(plutosvg CONFIG REQUIRED)
    target_link_libraries(imgui_core PUBLIC plutosvg::plutosvg)
  endif()

  list(APPEND pkg_config_requires_list freetype2)
endif()

add_executable(binary_to_compressed_c)
target_sources(binary_to_compressed_c PRIVATE ${_imgui_dir}/misc/fonts/binary_to_compressed_c.cpp)
target_compile_features(binary_to_compressed_c PRIVATE cxx_std_11)

if(DearImGui_Backend_OpenGL2)
  if(CMAKE_SYSTEM_NAME STREQUAL "Windows" AND CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
  else()
    find_package(OpenGL 2 QUIET REQUIRED)
  endif()

  add_library(imgui_impl_opengl2)
  target_sources(imgui_impl_opengl2
    PRIVATE ${_imgui_dir}/backends/imgui_impl_opengl2.cpp
    PUBLIC FILE_SET imgui_impl_opengl2_set
    BASE_DIRS ${_imgui_dir}/backends
    TYPE HEADERS
    FILES ${_imgui_dir}/backends/imgui_impl_opengl2.h)
  target_link_libraries(imgui_impl_opengl2
    PUBLIC imgui_core
    PUBLIC $<IF:$<AND:$<PLATFORM_ID:Windows>,$<CXX_COMPILER_ID:GNU,Clang>>,opengl32,OpenGL::GL>)

  list(APPEND imgui_backends_list imgui_impl_opengl2)
  list(APPEND pkg_config_requires_list opengl)
endif()

if(DearImGui_Backend_OpenGL3)
  if(CMAKE_SYSTEM_NAME STREQUAL "Windows" AND CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
  else()
    include(FindOpenGL)
  endif()

  if(${IMGUI_VERSION} VERSION_LESS 18400)
    find_package(GLEW REQUIRED)
  endif()

  add_library(imgui_impl_opengl3)
  target_sources(imgui_impl_opengl3
    PRIVATE ${_imgui_dir}/backends/imgui_impl_opengl3.cpp
    PUBLIC FILE_SET imgui_impl_opengl3_set
    BASE_DIRS ${_imgui_dir}/backends
    TYPE HEADERS
    FILES ${_imgui_dir}/backends/imgui_impl_opengl3.h)
  target_link_libraries(imgui_impl_opengl3
    PUBLIC imgui_core
    PUBLIC $<IF:$<AND:$<PLATFORM_ID:Windows>,$<CXX_COMPILER_ID:GNU,Clang>>,opengl32,$<IF:$<TARGET_EXISTS:imgui_impl_android>,EGL,OpenGL::GL>>
    PUBLIC $<$<VERSION_LESS:${IMGUI_VERSION},18400>:GLEW::GLEW>)
  target_compile_definitions(imgui_impl_opengl3 PUBLIC $<$<VERSION_LESS:${IMGUI_VERSION},18400>:IMGUI_IMPL_OPENGL_LOADER_GLEW>)

  list(APPEND imgui_backends_list imgui_impl_opengl3)
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

  add_library(imgui_impl_vulkan)
  target_sources(imgui_impl_vulkan
    PRIVATE ${_imgui_dir}/backends/imgui_impl_vulkan.cpp
    PUBLIC FILE_SET imgui_impl_vulkan_set
    BASE_DIRS ${_imgui_dir}/backends
    TYPE HEADERS
    FILES ${_imgui_dir}/backends/imgui_impl_vulkan.h)
  target_link_libraries(imgui_impl_vulkan
    PUBLIC imgui_core
    PUBLIC $<IF:$<AND:$<PLATFORM_ID:Windows>,$<CXX_COMPILER_ID:GNU,Clang>>,Vulkan::Headers ${Vulkan_Libraries},Vulkan::Vulkan>)

  list(APPEND imgui_backends_list imgui_impl_vulkan)
  list(APPEND pkg_config_requires_list vulkan)
endif()

if(DearImGui_Backend_WebGPU)
  add_library(imgui_impl_wgpu)
  target_sources(imgui_impl_wgpu
    PRIVATE ${_imgui_dir}/backends/imgui_impl_wgpu.cpp
    PUBLIC FILE_SET imgui_impl_wgpu_set
    BASE_DIRS ${_imgui_dir}/backends
    TYPE HEADERS
    FILES ${_imgui_dir}/backends/imgui_impl_wgpu.h)

  target_link_libraries(imgui_impl_wgpu PUBLIC imgui_core)

  if(CMAKE_SYSTEM_NAME STREQUAL "Emscripten")
  else()
    find_package(Threads REQUIRED)
    find_package(Dawn REQUIRED)

    target_link_libraries(imgui_impl_wgpu
      PUBLIC dawn::webgpu_dawn dawn::dawn_public_config Threads::Threads)
    target_compile_definitions(imgui_impl_wgpu PUBLIC IMGUI_IMPL_WEBGPU_BACKEND_DAWN)
  endif()

  list(APPEND imgui_backends_list imgui_impl_wgpu)
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

  add_library(imgui_impl_android)
  target_sources(imgui_impl_android
    PRIVATE ${_imgui_dir}/backends/imgui_impl_android.cpp
    PUBLIC FILE_SET imgui_impl_android_set
    BASE_DIRS ${_imgui_dir}/backends
    TYPE HEADERS
    FILES ${_imgui_dir}/backends/imgui_impl_android.h
    PRIVATE FILE_SET HEADERS
    BASE_DIRS ${Android_NDK_INCLUDE_DIR}
    FILES ${Android_NDK_INCLUDE_DIR}/android/native_window.h ${Android_NDK_INCLUDE_DIR}/android/input.h ${Android_NDK_INCLUDE_DIR}/android/keycodes.h ${Android_NDK_INCLUDE_DIR}/android/log.h)
  target_link_directories(imgui_impl_android PUBLIC ${Android_NDK_LIB_DIR})
  target_link_libraries(imgui_impl_android
    PUBLIC imgui_core
    PUBLIC log android)

  list(APPEND imgui_backends_list imgui_impl_android)

  if(Example_Android_OpenGL3)
    find_program(GRADLE_COMMAND gradle REQUIRED)

    cmake_path(SET android_example_output_path ${_imgui_dir}/examples/example_android_opengl3/android/app/build/outputs/apk)
    cmake_path(ABSOLUTE_PATH android_example_output_path NORMALIZE)

    add_custom_command(TARGET imgui_impl_android POST_BUILD
      COMMAND ${GRADLE_COMMAND} --project-dir ${_imgui_dir}/examples/example_android_opengl3/android assembleRelease
      WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
      COMMENT "Building .apk files. Output will be on: ${android_example_output_path}"
      VERBATIM)
  endif()
endif()

if(DearImGui_Backend_Allegro5)
  find_package(Allegro CONFIG REQUIRED)

  add_library(imgui_impl_allegro5)
  target_sources(imgui_impl_allegro5
    PRIVATE ${_imgui_dir}/backends/imgui_impl_allegro5.cpp
    PUBLIC FILE_SET imgui_impl_allegro5_set
    BASE_DIRS ${_imgui_dir}/backends
    TYPE HEADERS
    FILES ${_imgui_dir}/backends/imgui_impl_allegro5.h)
  target_link_libraries(imgui_impl_allegro5
    PUBLIC imgui_core
    PUBLIC Allegro::allegro Allegro::allegro_primitives INTERFACE Allegro::allegro_main)

  list(APPEND imgui_backends_list imgui_impl_allegro5)
  list(APPEND pkg_config_requires_list allegro-5 allegro_primitives-5)

  if(Example_Allegro5)
    add_executable(example_allegro5)
    target_sources(example_allegro5 PRIVATE ${_imgui_dir}/examples/example_allegro5/main.cpp)
    target_link_libraries(example_allegro5
      PRIVATE imgui_impl_allegro5
      PRIVATE "$<$<PLATFORM_ID:Darwin>:-framework OpenGL -framework Carbon -framework ApplicationServices>")

    list(APPEND imgui_examples_list example_allegro5)
  endif()
endif()

if(DearImGui_Backend_GLFW)
  if(CMAKE_SYSTEM_NAME STREQUAL "Emscripten")
  else()
    find_package(glfw3 CONFIG REQUIRED)
  endif()

  add_library(imgui_impl_glfw)
  target_sources(imgui_impl_glfw
    PRIVATE ${_imgui_dir}/backends/imgui_impl_glfw.cpp
    PUBLIC FILE_SET imgui_impl_glfw_set
    BASE_DIRS ${_imgui_dir}/backends
    TYPE HEADERS
    FILES ${_imgui_dir}/backends/imgui_impl_glfw.h)
  target_link_libraries(imgui_impl_glfw
    PUBLIC imgui_core
    PUBLIC $<$<NOT:$<PLATFORM_ID:Emscripten>>:glfw>)

  list(APPEND imgui_backends_list imgui_impl_glfw)
  list(APPEND pkg_config_requires_list glfw3)

  if(Example_GLFW_OpenGL3)
    add_executable(example_glfw_opengl3)
    target_sources(example_glfw_opengl3 PRIVATE ${_imgui_dir}/examples/example_glfw_opengl3/main.cpp)
    target_compile_options(example_glfw_opengl3 PRIVATE $<$<PLATFORM_ID:Emscripten>:--use-port=contrib.glfw3>)
    target_link_options(example_glfw_opengl3 PRIVATE $<$<PLATFORM_ID:Emscripten>:--use-port=contrib.glfw3>)
    target_link_libraries(example_glfw_opengl3 PRIVATE imgui_impl_glfw imgui_impl_opengl3)

    if(CMAKE_SYSTEM_NAME STREQUAL "Emscripten")
      set_target_properties(example_glfw_opengl3 PROPERTIES SUFFIX ".html")
    endif() # $ emrun example_glfw_opengl3.html

    list(APPEND imgui_examples_list example_glfw_opengl3)
  endif()

  if(Example_GLFW_OpenGL2)
    add_executable(example_glfw_opengl2)
    target_sources(example_glfw_opengl2 PRIVATE ${_imgui_dir}/examples/example_glfw_opengl2/main.cpp)
    target_link_libraries(example_glfw_opengl2 PRIVATE imgui_impl_glfw imgui_impl_opengl2)

    list(APPEND imgui_examples_list example_glfw_opengl2)
  endif()

  if(Example_GLFW_Vulkan)
    add_executable(example_glfw_vulkan)
    target_sources(example_glfw_vulkan PRIVATE ${_imgui_dir}/examples/example_glfw_vulkan/main.cpp)
    target_link_libraries(example_glfw_vulkan PRIVATE imgui_impl_glfw imgui_impl_vulkan)

    list(APPEND imgui_examples_list example_glfw_vulkan)
  endif()

  if(Example_GLFW_Metal)
    add_executable(example_glfw_metal)
    target_sources(example_glfw_metal PRIVATE ${_imgui_dir}/examples/example_glfw_metal/main.mm)
    target_link_libraries(example_glfw_metal
      PRIVATE imgui_impl_glfw imgui_impl_metal
      PRIVATE "$<$<PLATFORM_ID:Darwin>:-framework QuartzCore>")

    list(APPEND imgui_examples_list example_glfw_metal)
  endif()

  if(Example_GLFW_WebGPU)
  # WebGPU support must be enabled on used web browser:
    # On Chrome: Go to the address chrome://flags/ and toggle 'Unsafe WebGPU Support' to 'Enabled'
    # On Firefox Nightly: Go to the address about:config, search as WebGPU and make sure 'dom.webgpu.enabled' set to true
    # On Safari: Go Settings > Advanced > check 'Show features for web developers' and then on Feature Flags > check 'WebGPU'

    # Known to work with Emscripten 4.0.10 with a deprecation notice
    # Can be installed with:
    # $ emsdk install 4.0.10
    # $ emsdk activate 4.0.10
    # https://emscripten.org/docs/getting_started/downloads.html

    add_executable(example_glfw_wgpu)
    target_sources(example_glfw_wgpu PRIVATE ${_imgui_dir}/examples/example_glfw_wgpu/main.cpp)
    target_compile_features(example_glfw_wgpu PRIVATE cxx_std_17)
    target_compile_options(example_glfw_wgpu PRIVATE --use-port=contrib.glfw3 -sUSE_WEBGPU) #--use-port=emdawnwebgpu)
    target_link_options(example_glfw_wgpu PRIVATE --use-port=contrib.glfw3 -sUSE_WEBGPU) #--use-port=emdawnwebgpu)
    target_link_libraries(example_glfw_wgpu PRIVATE imgui_impl_glfw imgui_impl_wgpu)
    set_target_properties(example_glfw_wgpu PROPERTIES OUTPUT_NAME index)
    add_custom_command(TARGET example_glfw_wgpu
      POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy ${_imgui_dir}/examples/example_glfw_wgpu/web/index.html $<TARGET_FILE_DIR:example_glfw_wgpu>/example_glfw_wgpu.html)
  endif() # $ emrun build.directory/example_glfw_wgpu.html
endif()

if(DearImGui_Backend_FreeGLUT)
  find_package(FreeGLUT CONFIG REQUIRED)

  add_library(imgui_impl_glut)
  target_sources(imgui_impl_glut
    PRIVATE ${_imgui_dir}/backends/imgui_impl_glut.cpp
    PUBLIC FILE_SET imgui_impl_glut_set
    BASE_DIRS ${_imgui_dir}/backends
    TYPE HEADERS
    FILES ${_imgui_dir}/backends/imgui_impl_glut.h)
  target_link_libraries(imgui_impl_glut
    PUBLIC imgui_core
    PUBLIC $<IF:$<TARGET_EXISTS:FreeGLUT::freeglut>,FreeGLUT::freeglut,FreeGLUT::freeglut_static>)

  list(APPEND imgui_backends_list imgui_impl_glut)
  list(APPEND pkg_config_requires_list glut)

  if(Example_FreeGLUT_OpenGL2)
    add_executable(example_glut_opengl2)
    target_sources(example_glut_opengl2 PRIVATE ${_imgui_dir}/examples/example_glut_opengl2/main.cpp)
    target_link_libraries(example_glut_opengl2 PRIVATE imgui_impl_glut imgui_impl_opengl2)

    list(APPEND imgui_examples_list example_glut_opengl2)
  endif()
endif()

if(DearImGui_Backend_SDL2)
  if(NOT CMAKE_SYSTEM_NAME STREQUAL "Emscripten")
    find_package(SDL2 CONFIG REQUIRED)
  endif()

  add_library(imgui_impl_sdl2)
  target_sources(imgui_impl_sdl2
    PRIVATE $<IF:$<VERSION_GREATER:${IMGUI_VERSION},18920>,${_imgui_dir}/backends/imgui_impl_sdl2.cpp,${_imgui_dir}/backends/imgui_impl_sdl.cpp>
    PUBLIC FILE_SET imgui_impl_sdl2_set
    BASE_DIRS ${_imgui_dir}/backends
    TYPE HEADERS
    FILES $<IF:$<VERSION_GREATER:${IMGUI_VERSION},18920>,${_imgui_dir}/backends/imgui_impl_sdl2.h,${_imgui_dir}/backends/imgui_impl_sdl.h>)
  target_link_libraries(imgui_impl_sdl2
    PUBLIC imgui_core
    PUBLIC $<$<NOT:$<PLATFORM_ID:Emscripten>>:$<IF:$<TARGET_EXISTS:SDL2::SDL2>,SDL2::SDL2,SDL2::SDL2-static>>)
  target_compile_options(imgui_impl_sdl2 PUBLIC $<$<PLATFORM_ID:Emscripten>:--use-port=sdl2>)
  target_link_options(imgui_impl_sdl2 PUBLIC $<$<PLATFORM_ID:Emscripten>:--use-port=sdl2>)

  list(APPEND imgui_backends_list imgui_impl_sdl2)
  list(APPEND pkg_config_requires_list sdl2)

  if(Example_SDL2_OpenGL2)
    add_executable(example_sdl2_opengl2)
    target_sources(example_sdl2_opengl2 PRIVATE $<IF:$<VERSION_GREATER:${IMGUI_VERSION},18920>,${_imgui_dir}/examples/example_sdl2_opengl2/main.cpp,${_imgui_dir}/examples/example_sdl_opengl2/main.cpp>)
    target_link_libraries(example_sdl2_opengl2
      PRIVATE imgui_impl_sdl2 imgui_impl_opengl2
      PRIVATE $<$<PLATFORM_ID:Windows>:SDL2::SDL2main>)

    list(APPEND imgui_examples_list example_sdl2_opengl2)
  endif()

  if(Example_SDL2_OpenGL3)
    add_executable(example_sdl2_opengl3)
    target_sources(example_sdl2_opengl3 PRIVATE $<IF:$<VERSION_GREATER:${IMGUI_VERSION},18920>,${_imgui_dir}/examples/example_sdl2_opengl3/main.cpp,${_imgui_dir}/examples/example_sdl_opengl3/main.cpp>)
    target_link_libraries(example_sdl2_opengl3
      PRIVATE imgui_impl_sdl2 imgui_impl_opengl3
      PRIVATE $<$<PLATFORM_ID:Windows>:SDL2::SDL2main>)

    if(CMAKE_SYSTEM_NAME STREQUAL "Emscripten")
      set_target_properties(example_sdl2_opengl3 PROPERTIES SUFFIX ".html")
    endif()

    list(APPEND imgui_examples_list example_sdl2_opengl3)
  endif()

  if(Example_SDL2_Vulkan)
    add_executable(example_sdl2_vulkan)
    target_sources(example_sdl2_vulkan PRIVATE $<IF:$<VERSION_GREATER:${IMGUI_VERSION},18920>,${_imgui_dir}/examples/example_sdl2_vulkan/main.cpp,${_imgui_dir}/examples/example_sdl_vulkan/main.cpp>)
    target_link_libraries(example_sdl2_vulkan
      PRIVATE imgui_impl_sdl2 imgui_impl_vulkan
      PRIVATE $<$<PLATFORM_ID:Windows>:SDL2::SDL2main>)

    list(APPEND imgui_examples_list example_sdl2_vulkan)
  endif()

  if(Example_SDL2_DirectX11)
    add_executable(example_sdl2_directx11)
    target_sources(example_sdl2_directx11 PRIVATE $<IF:$<VERSION_GREATER:${IMGUI_VERSION},18920>,${_imgui_dir}/examples/example_sdl2_directx11/main.cpp,${_imgui_dir}/examples/example_sdl_directx11/main.cpp>)
    target_link_libraries(example_sdl2_directx11
      PRIVATE imgui_impl_sdl2 imgui_impl_dx11
      PRIVATE $<$<PLATFORM_ID:Windows>:SDL2::SDL2main>)

    list(APPEND imgui_examples_list example_sdl2_directx11)
  endif()

  if(Example_SDL2_Metal)
    add_executable(example_sdl2_metal)
    target_sources(example_sdl2_metal PRIVATE $<IF:$<VERSION_GREATER:${IMGUI_VERSION},18920>,${_imgui_dir}/examples/example_sdl2_metal/main.mm,${_imgui_dir}/examples/example_sdl_metal/main.mm>)
    target_link_libraries(example_sdl2_metal PRIVATE imgui_impl_sdl2 imgui_impl_metal)

    list(APPEND imgui_examples_list example_sdl2_metal)
  endif()
endif()

if(DearImGui_Backend_SDLRenderer2)
  find_package(SDL2 CONFIG REQUIRED)

  add_library(imgui_impl_sdlrenderer2)
  target_sources(imgui_impl_sdlrenderer2
    PRIVATE $<IF:$<VERSION_GREATER:${IMGUI_VERSION},18950>,${_imgui_dir}/backends/imgui_impl_sdlrenderer2.cpp,${_imgui_dir}/backends/imgui_impl_sdlrenderer.cpp>
    PUBLIC FILE_SET imgui_impl_sdlrenderer2_set
    BASE_DIRS ${_imgui_dir}/backends
    TYPE HEADERS
    FILES $<IF:$<VERSION_GREATER:${IMGUI_VERSION},18950>,${_imgui_dir}/backends/imgui_impl_sdlrenderer2.h,${_imgui_dir}/backends/imgui_impl_sdlrenderer.h>)
  target_link_libraries(imgui_impl_sdlrenderer2
    PUBLIC imgui_core
    PUBLIC $<IF:$<TARGET_EXISTS:SDL2::SDL2>,SDL2::SDL2,SDL2::SDL2-static>)

  list(APPEND imgui_backends_list imgui_impl_sdlrenderer2)
  list(APPEND pkg_config_requires_list sdl2)

  if(Example_SDL2_SDLRenderer2)
    add_executable(example_sdl2_sdlrenderer2)
    target_sources(example_sdl2_sdlrenderer2 PRIVATE $<IF:$<VERSION_GREATER:${IMGUI_VERSION},18950>,${_imgui_dir}/examples/example_sdl2_sdlrenderer2/main.cpp,$<IF:$<VERSION_GREATER:${IMGUI_VERSION},18920>,${_imgui_dir}/examples/example_sdl2_sdlrenderer/main.cpp,${_imgui_dir}/examples/example_sdl_sdlrenderer/main.cpp>>)
    target_link_libraries(example_sdl2_sdlrenderer2
      PRIVATE imgui_impl_sdlrenderer2 imgui_impl_sdl2
      PRIVATE $<$<PLATFORM_ID:Windows>:SDL2::SDL2main>)

    list(APPEND imgui_examples_list example_sdl2_sdlrenderer2)
  endif()
endif()

if(DearImGui_Backend_SDL3)
  find_package(SDL3 CONFIG REQUIRED)

  add_library(imgui_impl_sdl3)
  target_sources(imgui_impl_sdl3
    PRIVATE ${_imgui_dir}/backends/imgui_impl_sdl3.cpp
    PUBLIC FILE_SET imgui_impl_sdl3_set
    BASE_DIRS ${_imgui_dir}/backends
    TYPE HEADERS
    FILES ${_imgui_dir}/backends/imgui_impl_sdl3.h)
  target_link_libraries(imgui_impl_sdl3
    PUBLIC imgui_core
    PUBLIC SDL3::SDL3)
  target_compile_definitions(imgui_impl_sdl3 PUBLIC $<$<VERSION_LESS:${IMGUI_VERSION},19130>:SDL_ENABLE_OLD_NAMES>)

  list(APPEND imgui_backends_list imgui_impl_sdl3)
  list(APPEND pkg_config_requires_list sdl3)

  if(Example_SDL3_OpenGL3)
    add_executable(example_sdl3_opengl3)
    target_sources(example_sdl3_opengl3 PRIVATE ${_imgui_dir}/examples/example_sdl3_opengl3/main.cpp)
    target_link_libraries(example_sdl3_opengl3 PRIVATE imgui_impl_sdl3 imgui_impl_opengl3)
    target_compile_definitions(example_sdl3_opengl3 PRIVATE $<$<VERSION_LESS:${IMGUI_VERSION},19130>:SDL_INIT_TIMER=0x00000001u>)

    list(APPEND imgui_examples_list example_sdl3_opengl3)
  endif()

  if(Example_SDL3_Vulkan)
    add_executable(example_sdl3_vulkan)
    target_sources(example_sdl3_vulkan PRIVATE ${_imgui_dir}/examples/example_sdl3_vulkan/main.cpp)
    target_link_libraries(example_sdl3_vulkan PRIVATE imgui_impl_sdl3 imgui_impl_vulkan)

    list(APPEND imgui_examples_list example_sdl3_vulkan)
  endif()

  if(Example_SDL3_Metal)
    add_executable(example_sdl3_metal)
    target_sources(example_sdl3_metal PRIVATE ${_imgui_dir}/examples/example_sdl3_metal/main.mm)
    target_link_libraries(example_sdl3_metal PRIVATE imgui_impl_sdl3 imgui_impl_metal)

    list(APPEND imgui_examples_list example_sdl3_metal)
  endif()
endif()

if(DearImGui_Backend_SDLGPU3)
  find_package(SDL3 CONFIG REQUIRED)

  add_library(imgui_impl_sdlgpu3)
  target_sources(imgui_impl_sdlgpu3
    PRIVATE ${_imgui_dir}/backends/imgui_impl_sdlgpu3.cpp
    PUBLIC FILE_SET imgui_impl_sdlgpu3_set
    BASE_DIRS ${_imgui_dir}/backends
    TYPE HEADERS
    FILES ${_imgui_dir}/backends/imgui_impl_sdlgpu3.h ${_imgui_dir}/backends/imgui_impl_sdlgpu3_shaders.h)
  target_link_libraries(imgui_impl_sdlgpu3
    PUBLIC imgui_core
    PUBLIC SDL3::SDL3)

  list(APPEND imgui_backends_list imgui_impl_sdlgpu3)
  list(APPEND pkg_config_requires_list sdl3)

  if(Example_SDL3_SDLGPU3)
    add_executable(example_sdl3_sdlgpu3)
    target_sources(example_sdl3_sdlgpu3 PRIVATE ${_imgui_dir}/examples/example_sdl3_sdlgpu3/main.cpp)
    target_link_libraries(example_sdl3_sdlgpu3 PRIVATE imgui_impl_sdlgpu3 imgui_impl_sdl3)

    list(APPEND imgui_examples_list example_sdl3_sdlgpu3)
  endif()
endif()

if(DearImGui_Backend_SDLRenderer3)
  find_package(SDL3 CONFIG REQUIRED)

  add_library(imgui_impl_sdlrenderer3)
  target_sources(imgui_impl_sdlrenderer3
    PRIVATE ${_imgui_dir}/backends/imgui_impl_sdlrenderer3.cpp
    PUBLIC FILE_SET imgui_impl_sdlrenderer3_set
    BASE_DIRS ${_imgui_dir}/backends
    TYPE HEADERS
    FILES ${_imgui_dir}/backends/imgui_impl_sdlrenderer3.h)
  target_link_libraries(imgui_impl_sdlrenderer3
    PUBLIC imgui_impl_sdl3
    PUBLIC SDL3::SDL3)

  list(APPEND imgui_backends_list imgui_impl_sdlrenderer3)
  list(APPEND pkg_config_requires_list sdl3)

  if(Example_SDL3_SDLRenderer3)
    add_executable(example_sdl3_sdlrenderer3)
    target_sources(example_sdl3_sdlrenderer3 PRIVATE ${_imgui_dir}/examples/example_sdl3_sdlrenderer3/main.cpp)
    target_link_libraries(example_sdl3_sdlrenderer3 PRIVATE imgui_impl_sdlrenderer3 imgui_impl_sdl3)
    target_compile_definitions(example_sdl3_sdlrenderer3 PRIVATE $<$<VERSION_LESS:${IMGUI_VERSION},19130>:SDL_INIT_TIMER=0x00000001u>)

    list(APPEND imgui_examples_list example_sdl3_sdlrenderer3)
  endif()
endif()

if(DearImGui_Backend_Win32)
  add_library(imgui_impl_win32)
  target_sources(imgui_impl_win32
    PRIVATE ${_imgui_dir}/backends/imgui_impl_win32.cpp
    PUBLIC FILE_SET imgui_impl_win32_set
    BASE_DIRS ${_imgui_dir}/backends
    TYPE HEADERS
    FILES ${_imgui_dir}/backends/imgui_impl_win32.h)
  target_link_libraries(imgui_impl_win32
    PUBLIC imgui_core
    PUBLIC $<$<AND:$<PLATFORM_ID:Windows>,$<CXX_COMPILER_ID:GNU,Clang>>:dwmapi>)

  list(APPEND imgui_backends_list imgui_impl_win32)

  if(Example_Win32_OpenGL3)
    add_executable(example_win32_opengl3)
    target_sources(example_win32_opengl3 PRIVATE ${_imgui_dir}/examples/example_win32_opengl3/main.cpp)
    target_link_libraries(example_win32_opengl3 PRIVATE imgui_impl_win32 imgui_impl_opengl3)

    list(APPEND imgui_examples_list example_win32_opengl3)
  endif()

  if(Example_Win32_Vulkan)
    add_executable(example_win32_vulkan)
    target_sources(example_win32_vulkan PRIVATE ${_imgui_dir}/examples/example_win32_vulkan/main.cpp)
    target_link_libraries(example_win32_vulkan PRIVATE imgui_impl_win32 imgui_impl_vulkan)

    list(APPEND imgui_examples_list example_win32_vulkan)
  endif()

  if(Example_Win32_DirectX9)
    add_executable(example_win32_directx9)
    target_sources(example_win32_directx9 PRIVATE ${_imgui_dir}/examples/example_win32_directx9/main.cpp)
    target_link_libraries(example_win32_directx9 PRIVATE imgui_impl_win32 imgui_impl_dx9)

    list(APPEND imgui_examples_list example_win32_directx9)
  endif()

  if(Example_Win32_DirectX10)
    add_executable(example_win32_directx10)
    target_sources(example_win32_directx10 PRIVATE ${_imgui_dir}/examples/example_win32_directx10/main.cpp)
    target_link_libraries(example_win32_directx10 PRIVATE imgui_impl_win32 imgui_impl_dx10)

    list(APPEND imgui_examples_list example_win32_directx10)
  endif()

  if(Example_Win32_DirectX11)
    add_executable(example_win32_directx11)
    target_sources(example_win32_directx11 PRIVATE ${_imgui_dir}/examples/example_win32_directx11/main.cpp)
    target_link_libraries(example_win32_directx11 PRIVATE imgui_impl_win32 imgui_impl_dx11)

    list(APPEND imgui_examples_list example_win32_directx11)
  endif()

  if(Example_Win32_DirectX12)
    add_executable(example_win32_directx12)
    target_sources(example_win32_directx12 PRIVATE ${_imgui_dir}/examples/example_win32_directx12/main.cpp)
    target_link_libraries(example_win32_directx12 PRIVATE imgui_impl_win32 imgui_impl_dx12)

    list(APPEND imgui_examples_list example_win32_directx12)
  endif()
endif()

if(Example_NULL)
  add_executable(example_null)
  target_sources(example_null PRIVATE ${_imgui_dir}/examples/example_null/main.cpp)
  target_link_libraries(example_null PRIVATE imgui_core)

  list(APPEND imgui_examples_list example_null)
endif()

if((DearImGui_Backend_DirectX9 OR DearImGui_Backend_DirectX10 OR DearImGui_Backend_DirectX11 OR DearImGui_Backend_DirectX12) AND (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC"))
  set(sdk_version ${CMAKE_VS_WINDOWS_TARGET_PLATFORM_VERSION})

  find_path(Windows_SDK_INCLUDE_DIR
    NAMES d3d9.h
    PATHS "[HKLM/Software/Microsoft/Windows Kits/Installed Roots;KitsRoot10]" "C:/Program Files (x86)/Windows Kits/10"
    PATH_SUFFIXES "Include/${sdk_version}/shared" NO_CACHE REQUIRED)

  find_path(Windows_SDK_INCLUDE_DIR_1
    NAMES d3d10.h d3d10_1.h d3d11.h d3d12.h d3dcompiler.h
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

  find_path(Windows_SDK_LIB_DIR
    NAMES d3d9.lib d3d10.lib d3d10_1.lib d3d11.lib d3d12.lib d3dcompiler.lib
    PATHS "[HKLM/SOFTWARE/Microsoft/Windows Kits/Installed Roots;KitsRoot10]"
    PATH_SUFFIXES "Lib/${sdk_version}/um/${arch}" "C:/Program Files (x86)/Windows Kits/10/Lib/${sdk_version}/um/${arch}" REGISTRY_VIEW HOST NO_CACHE REQUIRED)
endif()

if(DearImGui_Backend_DirectX9)
  add_library(imgui_impl_dx9)
  target_sources(imgui_impl_dx9
    PRIVATE ${_imgui_dir}/backends/imgui_impl_dx9.cpp
    PUBLIC FILE_SET imgui_impl_dx9_set
    BASE_DIRS ${_imgui_dir}/backends
    TYPE HEADERS
    FILES ${_imgui_dir}/backends/imgui_impl_dx9.h)
  target_link_libraries(imgui_impl_dx9 PUBLIC imgui_core)

  if(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    target_link_directories(imgui_impl_dx9 PRIVATE ${Windows_SDK_LIB_DIR})
    target_link_libraries(imgui_impl_dx9 PUBLIC d3d9.lib)
    target_sources(imgui_impl_dx9 PRIVATE FILE_SET HEADERS BASE_DIRS ${Windows_SDK_INCLUDE_DIR} FILES ${Windows_SDK_INCLUDE_DIR}/d3d9.h)
  elseif(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang") # cross compiling with mingw or msys2
    target_link_libraries(imgui_impl_dx9 PUBLIC d3d9.lib)
  endif()

  list(APPEND imgui_backends_list imgui_impl_dx9)
endif()

if(DearImGui_Backend_DirectX10)
  add_library(imgui_impl_dx10)
  target_sources(imgui_impl_dx10
    PRIVATE ${_imgui_dir}/backends/imgui_impl_dx10.cpp
    PUBLIC FILE_SET imgui_impl_dx10_set
    BASE_DIRS ${_imgui_dir}/backends
    TYPE HEADERS
    FILES ${_imgui_dir}/backends/imgui_impl_dx10.h)
  target_link_libraries(imgui_impl_dx10 PUBLIC imgui_core)

  if(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    target_link_directories(imgui_impl_dx10 PRIVATE ${Windows_SDK_LIB_DIR})
    target_link_libraries(imgui_impl_dx10 PUBLIC d3d10_1.lib d3d10.lib d3dcompiler.lib)
    target_sources(imgui_impl_dx10
      PRIVATE FILE_SET HEADERS
      BASE_DIRS ${Windows_SDK_INCLUDE_DIR_1}
      FILES ${Windows_SDK_INCLUDE_DIR_1}/d3d10_1.h ${Windows_SDK_INCLUDE_DIR_1}/d3d10.h ${Windows_SDK_INCLUDE_DIR_1}/d3dcompiler.h)
  elseif(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_link_libraries(imgui_impl_dx10 PUBLIC d3d10.lib d3dcompiler.lib)
  endif()

  list(APPEND imgui_backends_list imgui_impl_dx10)
endif()

if(DearImGui_Backend_DirectX11)
  add_library(imgui_impl_dx11)
  target_sources(imgui_impl_dx11
    PRIVATE ${_imgui_dir}/backends/imgui_impl_dx11.cpp
    PUBLIC FILE_SET imgui_impl_dx11_set
    BASE_DIRS ${_imgui_dir}/backends
    TYPE HEADERS
    FILES ${_imgui_dir}/backends/imgui_impl_dx11.h)
  target_link_libraries(imgui_impl_dx11 PUBLIC imgui_core)

  if(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    target_link_directories(imgui_impl_dx11 PRIVATE ${Windows_SDK_LIB_DIR})
    target_link_libraries(imgui_impl_dx11 PUBLIC d3d11.lib d3dcompiler.lib)
    target_sources(imgui_impl_dx11
      PRIVATE FILE_SET HEADERS
      BASE_DIRS ${Windows_SDK_INCLUDE_DIR_1}
      FILES ${Windows_SDK_INCLUDE_DIR_1}/d3d11.h ${Windows_SDK_INCLUDE_DIR_1}/d3dcompiler.h)
  elseif(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_link_libraries(imgui_impl_dx11 PUBLIC d3d11.lib d3dcompiler.lib)
  endif()

  list(APPEND imgui_backends_list imgui_impl_dx11)
endif()

if(DearImGui_Backend_DirectX12)
  add_library(imgui_impl_dx12)
  target_sources(imgui_impl_dx12
    PRIVATE ${_imgui_dir}/backends/imgui_impl_dx12.cpp
    PUBLIC FILE_SET imgui_impl_dx12_set
    BASE_DIRS ${_imgui_dir}/backends
    TYPE HEADERS
    FILES ${_imgui_dir}/backends/imgui_impl_dx12.h)
  target_link_libraries(imgui_impl_dx12 PUBLIC imgui_core)

  if(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    target_link_directories(imgui_impl_dx12 PRIVATE ${Windows_SDK_LIB_DIR})
    target_link_libraries(imgui_impl_dx12 PUBLIC d3d12.lib dxgi.lib d3dcompiler.lib)
    target_sources(imgui_impl_dx12
      PRIVATE FILE_SET HEADERS
      BASE_DIRS ${Windows_SDK_INCLUDE_DIR_1}
      FILES ${Windows_SDK_INCLUDE_DIR_1}/d3d12.h ${Windows_SDK_INCLUDE_DIR_1}/d3dcompiler.h
      PRIVATE FILE_SET HEADERS
      BASE_DIRS ${Windows_SDK_INCLUDE_DIR}
      FILES ${Windows_SDK_INCLUDE_DIR}/dxgi1_4.h)
  elseif(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_link_libraries(imgui_impl_dx12 PUBLIC d3d12.lib dxgi.lib d3dcompiler.lib)
  endif()

  list(APPEND imgui_backends_list imgui_impl_dx12)
endif()

if(DearImGui_Backend_Metal)
  add_library(imgui_impl_metal)
  target_sources(imgui_impl_metal
    PRIVATE ${_imgui_dir}/backends/imgui_impl_metal.mm
    PUBLIC FILE_SET imgui_impl_metal_set
    BASE_DIRS ${_imgui_dir}/backends
    TYPE HEADERS
    FILES ${_imgui_dir}/backends/imgui_impl_metal.h)
  target_link_libraries(imgui_impl_metal
    PUBLIC imgui_core
    PUBLIC "-framework Metal" INTERFACE "-framework MetalKit")
  target_compile_options(imgui_impl_metal PUBLIC $<$<VERSION_LESS:${IMGUI_VERSION},18940>:-fobjc-arc>)

  list(APPEND imgui_backends_list imgui_impl_metal)
endif()

if(DearImGui_Backend_OSX)
  add_library(imgui_impl_osx)
  target_sources(imgui_impl_osx
    PRIVATE ${_imgui_dir}/backends/imgui_impl_osx.mm
    PUBLIC FILE_SET imgui_impl_osx_set
    BASE_DIRS ${_imgui_dir}/backends
    TYPE HEADERS
    FILES ${_imgui_dir}/backends/imgui_impl_osx.h)
  target_link_libraries(imgui_impl_osx
    PUBLIC imgui_core
    PUBLIC "-framework AppKit" "-framework Carbon" "-framework GameController")

  list(APPEND imgui_backends_list imgui_impl_osx)

  if(Example_Apple_OpenGL2)
    add_executable(example_apple_opengl2 MACOSX_BUNDLE)
    target_sources(example_apple_opengl2 PRIVATE ${_imgui_dir}/examples/example_apple_opengl2/main.mm)
    target_link_libraries(example_apple_opengl2 PRIVATE imgui_impl_osx imgui_impl_opengl2)

    list(APPEND imgui_examples_list example_apple_opengl2)
  endif()

  if(Example_Apple_Metal)
    add_executable(example_apple_metal MACOSX_BUNDLE)
    target_sources(example_apple_metal PRIVATE ${_imgui_dir}/examples/example_apple_metal/main.mm)
    target_link_libraries(example_apple_metal PRIVATE imgui_impl_osx imgui_impl_metal)

    list(APPEND imgui_examples_list example_apple_metal)
  endif()
endif()

if(Pkg-config)
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

  file(CONFIGURE OUTPUT ${PROJECT_BINARY_DIR}/dearimgui.pc CONTENT
[=[
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
Libs: -L${libdir} -limgui_core @pkg_config_imgui_backends_string@
Libs.private:
Cflags: -I${includedir}
]=] @ONLY)

endif()

if(Install)
  set_property(GLOBAL PROPERTY INSTALL_PARALLEL ON)

  install(DIRECTORY ${_imgui_dir}/docs DESTINATION ${CMAKE_INSTALL_DATADIR}/Dear\ ImGui)
  install(FILES ${_imgui_dir}/LICENSE.txt DESTINATION ${CMAKE_INSTALL_DATADIR}/Dear\ ImGui)
  install(EXPORT imgui_export DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/DearImGui NAMESPACE ImGui:: FILE DearImGuiTargets.cmake)
  install(TARGETS imgui_core EXPORT imgui_export FILE_SET imgui_core_set FILE_SET HEADERS FILE_SET imgui_misc_cpp_set FILE_SET imgui_misc_freetype_set)

  install(DIRECTORY $<$<VERSION_GREATER:${IMGUI_VERSION},18100>:${_imgui_dir}/misc/debuggers> DESTINATION ${CMAKE_INSTALL_DATADIR}/Dear\ ImGui)
  install(TARGETS binary_to_compressed_c DESTINATION ${CMAKE_INSTALL_DATADIR}/Dear\ ImGui/fonts)
  install(DIRECTORY ${_imgui_dir}/misc/fonts DESTINATION ${CMAKE_INSTALL_DATADIR}/Dear\ ImGui FILES_MATCHING PATTERN "*.ttf")

  foreach(backend IN LISTS imgui_backends_list)
    get_property(name TARGET ${backend} PROPERTY NAME)
    get_property(file_set TARGET ${backend} PROPERTY HEADER_SETS)
    list(REMOVE_ITEM file_set HEADERS)

    install(TARGETS ${name} EXPORT imgui_export FILE_SET ${file_set})
  endforeach()

  if(Install_examples)
    if(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
      set_target_properties(${imgui_examples_list} PROPERTIES MACOSX_BUNDLE TRUE)
      install(TARGETS ${imgui_examples_list} BUNDLE DESTINATION Applications)
    else()
      install(TARGETS ${imgui_examples_list})
    endif()
  endif()

  if(Pkg-config)
    install(FILES ${PROJECT_BINARY_DIR}/dearimgui.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
  endif()

  configure_package_config_file(DearImGuiConfig.cmake.in ${PROJECT_BINARY_DIR}/DearImGuiConfig.cmake INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/DearImGui)
  write_basic_package_version_file(${PROJECT_BINARY_DIR}/DearImGuiConfigVersion.cmake VERSION ${IMGUI_VERSION} COMPATIBILITY ExactVersion)

  install(FILES ${PROJECT_BINARY_DIR}/DearImGuiConfig.cmake ${PROJECT_BINARY_DIR}/DearImGuiConfigVersion.cmake DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/DearImGui)
endif()

if(Uninstall)
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

  add_custom_target(uninstall
    COMMAND ${CMAKE_COMMAND} -P ${PROJECT_BINARY_DIR}/cmake_uninstall.cmake
    WORKING_DIRECTORY ${PROJECT_BINARY_DIR})
endif()

if(visualize_targets)
  if(NOT EXISTS ${PROJECT_BINARY_DIR}/CMakeGraphVizOptions.cmake)
    file(WRITE ${PROJECT_BINARY_DIR}/CMakeGraphVizOptions.cmake
[=[
set(GRAPHVIZ_GENERATE_PER_TARGET FALSE)
set(GRAPHVIZ_GENERATE_DEPENDERS FALSE)
]=])
  endif()

  find_program(DOT_COMMAND dot REQUIRED)

  add_custom_target(graph
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
Example_NULL
Example_Allegro5 Example_Android_OpenGL3
Example_GLFW_OpenGL2 Example_GLFW_OpenGL3 Example_GLFW_Vulkan Example_GLFW_Metal Example_GLFW_WebGPU
Example_FreeGLUT_OpenGL2
Example_SDL2_DirectX11 Example_SDL2_OpenGL2 Example_SDL2_OpenGL3 Example_SDL2_Vulkan Example_SDL2_Metal Example_SDL2_SDLRenderer2
Example_SDL3_OpenGL3 Example_SDL3_SDLGPU3 Example_SDL3_SDLRenderer3 Example_SDL3_Vulkan Example_SDL3_Metal
Example_Win32_DirectX9 Example_Win32_DirectX10 Example_Win32_DirectX11 Example_Win32_DirectX12 Example_Win32_OpenGL3 Example_Win32_Vulkan
Example_Apple_Metal Example_Apple_OpenGL2
BUILD_SHARED_LIBS Pkg-config Install Install_examples Uninstall visualize_targets)
list(SORT options CASE INSENSITIVE)

message(STATUS "Configuration summary:\n")
foreach(o IN LISTS options)
  message(" * ${o}: ${${o}}")
endforeach()
