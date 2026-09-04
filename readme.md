[CMake](https://cmake.org) build support for the [Dear ImGui](https://github.com/ocornut/imgui). It can build and install the library, backends and examples.

This work proposed for the Dear ImGui upstream: https://github.com/ocornut/imgui/issues/8896

## Building

```cmake
cmake -D IMGUI_SOURCE_DIR=path.to.imgui -S . -B build
```

Setting `IMGUI_SOURCE_DIR` variable on configuration step is mandatory. An example usage might be like this:

```cmake
cmake -G 'Ninja Multi-Config' -D IMGUI_SOURCE_DIR=imgui -S . -B build
cmake -D IMGUI_ENABLE_FREETYPE=ON -D DearImGui_Backend_SDL2=ON -D DearImGui_Backend_OpenGL3=ON -S . -B build # Options in imconfig.h, like IMGUI_ENABLE_FREETYPE, can be specified
cmake --build build --config Release
cmake --install build --config Release
```
The commands above assume the dependencies of the backend are installed on the host system. Package managers can be used for that or the user can visit the project site of the used backend and get the source and then follow the build instructions. The [vcpkg](https://vcpkg.io/en) used on this repository.

```cmake
vcpkg install
cmake -D VCPKG_MANIFEST_MODE=ON -D IMGUI_SOURCE_DIR=imgui -S . -B build --toolchain $VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake
```

CMake presets file can be used for easy configuration:
```cmake
cmake --list-presets # list available presets
cmake --preset Default
cmake --build --preset Default --config Debug
cmake --install build --config Debug
```

Following backend options are available:

| Build options                    | Targets                    |
|----------------------------------|----------------------------|
| (default)                        | `ImGui::Core`              |
| DearImGui_Backend_NULL           | `ImGui::Impl_NULL`         |
| DearImGui_Backend_Android        | `ImGui::Impl_Android`      |
| DearImGui_Backend_OpenGL2        | `ImGui::Impl_OpenGL2`      |
| DearImGui_Backend_OpenGL3        | `ImGui::Impl_OpenGL3`      |
| DearImGui_Backend_Vulkan         | `ImGui::Impl_Vulkan`       |
| DearImGui_Backend_Allegro5       | `ImGui::Impl_Allegro5`     |
| DearImGui_Backend_GLFW           | `ImGui::Impl_GLFW`         |
| DearImGui_Backend_FreeGLUT       | `ImGui::Impl_GLUT`         |
| DearImGui_Backend_SDL2           | `ImGui::Impl_SDL2`         |
| DearImGui_Backend_SDLRenderer2   | `ImGui::Impl_SDLRenderer2` |
| DearImGui_Backend_SDL3           | `ImGui::Impl_SDL3`         |
| DearImGui_Backend_SDLGPU3        | `ImGui::Impl_SDLGPU3`      |
| DearImGui_Backend_SDLRenderer3   | `ImGui::Impl_SDLRenderer3` |
| DearImGui_Backend_WebGPU         | `ImGui::Impl_WebGPU`       |
| DearImGui_Backend_Win32          | `ImGui::Impl_Win32`        |
| DearImGui_Backend_DirectX9       | `ImGui::Impl_DirectX9`     |
| DearImGui_Backend_DirectX10      | `ImGui::Impl_DirectX10`    |
| DearImGui_Backend_DirectX11      | `ImGui::Impl_DirectX11`    |
| DearImGui_Backend_DirectX12      | `ImGui::Impl_DirectX12`    |
| DearImGui_Backend_OSX            | `ImGui::Impl_OSX`          |
| DearImGui_Backend_Metal          | `ImGui::Impl_Metal`        |
| DearImGui_Backend_Metal4         | `ImGui::Impl_Metal4`       |

All backend options are `OFF` by default and all the configuration macros on `imconfig.h` can be passed as CMake command line with `-D` variable.

Example programs set as dependent options and will be available when their backends enabled. Libraries and examples can be installed by setting `DearImGui_Install` and `DearImGui_Install_Examples` options `ON`. An `uninstall` custom target is provided to undo the latest install step.

```cmake
cmake --build build --target uninstall
```

## Usage

All the include paths are kept as is. Compiled and installed library can be used with:

```cmake
# project setup
# ...

find_package(ImGui CONFIG REQUIRED)
# ...
target_link_libraries(tgt PUBLIC ImGui::Impl_SDL2 ImGui::Impl_OpenGL3)
```
### With `pkg-config` command

The build script can be used to generate a **pkg-config** file:
```cmake
cmake -D DearImGui_Pkg_Config=ON -S . -B build
```
This generates a `imgui.pc` file in build directory which can be installed and used:
```bash
c++ -o out main.cpp $(pkg-config --cflags --libs imgui)
```
## Alternative approaches

Some alternative designs are also considered:
- On [single-target](https://github.com/adembudak/CMakeForImGui/tree/single-target) branch all the targets are (ImGui itself and backends) linked with a single `Unofficial::DearImGui::imgui` target rather than a target per backend.
- On [pre.v1.80](https://github.com/adembudak/CMakeForImGui/tree/pre.v1.80) it's tried to add support Dear ImGui versions before v1.80.
- On [thirdparties-as-components](https://github.com/adembudak/CMakeForImGui/tree/thirdparties-as-components) branch some thirdparty projects can be specified as `COMPONENTS` option of `find_package()`.

These branches differentiated too much from the main at this point.
- On [CMakeForImGuiThirdParties](https://github.com/adembudak/CMakeForImGuiThirdParties) repository it's shown how thirdparty ImGui software benefit from this work.

## License

This code licensed under [MIT](https://opensource.org/licenses/MIT) license. The projects it depends have their own licenses and should be reviewed in accordance with their respective licenses.

## Help needed

Any kind of feedback, review or input is appreciated and welcome...

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Status](https://github.com/adembudak/CMakeForImGui/actions/workflows/main.yml/badge.svg)](https://github.com/adembudak/CMakeForImGui/actions/workflows/main.yml)
