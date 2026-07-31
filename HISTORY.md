# Release History

## v3.46.0-1.0.pre-328

- Flutter `3.46.0-1.0.pre-328`
- Framework `1215a75c452ad31502e146f8e621c70b3f2b3689`
- Engine `1215a75c452ad31502e146f8e621c70b3f2b3689`
- Dart `3.13.0-edge.bede5b5178ecb4e0307a9ad81a8026abcf34df24`
- Built on old-world LoongArch64 UOS 20 class hardware
- Release asset:
  `flutter-3.46.0-1.0.pre-328-loongarch64-oldworld-uos20.tar.gz`
- SHA256:
  `c1d58dcd8a5e7f682f6ed754ec2d5d54bd87cd4b4c46287f68c0088640c13d5c`
- Rebuilt and replaced on `2026-07-31` after UOS 20 LoongGPU runtime
  validation exposed a process-wide C++ runtime conflict in the original
  archive.

Validated:

- `flutter --version --no-version-check`
- `flutter create --platforms=linux`
- `flutter build linux --debug --target-platform linux-loong64`
- `flutter build linux --profile --target-platform linux-loong64`
- `flutter build linux --release --target-platform linux-loong64`
- AOT snapshot generation with old-world `gen_snapshot`
- `impellerc` shader compilation
- GTK runner link and install
- bundle RUNPATH: `$ORIGIN/lib`
- debug, profile, and release startup on a real UOS 20 LoongGPU X11 session
- release GTK window creation at 1280x720
- system resolution of `libstdc++.so.6` and `libgcc_s.so.1`

Changes:

- Built debug, profile, and release Linux GTK engine artifacts.
- Linked the GCC 13.4 C++ runtime statically into old-world Engine binaries.
- Removed shared GCC runtime libraries from the Flutter engine cache and
  generated application bundles.
- Kept optional runtime handling in Flutter tool and CMake, but only for files
  that actually exist in the selected engine cache.
- Regenerated `flutter_patched_sdk` with the packaged Dart SDK to replace the
  stale Kernel format 130 artifact with Kernel format 136.
- Guarded native assets install migration so projects without native assets do
  not fail during CMake install.

## v3.45.0-1.0.pre-198

- Flutter `3.45.0-1.0.pre-198`
- Framework `0fed394754392b30db4cbce30170eb91675dc923`
- Engine `a70565e489b0c46279f748952c761e394cea3566`
- Dart `3.13.0-edge.814677061617134b666f6b5e3bcc42476911014b`
- Built on old-world LoongArch64 UOS 20 class hardware

Validated:

- `dart --version`
- `gen_snapshot --version`
- `flutter --version --no-version-check`
- `flutter --no-version-check doctor -v`
- old-world GTK desktop app startup with `flutter-linglong-store`

Rebuild notes:

- Rebuilt the SDK on `2026-05-24`.
- Relinked `libflutter_linux_gtk.so` with old-world GCC 13.4 and binutils 2.42
  to avoid unresolved `R_LARCH_B26` branches left by the system linker.
- Added old-world compatibility for current `dart:ui` native entry points:
  `NativeSemanticsFlags::initSemanticsFlags`,
  `SemanticsUpdateBuilder::updateNode`, and
  `PlatformConfigurationNativeApi::RequestViewFocusChange`.
- Verified that the app no longer reports unresolved native functions,
  `std::system_error`, or startup termination, and that the main GTK window is
  visible on old-world UOS 20.

Known environment warnings from validation:

- Android SDK was not installed.
- Chrome was not installed.
- Maven network check timed out.
- The Flutter fork URL is non-standard; set `FLUTTER_GIT_URL` to silence that
  warning.
