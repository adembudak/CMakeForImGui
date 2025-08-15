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

<table>
  <thead>
    <tr>
      <th>Build option</th>
      <th>Generated target</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ImGui Demo</td>
      <td><code>Unofficial::DearImGui::imgui_demo</code></td>
    </tr>
    <tr><td>(default)</td><td rowspan="20"><code>Unofficial::DearImGui::imgui</code></td></tr>
    <tr><td>android</td></tr>
    <tr><td>opengl2</td></tr>
    <tr><td>opengl3</td></tr>
    <tr><td>vulkan</td></tr>
    <tr><td>allegro5</td></tr>
    <tr><td>glfw</td></tr>
    <tr><td>glut</td></tr>
    <tr><td>sdl2</td></tr>
    <tr><td>sdlrenderer2</td></tr>
    <tr><td>sdl3</td></tr>
    <tr><td>sdlgpu3</td></tr>
    <tr><td>sdlrenderer3</td></tr>
    <tr><td>win32</td></tr>
    <tr><td>dx9</td></tr>
    <tr><td>dx10</td></tr>
    <tr><td>dx11</td></tr>
    <tr><td>dx12</td></tr>
    <tr><td>osx</td></tr>
    <tr><td>metal</td></tr>
  </tbody>
</table>

All backend options are `OFF` by default and every enabled and built backends merged into the `Unofficial::DearImGui::imgui` target.

Example programs set as dependent options, like:
```cmake
cmake_dependent_option(example_sdl2_opengl3 "" OFF "examples AND sdl2 AND opengl3" OFF)
```
The `example_sdl2_opengl3` option will be available only when the `examples` option and the corresponding backends (`sdl2` and `opengl3`) are `ON`. The same is true for all the other examples and they're `OFF` by default.

Libraries and examples can be installed by setting `install` and `install_examples` options `ON`, respectively. An `uninstall` custom target is also provided to remove the artifacts where are they installed.

## Usage

All the include paths are kept as is. [This repo](https://github.com/adembudak/CMakeForImGui.test) demonstrates how to build programs as a client of the library:

```cmake
# project setup
# ...

find_package(CMakeForImGui CONFIG REQUIRED)
# ...
target_link_libraries(tgt PUBLIC Unofficial::DearImGui::imgui)
```
### With `pkg-config` command

The build script can be used to generate **pkg-config** file:
```cmake
cmake -Dpkg-config=ON -S . -B build
```

This generates a `cmakeforimgui.pc` file in build directory which can be installed and used, like:
```bash
c++ -o out main.cpp $(pkg-config --cflags --libs cmakeforimgui)
```

## Help needed

Any kind of feedback, review or input is appreciated and welcome...

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Status](https://github.com/adembudak/CMakeForImGui/actions/workflows/main.yml/badge.svg)](https://github.com/adembudak/CMakeForImGui/actions/workflows/main.yml)
