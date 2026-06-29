# Patches

## Current

`oldworld-loongarch64-support.patch` contains the source changes used by the
current old-world LoongArch64 SDK build.

Apply it from a Flutter checkout root:

```bash
patch -p1 < patches/oldworld-loongarch64-support.patch
```

It covers:

- GCC 13.4 old-world build compatibility.
- binutils 2.42 old-world link compatibility.
- `/lib64/ld.so.1` runtime compatibility.
- LoongArch builds without LSX/LASX compiler flags.
- missing standard includes and GCC warning fixes.
- Flutter tool copying for bundled Linux runtime libraries.
- Linux CMake linking and install support for bundled GCC runtime libraries.
- native assets install migration guard.

## Historical

`oldworld-loongarch64-engine.patch` is the earlier engine-only patch kept for
the `v3.45.0-1.0.pre-198` release record.
