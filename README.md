[CMake](https://cmake.org) build script support for [Dear ImGui](https://github.com/ocornut/imgui). This builds and installs the library and backends, also builds example programs.

This work proposed for the Dear ImGui upstream: https://github.com/ocornut/imgui/issues/8896

## Building

```cmake
cmake -DIMGUI_SOURCE_DIR=path.to.imgui -S . -B build
```

Setting `IMGUI_SOURCE_DIR` variable on configuration step is mandatory. An example usage might look like this:

```cmake
cmake -G"Ninja Multi-Config" -DIMGUI_SOURCE_DIR=imgui-1.91.9b -S . -B build
cmake -DDearImGui_Backend_SDL2=ON -DDearImGui_Backend_OpenGL3=ON -S . -B build
cmake --build build --config Release
cmake --install build --config Release
```
The commands above assume the dependencies of the backend are installed on host system. The user can refer to their websites or can use a package manager.

Following backend options are available:

| Build options                    | Targets                       |
|----------------------------------|-------------------------------|
| (default)                        | `ImGui::Core`                 |
| DearImGui_Backend_Android        | `ImGui::Backend_Android`      |
| DearImGui_Backend_OpenGL2        | `ImGui::Backend_OpenGL2`      |
| DearImGui_Backend_OpenGL3        | `ImGui::Backend_OpenGL3`      |
| DearImGui_Backend_Vulkan         | `ImGui::Backend_Vulkan`       |
| DearImGui_Backend_Allegro5       | `ImGui::Backend_Allegro5`     |
| DearImGui_Backend_GLFW           | `ImGui::Backend_GLFW`         |
| DearImGui_Backend_FreeGLUT       | `ImGui::Backend_FreeGLUT`     |
| DearImGui_Backend_SDL2           | `ImGui::Backend_SDL2`         |
| DearImGui_Backend_SDLRenderer2   | `ImGui::Backend_SDLRenderer2` |
| DearImGui_Backend_SDL3           | `ImGui::Backend_SDL3`         |
| DearImGui_Backend_SDLGPU3        | `ImGui::Backend_SDLGPU3`      |
| DearImGui_Backend_SDLRenderer3   | `ImGui::Backend_SDLRenderer3` |
| DearImGui_Backend_WebGPU         | `ImGui::Backend_WebGPU`       |
| DearImGui_Backend_Win32          | `ImGui::Backend_Win32`        |
| DearImGui_Backend_DirectX9       | `ImGui::Backend_DirectX9`     |
| DearImGui_Backend_DirectX10      | `ImGui::Backend_DirectX10`    |
| DearImGui_Backend_DirectX11      | `ImGui::Backend_DirectX11`    |
| DearImGui_Backend_DirectX12      | `ImGui::Backend_DirectX12`    |
| DearImGui_Backend_OSX            | `ImGui::Backend_OSX`          |
| DearImGui_Backend_Metal          | `ImGui::Backend_Metal`        |

All backend options are `OFF` by default.

Example programs set as dependent options, like:
```cmake
cmake_dependent_option(Example_SDL2_OpenGL3 "" OFF "DearImGui_Examples AND DearImGui_Backend_SDL2 AND DearImGui_Backend_OpenGL3" OFF)
```
The `Example_SDL2_OpenGL3` option will be available only when the `DearImGui_Examples` option and the corresponding backends are `ON`. The same is true for all the other examples and they're `OFF` by default.

Libraries and examples can be installed by setting `install` and `install_examples` options `ON`, respectively. An `uninstall` custom target is also provided to remove the artifacts where are they installed.

Following projects are also supported:

| Project | Variable must be set | Build option(s) | Generated target(s) |
|--------|---------------------|-----------------|---------------------|
| [imgui_test_engine](https://github.com/ocornut/imgui_test_engine) | IMGUI_TEST_ENGINE_SOURCE_DIR | imgui_test_engine | <br>`Unofficial::imgui_test_engine::imgui_test_engine`<br>`Unofficial::imgui_test_engine::imgui_app` |
| [imgui_club](https://github.com/ocornut/imgui_club) | IMGUI_CLUB_SOURCE_DIR | imgui_club<br>imgui_memory_editor<br>imgui_multicontext_compositor<br>imgui_threaded_rendering | <br>`Unofficial::imgui_club::imgui_memory_editor`<br>`Unofficial::imgui_club::imgui_multicontext_compositor`<br>`Unofficial::imgui_club::imgui_threaded_rendering` |
| [imgui_markdown](https://github.com/enkisoftware/imgui_markdown) | IMGUI_MARKDOWN_SOURCE_DIR | imgui_markdown | `Unofficial::imgui_markdown::imgui_markdown` |
| [ImPlot](https://github.com/epezent/implot) | IMPLOT_SOURCE_DIR | implot | `Unofficial::ImPlot::implot` |
| [ImPlot3D](https://github.com/brenocq/implot3d) | IMPLOT3D_SOURCE_DIR | implot3d | `Unofficial::ImPlot3D::implot3d` |
| [ImGuiFileDialog](https://github.com/aiekick/ImGuiFileDialog) | IMGUIFILEDIALOG_SOURCE_DIR | imguifiledialog | `Unofficial::ImGuiFileDialog::imguifiledialog` |

## Usage

All the include paths are kept as is. [This repo](https://github.com/adembudak/CMakeForImGui.test) demonstrates how to build programs as a client of the library:

```cmake
# project setup
# ...

find_package(CMakeForImGui CONFIG REQUIRED)
# ...
target_link_libraries(tgt PUBLIC ImGui::Backend_SDL2 ImGui::Backend_OpenGL3)
```
There is also [this branch](https://github.com/adembudak/CMakeForImGui/tree/single-target) available, where a single target approach is investigated. The same options are available but all the backends and ImGui itself is linked with a `Unofficial::DearImGui::imgui` target.

### With `pkg-config` command

The build script can be used to generate **pkg-config** file:
```cmake
cmake -Dpkg-config=ON -S . -B build
```

This generates a `dearimgui.pc` file in build directory which can be installed and used, like:
```bash
c++ -o out main.cpp $(pkg-config --cflags --libs dearimgui)
```

## Using with older versions of the Dear ImGui

Limited amount of checking has done for moved, removed or renamed files of the previous versions of the Dear ImGui. The main branch will work best with recent the versions of the project, i.e. v1.90+. Some work has done for the versions older than 1.80 on [this branch](https://github.com/adembudak/CMakeForImGui/tree/pre.v1.80).

## Help needed

Any kind of feedback, review or input is appreciated and welcome...

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Status](https://github.com/adembudak/CMakeForImGui/actions/workflows/main.yml/badge.svg)](https://github.com/adembudak/CMakeForImGui/actions/workflows/main.yml)
