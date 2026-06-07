# 09. CMake 기반 application과 library

[Back to Learning Path](../README.md#learning-path)

Related Commit:

- `3f6f2d8 application: Add Yocto recipes for CMake-based library and application`
- `370321c external: Add bbappends for CMake projects and register to packagegroup`

## When to Use

CMake 프로젝트를 Yocto package로 만들고, CMake 기반 application이 CMake 기반 library에 link되게 하고 싶다면 `cmake` class를 사용한다.

## What This Chapter Covers

이 chapter는 `inherit cmake`가 CMake project의 configure, compile, install task를 어떻게 제공하는지 설명한다. recipe가 source, build directory, dependency만 명시해도 class가 표준 build flow를 만들어 주는 구조를 다룬다.

## Required Additions

| 항목 | 역할 |
| --- | --- |
| CMake library recipe | `inherit cmake`로 library build/install |
| CMake application recipe | library를 link하는 executable build/install |
| library dependency | application recipe의 build-time dependency |
| `pkgconfig` | library discovery가 필요한 경우 pkg-config support |
| packagegroup `.bbappend` | image에 CMake 예제 package 포함 |
| `externalsrc` `.bbappend` | local source 기반 개발 build |

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

library recipe:

```bitbake
DESCRIPTION = "This recipe makes shared library with CMake"
SRC_URI = "git://github.com/yocto-textbook/hello-cmake-library.git;protocol=https;branch=main"
SRCREV = "${AUTOREV}"
S = "${WORKDIR}/git"
B = "${WORKDIR}/build"

inherit cmake
```

application recipe:

```bitbake
DESCRIPTION = "This recipe builds an executable that uses libhello-cmake.so"
SRC_URI = "git://github.com/yocto-textbook/hello-cmake-application.git;protocol=https;branch=main"
SRCREV = "${AUTOREV}"
S = "${WORKDIR}/git"
B = "${WORKDIR}/build"

inherit cmake pkgconfig
DEPENDS = "hello-cmake-library"
```

## Key Takeaway

CMake 프로젝트는 `inherit cmake`만으로 configure, compile, install workflow 대부분을 Yocto class에 맡길 수 있다. 그래서 Makefile 예제와 비교하면 Yocto class의 가치가 잘 보인다.

## Verification Commands

```sh
bitbake hello-cmake-library hello-cmake-application
grep hello-cmake buildhistory/images/textbook/glibc/textbook-core-image/installed-package-names.txt
```
