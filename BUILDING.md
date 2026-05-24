# Building Old-World Flutter

The release is produced directly on an old-world LoongArch64 machine. Do not
reuse new-world `loong64` binaries for this target.

## Requirements

- old-world LoongArch64 Linux system
- GCC/binutils that can emit binaries using `/lib64/ld.so.1`
- Python 3, Ninja, GN, pkg-config, GTK 3, fontconfig, OpenGL development files
- Flutter engine checkout with the old-world patch applied
- Dart SDK checkout with Loong64 VM support built for the same old-world system

## From Zero

1. Prepare an old-world LoongArch64 system and confirm the dynamic linker:

```bash
uname -m
readelf -l /bin/ls | grep interpreter
```

Expected interpreter:

```text
/lib64/ld.so.1
```

2. Fetch the Flutter framework, engine, Dart SDK, and native support forks.
   Keep the framework `.git` directory because Flutter uses it for version and
   feature checks.

3. Apply the old-world engine patch from this repository:

```bash
cd engine/src
patch -p1 < /path/to/flutter-loongarch64-releases/patches/oldworld-loongarch64-engine.patch
```

4. Use an old-world-capable GCC/binutils pair. The validated rebuild used GCC
   13.4 with binutils 2.42 for final linking, because the system linker from
   old-world UOS 20 did not correctly resolve `R_LARCH_B26`.

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

## SDK Assembly

Copy the rebuilt engine artifacts into Flutter's cache:

```bash
cache=/path/to/flutter/bin/cache/artifacts/engine
out=/path/to/engine/src/out/linux_release_loong64_gtk_oldworld

for mode in linux-loong64 linux-loong64-profile linux-loong64-release; do
  mkdir -p "$cache/$mode"
  cp "$out/libflutter_linux_gtk.so" "$cache/$mode/libflutter_linux_gtk.so"
  cp "$out/gen_snapshot" "$cache/$mode/gen_snapshot"
  cp "$out/icudtl.dat" "$cache/$mode/icudtl.dat"
done
```

Then rebuild a Linux app with:

```bash
flutter --no-version-check build linux --release --target-platform=linux-loong64
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

For the GTK startup fix, `flutter-linglong-store` was used as a smoke app. A
successful run creates a `玲珑应用商店社区版` X11 window instead of hanging at a
10x10 placeholder window.
