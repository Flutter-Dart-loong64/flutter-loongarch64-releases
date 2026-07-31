# Old-World LoongArch64 Flutter Bring-Up

This document records the old-world LoongArch64 route used for the current UOS
20 SDK release.

## Target

- Architecture reported by system: `loongarch64`
- ABI family: old-world ABI
- Dynamic linker: `/lib64/ld.so.1`
- Flutter target platform name: `linux-loong64`
- Current SDK version: `3.46.0-1.0.pre-328`

Old-world and new-world LoongArch binaries are not compatible. New-world systems
normally use `/lib64/ld-linux-loongarch-lp64d.so.1`; old-world systems use
`/lib64/ld.so.1`.

## Toolchain

The validated route used:

- GCC 13.4 old-world toolchain
- binutils 2.42 old-world toolchain

The old system linker left unresolved `R_LARCH_B26` branches in some Flutter
engine shared-library constructors. Rebuilding with the newer old-world
GCC/binutils pair removed those dynamic branch relocations.

## Patch Scope

`patches/oldworld-loongarch64-support.patch` covers:

- GCC build compatibility where upstream assumes Clang warnings or Clang-only
  flags.
- old-world LoongArch builds without relying on LSX/LASX compiler flags that
  are not available in the target toolchain.
- missing standard includes exposed by GCC.
- Vulkan/SwiftShader/ANGLE build fixes needed by the old-world compiler.
- Linux GTK engine artifact plumbing for `linux-loong64`.
- static GCC C++ runtime linking for old-world Loong64 Engine targets.
- optional Flutter tool and Linux CMake handling for runtime libraries that are
  actually present in an engine cache.
- guarded native assets install migration for projects without native assets.

## Engine Build

The final SDK includes debug, profile, and release Linux GTK engine artifacts.
Release was configured with fontconfig enabled and Vulkan disabled:

```bash
python3 ./flutter/tools/gn \
  --linux \
  --linux-cpu loong64 \
  --runtime-mode release \
  --enable-fontconfig \
  --no-enable-unittests \
  --no-goma \
  --no-clang \
  --target-sysroot / \
  --prebuilt-dart-sdk \
  --target-dir linux_release_loong64_gtk_oldworld \
  --gn-args='is_qnx=false system_libdir="lib/loongarch64-linux-gnu" skia_use_vulkan=false shell_enable_vulkan=false impeller_enable_vulkan=false test_enable_vulkan=false glfw_vulkan_library=""'
```

The same configuration pattern was used for debug and profile with matching
target directories.

## SDK Runtime Libraries

The Engine requires GCC 13.4 language and library support, while UOS 20 ships an
older system `libstdc++` used by its LoongGPU user-space driver. Loading the GCC
13.4 shared `libstdc++` process-wide causes `gsgpu_dri.so` and
`libloong-gpucomp.so.1` to fail during initialization.

Old-world Engine binaries are therefore linked with `-static-libstdc++` and
`-static-libgcc`. The Engine export map keeps those implementations private.
`libflutter_linux_gtk.so`, `gen_snapshot`, `impellerc`, and the other packaged
Engine tools have no dynamic `libstdc++.so.6` dependency.

The Flutter engine cache and generated application bundles do not contain
`libstdc++.so*` or `libgcc_s.so*`. The GTK runner uses the UOS 20 system
libraries, preserving compatibility with the system LoongGPU stack.

## Flutter Tool Changes

The Linux unpack target and CMake template retain optional support for runtime
libraries supplied by another self-contained engine cache. Files are copied,
linked, and installed only when they actually exist. `linux-loong64` no longer
forces absent runtime files into the build graph.

The installed executable uses:

```text
RUNPATH $ORIGIN/lib
```

The RUNPATH locates `libflutter_linux_gtk.so` in the application bundle. With
no bundled GCC runtime, `libstdc++` and `libgcc_s` resolve from UOS 20.

## Validation

The current SDK was validated with a generated Linux desktop app:

```bash
flutter create --platforms=linux smoke_app
cd smoke_app
flutter build linux --debug --target-platform linux-loong64 --no-version-check
flutter build linux --profile --target-platform linux-loong64 --no-version-check
flutter build linux --release --target-platform linux-loong64 --no-version-check
```

Verified build stages:

- Flutter tool snapshot rebuild
- Dart pub dependency resolution
- patched SDK generation matching the packaged Dart Kernel format
- `gen_snapshot` AOT output
- `impellerc` shader compilation
- GTK runner CMake configure
- Ninja build and install
- app bundle generation
- debug, profile, and release startup on a real UOS 20 X11 LoongGPU session

Verified bundle files:

- `flutter_oldworld_smoke`
- `lib/libflutter_linux_gtk.so`
- `lib/libapp.so`

The bundle executable reported:

```text
RUNPATH Library runpath: [$ORIGIN/lib]
```

`ldd` resolves the runner's `libstdc++.so.6` and `libgcc_s.so.1` from the UOS
20 system. A 1280x720 GTK window was observed during the release-mode LoongGPU
startup test.

## Package

Current package:

```text
flutter-3.46.0-1.0.pre-328-loongarch64-oldworld-uos20.tar.gz
```

SHA256:

```text
c1d58dcd8a5e7f682f6ed754ec2d5d54bd87cd4b4c46287f68c0088640c13d5c
```

The package keeps the Flutter `.git` metadata so the Flutter tool can report
version and channel information correctly.
