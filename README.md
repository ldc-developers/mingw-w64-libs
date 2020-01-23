# mingw-w64-libs

Microsoft's license doesn't permit bundling other prebuilt compilers with their WinSDK & Visual C++ runtime libraries (for
linking, i.e., either the static libs or the import libs for the dynamic runtime).

This repository consists of a set of tools to generate import libraries for the Windows and Visual C++ runtime DLLs, intended
as drop-in replacement for the official Microsoft import libraries.

The [MinGW-w64 runtime](https://sourceforge.net/projects/mingw-w64/files/mingw-w64/mingw-w64-release/) is based on custom
import libraries for the Microsoft DLLs, but also extends the MS runtime for Posix/GNU compatibility and overrides some
functions, e.g., because MinGW uses 80-bit extended precision for `long double`, whereas Microsoft uses 64-bit double precision.

The generated import libraries here are based on the .def definition files from MinGW-w64, but don't include any MinGW runtime
additions. The libraries are built by the [buildsdk.d](https://github.com/ldc-developers/mingw-w64-libs/blob/master/buildsdk.d)
tool; see the top comment in that file for how to use it.

The basic approach is:

1. Patch the MinGW-w64 .def files by trying to undo all MinGW-specific overriding of Microsoft functions.
2. Ignore the aliases in the .def files and instead use verified aliases in the `*.aliases{32,64}` files. These files have been
generated via the [extractAliases.d](https://github.com/ldc-developers/mingw-w64-libs/blob/master/extractAliases.d) tool, which
parses the `dumpbin /symbols` output of Microsoft .lib files.
3. Generate .lib import libraries based on the patched .def files.
4. Provide a minimal C runtime skeleton in the `msvcrt_*.c` files, providing the `mainCRTStartup` / `WinMainCRTStartup` /
`_DllMainCRTStartup` entry points, invoking the C runtime constructors/destructors, `atexit` handlers etc. These objects are
merged into the generated `vcruntime140.lib` library (targeting the Visual C++ 2015 runtime) and should be enough for a typical
D application.

Evolution of this approach:

* https://github.com/dlang/installer/pull/289
* https://github.com/dlang/installer/pull/346
