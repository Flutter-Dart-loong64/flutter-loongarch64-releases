# Building

The release is produced directly on an old-world LoongArch64 machine.

## Requirements

- old-world LoongArch64 Linux system
- GCC/binutils that can emit binaries using `/lib64/ld.so.1`
- Python 3, Ninja, GN, pkg-config, GTK 3, fontconfig, OpenGL development files
- Flutter engine checkout with the old-world patch applied
- Dart SDK checkout with Loong64 VM support built for the same old-world system

## Engine Configure

The engine build uses the Linux desktop target with fontconfig enabled:

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

## Engine Build

```bash
ninja -C out/linux_release_loong64_gtk_oldworld \
  gen_snapshot \
  libflutter_linux_gtk.so \
  libflutter_engine.so \
  zip_archives/linux-loong64-release/linux-loong64-flutter-gtk.zip \
  zip_archives/linux-loong64-release/artifacts.zip \
  zip_archives/linux-loong64/font-subset.zip
```

## Validation

```bash
file out/linux_release_loong64_gtk_oldworld/gen_snapshot
readelf -l out/linux_release_loong64_gtk_oldworld/gen_snapshot | grep interpreter
ldd out/linux_release_loong64_gtk_oldworld/libflutter_linux_gtk.so
```

Expected interpreter for host executables:

```text
/lib64/ld.so.1
```

After assembling the SDK cache:

```bash
flutter --version --no-version-check
flutter --no-version-check doctor -v
```

The validated old-world machine reported:

```text
Linux (desktop) • linux • linux-loong64 • UOS Desktop 20 Professional
```
