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
- Flutter tool support for optional bundled Linux runtime libraries.
- Linux CMake template support for linking and installing bundled runtime
  libraries.
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

The old-world engine is linked with GCC 13.4 C++ runtime symbols that are newer
than the system `libstdc++` on UOS 20. The SDK therefore ships GCC runtime
libraries through the Flutter engine cache.

Each Linux engine cache directory contains:

- `libstdc++.so`
- `libstdc++.so.6`
- `libgcc_s.so`
- `libgcc_s.so.1`

The unversioned files are used by the app link step. The versioned files are
loaded at runtime from the app bundle.

## Flutter Tool Changes

The Linux unpack target copies optional bundled runtime libraries from the
engine cache into `linux/flutter/ephemeral`.

The Linux CMake template does three things for `linux-loong64`:

- declares bundled runtime libraries as `flutter_assemble` outputs.
- links the GTK runner against those libraries from the ephemeral directory.
- installs the libraries into the app bundle `lib/` directory.

The installed executable uses:

```text
RUNPATH $ORIGIN/lib
```

This makes the app prefer the bundled old-world GCC runtime at startup.

## Validation

The current SDK was validated with a generated Linux desktop app:

```bash
flutter create --platforms=linux smoke_app
cd smoke_app
flutter build linux --release --target-platform linux-loong64 --no-version-check
```

Verified build stages:

- Flutter tool snapshot rebuild
- Dart pub dependency resolution
- `gen_snapshot` AOT output
- `impellerc` shader compilation
- GTK runner CMake configure
- Ninja build and install
- app bundle generation

Verified bundle files:

- `flutter_oldworld_smoke`
- `lib/libflutter_linux_gtk.so`
- `lib/libapp.so`
- `lib/libstdc++.so`
- `lib/libstdc++.so.6`
- `lib/libgcc_s.so`
- `lib/libgcc_s.so.1`

The bundle executable reported:

```text
RUNPATH Library runpath: [$ORIGIN/lib]
```

## Package

Current package:

```text
flutter-3.46.0-1.0.pre-328-loongarch64-oldworld-uos20.tar.gz
```

SHA256:

```text
12cdf2588d9753e2392101eb61740395faaa66a66c731a95a143eea926d49ad7
```

The package keeps the Flutter `.git` metadata so the Flutter tool can report
version and channel information correctly.
