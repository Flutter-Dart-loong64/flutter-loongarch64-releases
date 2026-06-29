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
  `12cdf2588d9753e2392101eb61740395faaa66a66c731a95a143eea926d49ad7`

Validated:

- `flutter --version --no-version-check`
- `flutter create --platforms=linux`
- `flutter build linux --release --target-platform linux-loong64`
- AOT snapshot generation with old-world `gen_snapshot`
- `impellerc` shader compilation
- GTK runner link and install
- bundle RUNPATH: `$ORIGIN/lib`
- bundled runtime libraries:
  `libstdc++.so`, `libstdc++.so.6`, `libgcc_s.so`, `libgcc_s.so.1`

Changes:

- Built debug, profile, and release Linux GTK engine artifacts.
- Added old-world GCC 13.4 runtime libraries to the Flutter engine cache.
- Taught Flutter tool's Linux unpack step to copy optional bundled runtime
  libraries from the engine cache.
- Updated the Linux CMake template to link and install bundled runtime
  libraries for `linux-loong64`.
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
