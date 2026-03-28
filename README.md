# Flutter engine binaries for armv7, aarch64, x64

This repo contains custom builds of the flutter engine for Linux armv7, aarch64 and x64 architectures. Only optimized debug and release builds are provided. 

This is a fork of [https://github.com/ardera/flutter-ci](https://github.com/ardera/flutter-ci), and doesn't include tuned builds for the raspberry pi, builds for windows or mac, nor builds for riscv.

# 📦 Downloads

### **See [Releases](https://github.com/JakeSays/flutter-build-ci/releases).**

*NOTE*: This repo only provides builds for SDK 3.41.4 and beyond. For older builds see [Upstream](https://github.com/ardera/flutter-ci/releases).
The SDK semantic versions are only provided because they're a bit nicer to read than the raw engine commit hashes, but the engine commits are still the "single source of truth"

# 🛠️ Build Config and Compiler Invocation
## Build Config
The engine build is configured with: [^2]
```
$ ./src/flutter/tools/gn \
  --runtime-mode <debug / profile / release> \
  [--unoptimized]
  --target-os linux \
  --linux-cpu <arm / arm64 / x64> \
  [--arm-float-abi hard] \
  --target-dir build \
  --embedder-for-target \
  --disable-desktop-embeddings \
  --no-build-glfw-shell \
  --no-build-embedder-examples \
  --no-goma
```

After that, the following args are added to the `args.gn` file for armv7/aarch64 without any CPU-specific tuning:
```
arm_cpu = "generic"
arm_tune = "generic"
```

For both armv7 and aarch64, the engine is built against the sysroot provided by the engine build scripts, which is some debian sid sysroot from 2020.
(See https://github.com/flutter/buildroot/blob/master/build/linux/sysroot_scripts/install-sysroot.py)

## Compiler Invocation
This will result in the clang compiler being invoked with the following args:

| artifact        | compiler arguments                                           |
| --------------- | ------------------------------------------------------------ |
| armv7-generic   | `--target=armv7-linux-gnueabihf    -mcpu=generic             -mtune=generic` |
| aarch64-generic | `--target=aarch64-linux-gnu        -mcpu=generic             -mtune=generic` |
| x64-generic     | `--target=x86_64-unknown-linux-gnu -mcpu=generic             -mtune=generic` |

## Debug Symbols

Some modifications are made to the engine build scripts so it's always built with `-ggdb -fdebug-default-version=4`.
The debug symbols are then split into a separate file using:
```bash
$ objcopy --only-keep-debug libflutter_engine.so libflutter_engine.so.dbgsyms
$ objcopy --strip-debug libflutter_engine.so
$ objcopy --add-gnu-debuglink=libflutter_engine.{debug/profile/release/debug_unopt}.dbgsyms libflutter_engine.so
```

That means you can just later download the debug symbols when you need them and step through the engine source code.

However, the resulting `libflutter_engine.so` is ~4MBs larger than one that has _all_ (not only debug symbols) stripped.
So, if you want to save a few more megabytes you can strip them using:
```bash
objcopy --strip-unneeded libflutter_engine.so
```
