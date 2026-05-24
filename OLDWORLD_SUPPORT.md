# Old-World LoongArch64 Flutter Bring-Up

This document records the old-world LoongArch64 Flutter SDK bring-up path used
for the `v3.45.0-1.0.pre-198` release.

## Target

- Architecture: old-world `loongarch64`
- ABI family: ABI1.0 old-world
- Dynamic linker: `/lib64/ld.so.1`
- Validated system: UOS Desktop 20 Professional
- Flutter target platform name used by the tool: `linux-loong64`

Old-world and new-world LoongArch binaries are not compatible. New-world
systems normally use `/lib64/ld-linux-loongarch-lp64d.so.1`; old-world systems
use `/lib64/ld.so.1`.

## Source Layout

Use separate checkouts for the framework, engine, Dart SDK, and native support
forks. The release package is assembled from the Flutter framework checkout
after the engine and Dart SDK artifacts have been copied into `bin/cache`.

The Flutter framework `.git` directory must stay in the release archive. The
Flutter tool reads it for channel, version, and feature checks.

## Toolchain Notes

The old-world UOS 20 system linker was not sufficient for the final GTK engine
shared library. It left unresolved `R_LARCH_B26` branches in constructors, which
showed up as `b 0` or `bl 0` self-branches and caused `libflutter_linux_gtk.so`
initialization to hang.

The validated rebuild used an old-world GCC 13.4 driver with binutils 2.42 for
the final link. After rebuilding, `readelf -rW libflutter_linux_gtk.so` reported
no dynamic `R_LARCH_B26` leftovers.

## Patch Scope

The patch in `patches/oldworld-loongarch64-engine.patch` covers:

- GCC build compatibility where upstream build files assume Clang warnings or
  Clang-only options.
- static GCC runtime linkage for engine shared libraries.
- old-world LoongArch builds without relying on LSX/LASX-only source paths.
- missing standard includes exposed by GCC.
- Linux GTK engine artifact plumbing for `linux-loong64`.
- current `dart:ui` ABI compatibility needed by the Flutter framework:
  viewport metrics, application locale, semantics tree enablement, view-aware
  semantics updates, native semantics flags, and view focus requests.

## Configure

```bash
cd engine/src
patch -p1 < /path/to/flutter-loongarch64-releases/patches/oldworld-loongarch64-engine.patch

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

## Build

```bash
ninja -C out/linux_release_loong64_gtk_oldworld \
  gen_snapshot \
  libflutter_linux_gtk.so \
  libflutter_engine.so \
  zip_archives/linux-loong64-release/linux-loong64-flutter-gtk.zip \
  zip_archives/linux-loong64-release/artifacts.zip \
  zip_archives/linux-loong64/font-subset.zip
```

If the system linker leaves branch relocations unresolved, use an old-world
GCC/binutils wrapper for the final Ninja invocation:

```bash
export PATH=/path/to/oldworld-gcc-binutils-wrapper:$PATH
ninja -C out/linux_release_loong64_gtk_oldworld libflutter_linux_gtk.so
```

## Assemble SDK

```bash
out=/path/to/engine/src/out/linux_release_loong64_gtk_oldworld
cache=/path/to/flutter/bin/cache/artifacts/engine

for mode in linux-loong64 linux-loong64-profile linux-loong64-release; do
  mkdir -p "$cache/$mode"
  cp "$out/libflutter_linux_gtk.so" "$cache/$mode/libflutter_linux_gtk.so"
  cp "$out/gen_snapshot" "$cache/$mode/gen_snapshot"
  cp "$out/icudtl.dat" "$cache/$mode/icudtl.dat"
done
```

## Runtime Fixes Verified

The rebuilt engine fixed these old-world startup failures:

- GTK process stuck during `libflutter_linux_gtk.so` initialization because of
  unresolved branch relocations.
- 10x10 placeholder window with no rendered Flutter UI.
- unresolved `dart:ui` native functions:
  - `NativeSemanticsFlags::initSemanticsFlags`
  - `PlatformConfigurationNativeApi::RequestViewFocusChange`
- semantics update signature mismatch between the framework and engine.

Validation with `flutter-linglong-store` on old-world UOS 20:

- process stays alive after launch.
- `xwininfo` reports the main `玲珑应用商店社区版` window at `1280x800`.
- log contains no unresolved native function, `std::system_error`, or startup
  termination.
- remaining GLib/XDG desktop portal settings warnings do not block the GUI.

## Package

```bash
cd /path/to/flutter-parent
tar -czf flutter-3.45.0-1.0.pre-198-loongarch64-oldworld-uos20.tar.gz flutter
sha256sum flutter-3.45.0-1.0.pre-198-loongarch64-oldworld-uos20.tar.gz \
  > flutter-3.45.0-1.0.pre-198-loongarch64-oldworld-uos20.tar.gz.sha256
```
