# Release History

## v3.45.0-1.0.pre-198

- Flutter `3.45.0-1.0.pre-198`
- Framework `0fed394754392b30db4cbce30170eb91675dc923`
- Engine `a70565e489b0c46279f748952c761e394cea3566`
- Dart `3.13.0-edge.814677061617134b666f6b5e3bcc42476911014b`
- Built on old-world LoongArch64 UOS 20 class hardware
- Verified:
  - `dart --version`
  - `gen_snapshot --version`
  - `flutter --version --no-version-check`
  - `flutter --no-version-check doctor -v`
  - old-world GTK desktop app startup with `flutter-linglong-store`

Rebuild notes:

- Replaced the original release asset with a rebuilt SDK on `2026-05-24`.
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
- The Flutter fork URL is non-standard; set `FLUTTER_GIT_URL` to silence that warning.
