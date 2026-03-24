# HackArena3 C++ SDK Consumer Example

This is a standalone consumer project that loads the released `hackarena3` C++ SDK package with `find_package(hackarena3 CONFIG REQUIRED)`.

## What you need

- Visual Studio toolchain in CLion with Architecture `amd64`
- the released SDK zip copied into `artifacts/`, or an extracted SDK directory provided via `HACKARENA3_SDK_ROOT`

For the packaged Windows SDK zip consumer flow, `vcpkg` is not required.

## Configure from terminal

Default local-zip workflow:

1. Copy the SDK zip into:

   `artifacts\hackarena3-cpp-sdk-0.1.0b4-Windows-AMD64.zip`

2. Configure:

   ```powershell
   cmake -S . -B build -G "Visual Studio 17 2022" -A x64
   ```

Explicit extracted-SDK workflow:

```powershell
cmake -S . -B build -G "Visual Studio 17 2022" -A x64 ^
  -DHACKARENA3_SDK_ROOT=C:\sdk\hackarena3-cpp-sdk-0.1.0b4-Windows-AMD64
```

## Build

```powershell
cmake --build build --config Release
```

## Run

```powershell
$env:HA3_WRAPPER_API_URL='https://ha3-api.dev.hackarena.pl/'
$env:HA_AUTH_PROFILE='preprod'
.\build\Release\bot.exe
```

## CLion setup

Use CMake options like this:

```text
-DHACKARENA3_SDK_ROOT=C:\sdk\hackarena3-cpp-sdk-0.1.0b4-Windows-AMD64
```

Notes:

- toolchain must be `Visual Studio`
- architecture must be `amd64`
- do not use MinGW with the `x64-windows` SDK/dependency set
- if `artifacts/hackarena3-cpp-sdk-*.zip` exists, the template auto-extracts and uses it by default

## How loading works

The SDK zip contains:

- `include/`
- `lib/`
- `lib/cmake/hackarena3/hackarena3Config.cmake`
- `vendor/` with bundled third-party dependencies

This project works in two modes:

- if `HACKARENA3_SDK_ROOT` is set, it adds that path to `CMAKE_PREFIX_PATH`
- otherwise it looks for a single `artifacts/hackarena3-cpp-sdk-*.zip`, extracts it into the build directory, and uses that extracted SDK root

Then CMake resolves:

```cmake
find_package(hackarena3 CONFIG REQUIRED)
```

and links:

```cmake
target_link_libraries(bot PRIVATE hackarena3::hackarena3)
```

On Windows, if the SDK package exposes `hackarena3_copy_runtime_dlls`, this template uses it automatically so the built `bot.exe` gets the bundled runtime DLLs copied next to it.
