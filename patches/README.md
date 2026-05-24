# Patches

`oldworld-loongarch64-engine.patch` contains the engine-side source changes used
for the old-world LoongArch64 SDK build.

The patch is intended to be applied from an engine checkout root:

```bash
patch -p1 < patches/oldworld-loongarch64-engine.patch
```

It covers:

- GCC old-world build compatibility.
- `/lib64/ld.so.1` old-world runtime compatibility.
- disabling LSX/LASX-only paths when the old-world toolchain lacks those flags.
- avoiding SwiftShader test dependencies on Loong64 where the bundled LLVM
  backend does not support LoongArch.
- missing standard includes exposed by GCC.
- static GCC runtime linkage for old-world shared libraries.
- current `dart:ui` GTK startup ABI compatibility, including viewport metrics,
  semantics flags, semantics updates, and view focus native entry points.
