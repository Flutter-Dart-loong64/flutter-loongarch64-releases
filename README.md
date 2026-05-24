# Flutter LoongArch64 Old-World Releases

This repository publishes Flutter SDK builds for old-world LoongArch64 Linux
systems.

Old-world LoongArch64 means the system reports `loongarch64` and uses the
old-world dynamic linker, usually `/lib64/ld.so.1`. New-world LoongArch systems
that use `loong64` should use the main `flutter-loong64-releases` repository
instead.

## Current SDK

- Flutter: `3.45.0-1.0.pre-198`
- Framework revision: `0fed394754392b30db4cbce30170eb91675dc923`
- Engine revision: `a70565e489b0c46279f748952c761e394cea3566`
- Dart SDK: `3.13.0-edge.814677061617134b666f6b5e3bcc42476911014b`
- Target system: old-world `loongarch64`, UOS 20 class systems
- Dynamic linker: `/lib64/ld.so.1`
- Rebuilt: `2026-05-24`, with old-world GTK startup and `dart:ui` ABI fixes

## Install

Download the release asset:

```bash
wget https://github.com/Flutter-Dart-loong64/flutter-loongarch64-releases/releases/download/v3.45.0-1.0.pre-198/flutter-3.45.0-1.0.pre-198-loongarch64-oldworld-uos20.tar.gz
wget https://github.com/Flutter-Dart-loong64/flutter-loongarch64-releases/releases/download/v3.45.0-1.0.pre-198/flutter-3.45.0-1.0.pre-198-loongarch64-oldworld-uos20.tar.gz.sha256
sha256sum -c flutter-3.45.0-1.0.pre-198-loongarch64-oldworld-uos20.tar.gz.sha256
tar -xzf flutter-3.45.0-1.0.pre-198-loongarch64-oldworld-uos20.tar.gz
export PATH="$PWD/flutter/bin:$PATH"
flutter --version --no-version-check
```

For a normal shell setup, add this to `.bashrc` or `.zshrc`:

```bash
export PATH="/path/to/flutter/bin:$PATH"
export FLUTTER_GIT_URL="https://github.com/Flutter-Dart-loong64/flutter.git"
```

## Notes

- This SDK is built on an old-world LoongArch64 machine, not cross-built from
  x86_64.
- The package keeps Flutter's `.git` metadata because the Flutter tool uses it
  for channel, version, and feature checks.
- `flutter doctor -v` is expected to warn if Android SDK, Chrome, or network
  access are not configured on the target machine.
- Linux desktop support is enabled for `linux-loong64`.
- Old-world `loongarch64` and new-world `loong64` binaries are not
  interchangeable. Use this repository only for old-world systems using
  `/lib64/ld.so.1`.

## Patch

The old-world engine source changes are kept in
`patches/oldworld-loongarch64-engine.patch`.

Apply it from an engine checkout root, the directory containing `build/` and
`flutter/`:

```bash
patch -p1 < patches/oldworld-loongarch64-engine.patch
```

The full bring-up and rebuild notes are in `OLDWORLD_SUPPORT.md`.
