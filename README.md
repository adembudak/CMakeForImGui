[CMake](https://cmake.org) build script support for [Dear ImGui](https://github.com/ocornut/imgui). This builds and installs the library and backends, also builds example programs.

This work proposed for the Dear ImGui upstream: https://github.com/ocornut/imgui/issues/8896

## Building

```cmake
cmake -D IMGUI_SOURCE_DIR=path.to.imgui -S . -B build
```

Setting `IMGUI_SOURCE_DIR` variable on configuration step is mandatory. An example usage might look like this:

```cmake
cmake -G 'Ninja Multi-Config' -DIMGUI_SOURCE_DIR=imgui -S . -B build
cmake -D DearImGui_Backend_SDL2=ON -D DearImGui_Backend_OpenGL3=ON -S . -B build
cmake --build build --config Release
cmake --install build --config Release
```
The commands above assume the dependencies of the backend are installed on host system. One can visit the project site of used backend and get the source and then follow the build instructions. Another option is using a package
manager. There are several ways to manage dependencies of and build C++ software. This repository used [vcpkg](https://vcpkg.io/en) for running code on GitHub workflow. If your package manager of choice is vcpkg, a vcpkg.json file lists dependencies present in .github/workflows/, so configure command can be:

```cmake
cmake -G 'Ninja Multi-Config' -D VCPKG_MANIFEST_MODE=ON -D VCPKG_MANIFEST_DIR=.github/workflows -D IMGUI_SOURCE_DIR=imgui -S . -B build --toolchain $VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake
# same as above
```
Note that this will not install some platform dependencies like DirectX libraries on Windows, one need to install Microsoft SDK for it or Metal framework on macOS, Xcode needs to be installed.

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

## Usage

All the include paths are kept as is. [This repo](https://github.com/adembudak/CMakeForImGui.test) demonstrates how to build programs as a client of the library:

```cmake
# project setup
# ...

find_package(CMakeForImGui CONFIG REQUIRED)
# ...
target_link_libraries(tgt PUBLIC ImGui::Backend_SDL2 ImGui::Backend_OpenGL3)
```
Some alternative designs are also considered:
- On [single-target](https://github.com/adembudak/CMakeForImGui/tree/single-target) branch all the targets are (ImGui itself and backends) linked with a single `Unofficial::DearImGui::imgui` target rather than a target per backend.
- On [thirdparties-as-components](https://github.com/adembudak/CMakeForImGui/tree/thirdparties-as-components) branch some thirdparty projects can be specified as `COMPONENTS` option of `find_package()`.
- On [pre.v1.80](https://github.com/adembudak/CMakeForImGui/tree/pre.v1.80) the build script tries to support Dear ImGui versions before v1.80.

### With `pkg-config` command

The build script can be used to generate **pkg-config** file:
```cmake
cmake -Dpkg-config=ON -S . -B build
```
This generates a `dearimgui.pc` file in build directory which can be installed and used, like:
```bash
c++ -o out main.cpp $(pkg-config --cflags --libs dearimgui)
```

## License

This code licensed under [MIT](https://opensource.org/licenses/MIT) license. The projects it depends have their own licenses and should be reviewed in accordance with their respective licenses.

## Help needed

Any kind of feedback, review or input is appreciated and welcome...

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Status](https://github.com/adembudak/CMakeForImGui/actions/workflows/main.yml/badge.svg)](https://github.com/adembudak/CMakeForImGui/actions/workflows/main.yml)
