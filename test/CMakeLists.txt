cmake_minimum_required(VERSION 4.0.0)

project(
  CMakeForImGui.test
  DESCRIPTION "Examples folder of the Dear ImGui built by CMakeForImGui"
  LANGUAGES CXX)

find_package(DearImGui CONFIG REQUIRED)
message("Dear ImGui version: ${DearImGui_VERSION}")

set(IMGUI_SOURCE_DIR "" CACHE PATH "Path to Dear ImGui source tree")
if(NOT IMGUI_SOURCE_DIR)
  message(FATAL_ERROR "set IMGUI_SOURCE_DIR to proceed:\ne.g. cmake -DIMGUI_SOURCE_DIR=path.to.imgui ...")
endif()
cmake_path(SET _imgui_dir ${IMGUI_SOURCE_DIR})

add_executable(example_null)
target_sources(example_null PRIVATE ${_imgui_dir}/examples/example_null/main.cpp)
target_link_libraries(example_null PRIVATE ImGui::Core)

add_executable(example_allegro5)
target_sources(example_allegro5 PRIVATE ${_imgui_dir}/examples/example_allegro5/main.cpp)
target_link_libraries(example_allegro5 PRIVATE ImGui::Backend_Allegro5)
target_link_libraries(example_allegro5 PRIVATE "$<$<PLATFORM_ID:Darwin>:-framework OpenGL -framework Carbon -framework ApplicationServices>")

add_executable(example_glfw_opengl3)
target_sources(example_glfw_opengl3 PRIVATE ${_imgui_dir}/examples/example_glfw_opengl3/main.cpp)
target_link_libraries(example_glfw_opengl3 PRIVATE ImGui::Backend_GLFW ImGui::Backend_OpenGL3)

add_executable(example_glfw_opengl2)
target_sources(example_glfw_opengl2 PRIVATE ${_imgui_dir}/examples/example_glfw_opengl2/main.cpp)
target_link_libraries(example_glfw_opengl2 PRIVATE ImGui::Backend_GLFW ImGui::Backend_OpenGL2)

add_executable(example_glfw_vulkan)
target_sources(example_glfw_vulkan PRIVATE ${_imgui_dir}/examples/example_glfw_vulkan/main.cpp)
target_link_libraries(example_glfw_vulkan PRIVATE ImGui::Backend_GLFW ImGui::Backend_Vulkan)

add_executable(example_glut_opengl2)
target_sources(example_glut_opengl2 PRIVATE ${_imgui_dir}/examples/example_glut_opengl2/main.cpp)
target_link_libraries(example_glut_opengl2 PRIVATE ImGui::Backend_FreeGLUT ImGui::Backend_OpenGL2)

add_executable(example_sdl2_opengl2)
target_sources(example_sdl2_opengl2 PRIVATE ${_imgui_dir}/examples/example_sdl2_opengl2/main.cpp)
target_link_libraries(example_sdl2_opengl2 PRIVATE ImGui::Backend_SDL2 ImGui::Backend_OpenGL2)
target_compile_definitions(example_sdl2_opengl2 PRIVATE $<$<PLATFORM_ID:Windows>:SDL_MAIN_HANDLED>)

add_executable(example_sdl2_opengl3)
target_sources(example_sdl2_opengl3 PRIVATE ${_imgui_dir}/examples/example_sdl2_opengl3/main.cpp)
target_link_libraries(example_sdl2_opengl3 PRIVATE ImGui::Backend_SDL2 ImGui::Backend_OpenGL3)
target_compile_definitions(example_sdl2_opengl3 PRIVATE $<$<PLATFORM_ID:Windows>:SDL_MAIN_HANDLED>)

add_executable(example_sdl2_vulkan)
target_sources(example_sdl2_vulkan PRIVATE ${_imgui_dir}/examples/example_sdl2_vulkan/main.cpp)
target_link_libraries(example_sdl2_vulkan PRIVATE ImGui::Backend_SDL2 ImGui::Backend_Vulkan)
target_compile_definitions(example_sdl2_vulkan PRIVATE $<$<PLATFORM_ID:Windows>:SDL_MAIN_HANDLED>)

add_executable(example_sdl2_sdlrenderer2)
target_sources(example_sdl2_sdlrenderer2 PRIVATE ${_imgui_dir}/examples/example_sdl2_sdlrenderer2/main.cpp)
target_link_libraries(example_sdl2_sdlrenderer2 PRIVATE ImGui::Backend_SDLRenderer2 ImGui::Backend_SDL2)
target_compile_definitions(example_sdl2_sdlrenderer2 PRIVATE $<$<PLATFORM_ID:Windows>:SDL_MAIN_HANDLED>)

if(${DearImGui_VERSION} VERSION_GREATER 19100)
  add_executable(example_sdl3_opengl3)
  target_sources(example_sdl3_opengl3 PRIVATE ${_imgui_dir}/examples/example_sdl3_opengl3/main.cpp)
  target_link_libraries(example_sdl3_opengl3 PRIVATE ImGui::Backend_SDL3 ImGui::Backend_OpenGL3)
  target_compile_definitions(example_sdl3_opengl3 PRIVATE $<$<VERSION_LESS:${DearImGui_VERSION},19130>:SDL_INIT_TIMER=0x00000001u>)

  if(${DearImGui_VERSION} VERSION_GREATER 19140)
    add_executable(example_sdl3_vulkan)
    target_sources(example_sdl3_vulkan PRIVATE ${_imgui_dir}/examples/example_sdl3_vulkan/main.cpp)
    target_link_libraries(example_sdl3_vulkan PRIVATE ImGui::Backend_SDL3 ImGui::Backend_Vulkan)
  endif()
endif()

if(${DearImGui_VERSION} VERSION_GREATER 19100)
  add_executable(example_sdl3_sdlrenderer3)
  target_sources(example_sdl3_sdlrenderer3 PRIVATE ${_imgui_dir}/examples/example_sdl3_sdlrenderer3/main.cpp)
  target_link_libraries(example_sdl3_sdlrenderer3 PRIVATE ImGui::Backend_SDLRenderer3 ImGui::Backend_SDL3)
  target_compile_definitions(example_sdl3_sdlrenderer3 PRIVATE $<$<VERSION_LESS:${DearImGui_VERSION},19130>:SDL_INIT_TIMER=0x00000001u>)
endif()

if(${DearImGui_VERSION} VERSION_GREATER 19160)
  add_executable(example_sdl3_sdlgpu3)
  target_sources(example_sdl3_sdlgpu3 PRIVATE ${_imgui_dir}/examples/example_sdl3_sdlgpu3/main.cpp)
  target_link_libraries(example_sdl3_sdlgpu3 PRIVATE ImGui::Backend_SDLGPU3 ImGui::Backend_SDL3)
endif()

if(WIN32)
  if(${DearImGui_VERSION} VERSION_GREATER 18950)
    add_executable(example_win32_opengl3)
    target_sources(example_win32_opengl3 PRIVATE ${_imgui_dir}/examples/example_win32_opengl3/main.cpp)
    target_link_libraries(example_win32_opengl3 PRIVATE ImGui::Backend_Win32 ImGui::Backend_OpenGL3)
  endif()

  add_executable(example_win32_directx9)
  target_sources(example_win32_directx9 PRIVATE ${_imgui_dir}/examples/example_win32_directx9/main.cpp)
  target_link_libraries(example_win32_directx9 PRIVATE ImGui::Backend_Win32 ImGui::Backend_DirectX9)

  add_executable(example_win32_directx10)
  target_sources(example_win32_directx10 PRIVATE ${_imgui_dir}/examples/example_win32_directx10/main.cpp)
  target_link_libraries(example_win32_directx10 PRIVATE ImGui::Backend_Win32 ImGui::Backend_DirectX10)

  add_executable(example_win32_directx11)
  target_sources(example_win32_directx11 PRIVATE ${_imgui_dir}/examples/example_win32_directx11/main.cpp)
  target_link_libraries(example_win32_directx11 PRIVATE ImGui::Backend_Win32 ImGui::Backend_DirectX11)

  add_executable(example_win32_directx12)
  target_sources(example_win32_directx12 PRIVATE ${_imgui_dir}/examples/example_win32_directx12/main.cpp)
  target_link_libraries(example_win32_directx12 PRIVATE ImGui::Backend_Win32 ImGui::Backend_DirectX12)

  if(${DearImGui_VERSION} VERSION_GREATER 19160)
    add_executable(example_win32_vulkan)
    target_sources(example_win32_vulkan PRIVATE ${_imgui_dir}/examples/example_win32_vulkan/main.cpp)
    target_link_libraries(example_win32_vulkan PRIVATE ImGui::Backend_Win32 ImGui::Backend_Vulkan)
  endif()

  add_executable(example_sdl2_directx11)
  target_sources(example_sdl2_directx11 PRIVATE ${_imgui_dir}/examples/example_sdl2_directx11/main.cpp)
  target_link_libraries(example_sdl2_directx11 PRIVATE ImGui::Backend_SDL2 ImGui::Backend_DirectX11)
  target_compile_definitions(example_sdl2_directx11 PRIVATE $<$<PLATFORM_ID:Windows>:SDL_MAIN_HANDLED>)
endif()

if(APPLE)
  add_executable(example_apple_opengl2)
  target_sources(example_apple_opengl2 PRIVATE ${_imgui_dir}/examples/example_apple_opengl2/main.mm)
  target_link_libraries(example_apple_opengl2 PRIVATE ImGui::Backend_OSX ImGui::Backend_OpenGL2)

  add_executable(example_apple_metal)
  target_sources(example_apple_metal PRIVATE ${_imgui_dir}/examples/example_apple_metal/main.mm)
  target_link_libraries(example_apple_metal PRIVATE ImGui::Backend_OSX ImGui::Backend_Metal)

  add_executable(example_sdl2_metal)
  target_sources(example_sdl2_metal PRIVATE ${_imgui_dir}/examples/example_sdl2_metal/main.mm)
  target_link_libraries(example_sdl2_metal PRIVATE ImGui::Backend_SDL2 ImGui::Backend_Metal)

  if(${DearImGui_VERSION} VERSION_GREATER 19210)
    add_executable(example_sdl3_metal)
    target_sources(example_sdl3_metal PRIVATE ${_imgui_dir}/examples/example_sdl3_metal/main.mm)
    target_link_libraries(example_sdl3_metal PRIVATE ImGui::Backend_SDL3 ImGui::Backend_Metal)
  endif()

  add_executable(example_glfw_metal)
  target_sources(example_glfw_metal PRIVATE ${_imgui_dir}/examples/example_glfw_metal/main.mm)
  target_link_libraries(example_glfw_metal PRIVATE ImGui::Backend_GLFW ImGui::Backend_Metal)
  target_link_libraries(example_glfw_metal PRIVATE "-framework QuartzCore")
endif()
