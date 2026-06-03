# 09. CMake 기반 application과 library

[학습 순서로 돌아가기](../README.md#추천-학습-순서)

관련 commit:

- `3f6f2d8 application: Add Yocto recipes for CMake-based library and application`
- `370321c external: Add bbappends for CMake projects and register to packagegroup`

## 필요한 상황

CMake 프로젝트를 Yocto package로 만들고, CMake 기반 application이 CMake 기반 library에 link되게 하고 싶다면 `cmake` class를 사용한다.

## 추가하면 되는 것

- CMake library recipe
- CMake application recipe
- library dependency
- 필요한 경우 `pkgconfig`
- packagegroup `.bbappend`
- local 개발용 `externalsrc` `.bbappend`

## 이 프로젝트의 구현

파일:

- `meta-textbook-application/recipes-library/hello-cmake-library/hello-cmake-library.bb`
- `meta-textbook-application/recipes-application/hello-cmake-application/hello-cmake-application.bb`
- `meta-textbook-external/recipes-library/hello-cmake-library/hello-cmake-library.bbappend`
- `meta-textbook-external/recipes-application/hello-cmake-application/hello-cmake-application.bbappend`

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

## 핵심 메시지

CMake 프로젝트는 `inherit cmake`만으로 configure, compile, install workflow 대부분을 Yocto class에 맡길 수 있다. 그래서 Makefile 예제와 비교하면 Yocto class의 가치가 잘 보인다.

## 확인 command

```sh
bitbake hello-cmake-library hello-cmake-application
grep hello-cmake build/buildhistory/images/textbook/glibc/textbook-core-image/installed-package-names.txt
```
