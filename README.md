# CMake support for ImGui

[CMake](https://cmake.org) build script support for [Dear ImGui](https://github.com/ocornut/imgui). This builds and installs the library and backends, also builds example programs.

## Building

```cmake
cmake -DIMGUI_SOURCE_DIR=path.to.imgui -S . -B build
```

Setting `IMGUI_SOURCE_DIR` variable on configuration step is mandatory. An example usage might look like this:

```cmake
cmake -G"Ninja Multi-Config" -DIMGUI_SOURCE_DIR=imgui-1.91.9b -S . -B build
cmake -Dsdl2=ON -Dopengl3=ON -S . -B build
cmake --build build --config Release
cmake --install build --config Release
```

Following backend options are available:

| Build option   | Generated target                                      |
|----------------|-------------------------------------------------------|
| (default)      | Unofficial::DearImGui::imgui_core                     |
| opengl2        | Unofficial::DearImGui::imgui_backend_opengl2          |
| opengl3        | Unofficial::DearImGui::imgui_backend_opengl3          |
| vulkan         | Unofficial::DearImGui::imgui_backend_vulkan           |
| allegro5       | Unofficial::DearImGui::imgui_backend_allegro5         |
| glfw           | Unofficial::DearImGui::imgui_backend_glfw             |
| glut           | Unofficial::DearImGui::imgui_backend_glut             |
| sdl2           | Unofficial::DearImGui::imgui_backend_sdl2             |
| sdlrenderer2   | Unofficial::DearImGui::imgui_backend_sdlrenderer2     |
| sdl3           | Unofficial::DearImGui::imgui_backend_sdl3             |
| sdlgpu3        | Unofficial::DearImGui::imgui_backend_sdlgpu3          |
| sdlrenderer3   | Unofficial::DearImGui::imgui_backend_sdlrenderer3     |
| win32          | Unofficial::DearImGui::imgui_backend_win32            |
| dx9            | Unofficial::DearImGui::imgui_backend_dx9              |
| dx10           | Unofficial::DearImGui::imgui_backend_dx10             |
| dx11           | Unofficial::DearImGui::imgui_backend_dx11             |
| dx12           | Unofficial::DearImGui::imgui_backend_dx12             |

All backend options are `OFF` by default.

Example programs set as dependent options, like:
```cmake
cmake_dependent_option(example_sdl2_opengl3 "" OFF "examples AND sdl2 AND opengl3" OFF)
```
The `example_sdl2_opengl3` option will be available only when the `examples` option and the corresponding backends (`sdl2` and `opengl3`) are `ON`. This is true for all the other examples. No install rules are written for example programs.

## Usage

All the include paths are kept as is. [This repo](https://github.com/adembudak/CMakeForImGui.test) demonstrates how to build programs as a client of the library:


```cmake
# project setup
# ...

find_package(CMakeForImGui CONFIG REQUIRED)
# ...
target_link_libraries(tgt PUBLIC Unofficial::DearImGui::sdl2 Unofficial::DearImGui::opengl3)
```

What is missing:
- Android backend
- WGPU backend
- Apple Metal backend
- Apple OpenGL backend
- Apple OS X backend
    and their corresponding example programs

See help-wanted tags in issues.

