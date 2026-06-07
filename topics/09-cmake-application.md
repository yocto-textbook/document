# 09. CMake Application and Library

[Back to Learning Path](../README.md#learning-path)

Related Commit:

- `3f6f2d8 application: Add Yocto recipes for CMake-based library and application`
- `370321c external: Add bbappends for CMake projects and register to packagegroup`

## When to Use

Use the `cmake` class when a CMake-based application or library must be built as a Yocto package.

## What This Chapter Covers

This chapter explains how `inherit cmake` provides the standard configure, compile, and install tasks for a CMake project. Compared with the Makefile example, the recipe can stay smaller because the Yocto CMake class knows how to run CMake with the correct cross toolchain file and sysroot.

## Required Additions

| Item | Description |
| --- | --- |
| CMake library recipe | Builds and installs the library through `inherit cmake`. |
| CMake application recipe | Builds and installs an executable that links to the library. |
| library dependency | Makes the library available in the application recipe sysroot. |
| `pkgconfig` | Supports library discovery when the project uses pkg-config. |
| packagegroup `.bbappend` | Adds the CMake example packages to the image. |
| `externalsrc` `.bbappend` | Allows local source builds during development. |

## Project Implementation

```text
.
├── meta-textbook-application
│   ├── recipes-library/hello-cmake-library/hello-cmake-library.bb
│   └── recipes-application/hello-cmake-application/hello-cmake-application.bb
└── meta-textbook-external
    ├── recipes-library/hello-cmake-library/hello-cmake-library.bbappend
    └── recipes-application/hello-cmake-application/hello-cmake-application.bbappend
```

Library recipe:

```bitbake
DESCRIPTION = "This recipe makes shared library with CMake"
SRC_URI = "git://github.com/yocto-textbook/hello-cmake-library.git;protocol=https;branch=main"
SRCREV = "${AUTOREV}"
S = "${WORKDIR}/git"
B = "${WORKDIR}/build"

inherit cmake
```

Application recipe:

```bitbake
DESCRIPTION = "This recipe builds an executable that uses libhello-cmake.so"
SRC_URI = "git://github.com/yocto-textbook/hello-cmake-application.git;protocol=https;branch=main"
SRCREV = "${AUTOREV}"
S = "${WORKDIR}/git"
B = "${WORKDIR}/build"

inherit cmake pkgconfig
DEPENDS = "hello-cmake-library"
```

`B = "${WORKDIR}/build"` keeps generated CMake files out of the source directory. `DEPENDS` ensures the library is staged into the application recipe sysroot before the application build starts.

## Key Takeaway

For standard CMake projects, `inherit cmake` lets Yocto handle most of the build workflow. The recipe mainly identifies the source, build directory, and dependencies. This is a useful contrast with Makefile recipes, where the recipe often has to describe more of the build and install process directly.

## Verification Commands

```sh
bitbake hello-cmake-library hello-cmake-application
grep hello-cmake buildhistory/images/textbook/glibc/textbook-core-image/installed-package-names.txt
```
