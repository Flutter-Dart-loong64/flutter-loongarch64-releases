# Building Old-World Flutter

This build is produced directly on an old-world LoongArch64 system. Do not use
new-world `loong64` binaries for this target.

## Requirements

- old-world LoongArch64 Linux system
- GCC 13.4 old-world toolchain
- binutils 2.42 old-world toolchain
- Python 3, Ninja, GN, pkg-config
- GTK 3, fontconfig, OpenGL development files
- Flutter framework checkout with LoongArch support
- Engine checkout and Dart SDK checkout matching the framework revision

Confirm the ABI before building:

```bash
uname -m
readelf -l /bin/ls | grep interpreter
```

Expected interpreter:

```text
/lib64/ld.so.1
```

## Source Setup

Use separate checkouts for:

- Flutter framework
- Flutter engine
- Dart SDK
- native asset support packages

The final SDK archive must keep the Flutter framework `.git` directory. The
Flutter tool uses it for version and channel checks.

Apply the support patch from a Flutter checkout root:

```bash
patch -p1 < patches/oldworld-loongarch64-support.patch
```

## Toolchain

Put GCC 13.4 and binutils 2.42 first in `PATH`:

```bash
export PATH="/opt/loongarch64-oldworld-gcc-13.4/bin:/opt/loongarch64-oldworld-binutils-2.42/bin:$PATH"
```

Use the paths used on your own machine. The validated route used GCC 13.4 for
compilation and binutils 2.42 for final linking.

## Engine Configure

Configure release:

```bash
cd engine/src

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

Build profile and debug with the same flags, changing `--runtime-mode` and
`--target-dir`.

## Engine Build

Build the release artifacts:

```bash
ninja -C out/linux_release_loong64_gtk_oldworld \
  gen_snapshot \
  libflutter_linux_gtk.so \
  zip_archives/linux-loong64-release/linux-loong64-flutter-gtk.zip \
  zip_archives/linux-loong64/font-subset.zip
```

Build profile and debug archives as well. The SDK cache should contain:

- `linux-loong64`
- `linux-loong64-profile`
- `linux-loong64-release`

## SDK Assembly

Copy the debug, profile, and release engine zips and tools into Flutter's cache.
Also build and extract `zip_archives/flutter_patched_sdk.zip` from the debug
output so its Kernel format matches the packaged Dart SDK.

Do not copy GCC shared runtime libraries into the engine cache. Confirm that the
Engine and tools contain the GCC 13.4 runtime statically:

```bash
for binary in \
  out/linux_release_loong64_gtk_oldworld/libflutter_linux_gtk.so \
  out/linux_release_loong64_gtk_oldworld/gen_snapshot \
  out/linux_release_loong64_gtk_oldworld/impellerc; do
  if readelf -d "$binary" | grep -q 'libstdc++.so'; then
    echo "unexpected dynamic libstdc++ dependency: $binary" >&2
    exit 1
  fi
done
```

Remove stale shared runtimes when updating an existing cache:

```bash
cache=flutter/bin/cache/artifacts/engine
for mode in linux-loong64 linux-loong64-profile linux-loong64-release; do
  rm -f "$cache/$mode"/libstdc++.so*
  rm -f "$cache/$mode"/libgcc_s.so*
done
```

Rebuild Flutter tool after changing Flutter tool sources:

```bash
rm -f flutter/bin/cache/flutter_tools.stamp flutter/bin/cache/flutter_tools.snapshot
flutter/bin/flutter --version --no-version-check
```

## Validation

Verify engine binaries:

```bash
file out/linux_release_loong64_gtk_oldworld/gen_snapshot
readelf -l out/linux_release_loong64_gtk_oldworld/gen_snapshot | grep interpreter
ldd out/linux_release_loong64_gtk_oldworld/libflutter_linux_gtk.so
```

Expected interpreter for host executables:

```text
/lib64/ld.so.1
```

Build a desktop smoke app:

```bash
flutter create --platforms=linux smoke_app
cd smoke_app
flutter build linux --release --target-platform linux-loong64 --no-version-check
```

Check the bundle:

```bash
find build/linux/loong64/release/bundle/lib -maxdepth 1 -type f | sort
readelf -d build/linux/loong64/release/bundle/smoke_app | grep RUNPATH
```

Expected runtime libraries in the bundle:

- `libflutter_linux_gtk.so`
- `libapp.so`

`libstdc++.so*` and `libgcc_s.so*` must not be present. Verify that the runner
resolves them from the UOS 20 system:

```bash
ldd build/linux/loong64/release/bundle/smoke_app \
  | grep -E 'libstdc\+\+|libgcc_s'
```

Expected RUNPATH:

```text
$ORIGIN/lib
```

## Package

Package from the directory that contains the assembled `flutter/` SDK:

```bash
version=3.46.0-1.0.pre-328
tar -czf "flutter-$version-loongarch64-oldworld-uos20.tar.gz" flutter
sha256sum "flutter-$version-loongarch64-oldworld-uos20.tar.gz" \
  > "flutter-$version-loongarch64-oldworld-uos20.tar.gz.sha256"
sha256sum -c "flutter-$version-loongarch64-oldworld-uos20.tar.gz.sha256"
```
