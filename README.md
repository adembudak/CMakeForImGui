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
The commands above assume the dependencies of the backend are installed on host system. The user can refer to their websites or can use a package manager.

Following backend options are available:

| Build option   | Generated target                                     |
|----------------|------------------------------------------------------|
| (default)      | `Unofficial::DearImGui::imgui_core`                  |
| android        | `Unofficial::DearImGui::imgui_backend_android`       |
| opengl2        | `Unofficial::DearImGui::imgui_backend_opengl2`       |
| opengl3        | `Unofficial::DearImGui::imgui_backend_opengl3`       |
| vulkan         | `Unofficial::DearImGui::imgui_backend_vulkan`        |
| allegro5       | `Unofficial::DearImGui::imgui_backend_allegro5`      |
| glfw           | `Unofficial::DearImGui::imgui_backend_glfw`          |
| glut           | `Unofficial::DearImGui::imgui_backend_glut`          |
| sdl2           | `Unofficial::DearImGui::imgui_backend_sdl2`          |
| sdlrenderer2   | `Unofficial::DearImGui::imgui_backend_sdlrenderer2`  |
| sdl3           | `Unofficial::DearImGui::imgui_backend_sdl3`          |
| sdlgpu3        | `Unofficial::DearImGui::imgui_backend_sdlgpu3`       |
| sdlrenderer3   | `Unofficial::DearImGui::imgui_backend_sdlrenderer3`  |
| win32          | `Unofficial::DearImGui::imgui_backend_win32`         |
| dx9            | `Unofficial::DearImGui::imgui_backend_dx9`           |
| dx10           | `Unofficial::DearImGui::imgui_backend_dx10`          |
| dx11           | `Unofficial::DearImGui::imgui_backend_dx11`          |
| dx12           | `Unofficial::DearImGui::imgui_backend_dx12`          |
| osx            | `Unofficial::DearImGui::imgui_backend_osx`           |
| metal          | `Unofficial::DearImGui::imgui_backend_metal`         |

All backend options are `OFF` by default.

Example programs set as dependent options, like:
```cmake
cmake_dependent_option(example_sdl2_opengl3 "" OFF "examples AND sdl2 AND opengl3" OFF)
```
The `example_sdl2_opengl3` option will be available only when the `examples` option and the corresponding backends (`sdl2` and `opengl3`) are `ON`. The same is true for all the other examples and they're `OFF` by default.

Libraries and examples can be installed by setting `install` and `install_examples` options `ON`, respectively. An `uninstall` custom target is also provided to remove the artifacts where are they installed.

Following projects are also supported:

| Project | Variable must be set | Build option(s) | Generated target(s) |
|--------|---------------------|-----------------|---------------------|
| [imgui_test_engine](https://github.com/ocornut/imgui_test_engine) | IMGUI_TEST_ENGINE_SOURCE_DIR | imgui_test_engine | <br>`Unofficial::imgui_test_engine::imgui_test_engine`<br>`Unofficial::imgui_test_engine::imgui_app` |
| [imgui_club](https://github.com/ocornut/imgui_club) | IMGUI_CLUB_SOURCE_DIR | imgui_club<br>imgui_memory_editor<br>imgui_multicontext_compositor<br>imgui_threaded_rendering | <br>`Unofficial::imgui_club::imgui_memory_editor`<br>`Unofficial::imgui_club::imgui_multicontext_compositor`<br>`Unofficial::imgui_club::imgui_threaded_rendering` |
| [imgui_markdown](https://github.com/enkisoftware/imgui_markdown) | IMGUI_MARKDOWN_SOURCE_DIR | imgui_markdown | `Unofficial::imgui_markdown::imgui_markdown` |
| [ImPlot](https://github.com/epezent/implot) | IMPLOT_SOURCE_DIR | implot | `Unofficial::ImPlot::implot`<br>`Unofficial::ImPlot::implot_demo` |
| [ImPlot3D](https://github.com/brenocq/implot3d) | IMPLOT3D_SOURCE_DIR | implot3d | `Unofficial::ImPlot3D::implot3d`<br>`Unofficial::ImPlot3D::implot3d_demo` |
| [ImGuiFileDialog](https://github.com/aiekick/ImGuiFileDialog) | IMGUIFILEDIALOG_SOURCE_DIR | imguifiledialog | `Unofficial::ImGuiFileDialog::imguifiledialog` |

## Usage

All the include paths are kept as is. [This repo](https://github.com/adembudak/CMakeForImGui.test) demonstrates how to build programs as a client of the library:

```cmake
# project setup
# ...

find_package(CMakeForImGui CONFIG REQUIRED)
# ...
target_link_libraries(tgt PUBLIC Unofficial::DearImGui::imgui_backend_sdl2 Unofficial::DearImGui::imgui_backend_opengl3)
```
There is also [this branch](https://github.com/adembudak/CMakeForImGui/tree/single-target) available, where a single target approach is investigated. The same options are available but all the backends and ImGui itself is linked with a `Unofficial::DearImGui::imgui` target.

### With `pkg-config` command

The build script can be used to generate **pkg-config** file:
```cmake
cmake -Dpkg-config=ON -S . -B build
```

This generates a `cmakeforimgui.pc` file in build directory which can be installed and used, like:
```bash
c++ -o out main.cpp $(pkg-config --cflags --libs cmakeforimgui)
```

## Using with older versions of the Dear ImGui

Limited amount of checking has done for moved, removed or renamed files of the previous versions of the Dear ImGui. The main branch will work best with recent the versions of the project, i.e. v1.90+. Some work has done for the versions older than 1.80 on [this branch](https://github.com/adembudak/CMakeForImGui/tree/pre.v1.80).

## Help needed

Any kind of feedback, review or input is appreciated and welcome...

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Status](https://github.com/adembudak/CMakeForImGui/actions/workflows/main.yml/badge.svg)](https://github.com/adembudak/CMakeForImGui/actions/workflows/main.yml)
