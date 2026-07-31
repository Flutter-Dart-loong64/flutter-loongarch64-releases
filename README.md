# Flutter LoongArch64 Old-World Releases

This repository publishes Flutter SDK builds for old-world LoongArch64 Linux
systems.

Old-world LoongArch64 systems report `loongarch64` and use the old ABI dynamic
linker, normally `/lib64/ld.so.1`. New-world LoongArch systems use the
`loong64` SDK from `flutter-loong64-releases`.

## Current SDK

- Version: `3.46.0-1.0.pre-328`
- Flutter framework: `1215a75c452ad31502e146f8e621c70b3f2b3689`
- Engine: `1215a75c452ad31502e146f8e621c70b3f2b3689`
- Dart SDK: `3.13.0-edge.bede5b5178ecb4e0307a9ad81a8026abcf34df24`
- Target system: old-world `loongarch64`, UOS 20 class systems
- Flutter target platform name: `linux-loong64`
- Dynamic linker: `/lib64/ld.so.1`

## Install

```bash
wget https://github.com/Flutter-Dart-loong64/flutter-loongarch64-releases/releases/download/v3.46.0-1.0.pre-328/flutter-3.46.0-1.0.pre-328-loongarch64-oldworld-uos20.tar.gz
wget https://github.com/Flutter-Dart-loong64/flutter-loongarch64-releases/releases/download/v3.46.0-1.0.pre-328/flutter-3.46.0-1.0.pre-328-loongarch64-oldworld-uos20.tar.gz.sha256
sha256sum -c flutter-3.46.0-1.0.pre-328-loongarch64-oldworld-uos20.tar.gz.sha256
tar -xzf flutter-3.46.0-1.0.pre-328-loongarch64-oldworld-uos20.tar.gz
export PATH="$PWD/flutter/bin:$PATH"
flutter --version --no-version-check
```

For a regular shell setup:

```bash
export PATH="/opt/flutter-loongarch64-oldworld/flutter/bin:$PATH"
export FLUTTER_GIT_URL="https://github.com/Flutter-Dart-loong64/flutter.git"
```

Use a directory that matches your own installation location.

## Notes

- The SDK is built on real old-world LoongArch64 hardware.
- The archive keeps Flutter's `.git` metadata. The Flutter tool reads it for
  version, channel, and feature checks.
- Linux desktop builds use `--target-platform linux-loong64`.
- The SDK cache includes debug, profile, and release GTK engine artifacts.
- GCC 13.4 C++ support is linked statically into the old-world Engine. Generated
  applications use the UOS 20 system `libstdc++` and `libgcc_s`, so proprietary
  LoongGPU drivers are not forced into a newer process-wide C++ runtime.
- Old-world `loongarch64` and new-world `loong64` binaries are not
  interchangeable.

## Patch

The current source patch is kept in:

```text
patches/oldworld-loongarch64-support.patch
```

Apply it from a Flutter checkout root that contains `engine/` and `packages/`:

```bash
patch -p1 < patches/oldworld-loongarch64-support.patch
```

The old engine-only patch from the earlier release is still kept for reference.
Build notes are in `BUILDING.md`, and the bring-up record is in
`OLDWORLD_SUPPORT.md`.
