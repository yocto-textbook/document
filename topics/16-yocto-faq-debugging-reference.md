# 16. Yocto 사용 FAQ와 debugging reference

[학습 순서로 돌아가기](../README.md#추천-학습-순서)

## 이런 설명이 필요하다면

Yocto를 실제로 사용하다 보면 다음 질문이 바로 생긴다.

- 대표적인 기본 variable은 무엇인가
- error가 발생하면 어디부터 봐야 하는가
- buildhistory는 무엇이고 어디에 남는가
- layer는 어떻게 추가/제거하는가
- variable과 recipe의 최종 형태는 어떻게 확인하는가
- `.bbappend`가 적용됐는지, 어떤 recipe가 선택됐는지 어떻게 보는가

이 장은 후반부의 “실전 운영 레퍼런스”로 사용한다.

```mermaid
flowchart TB
    Error["BitBake ERROR"] --> Identify["recipe + task 확인"]
    Identify --> Logs["log.do_<task> 읽기"]
    Logs --> Vars["bitbake-getvar / bitbake -e<br/>최종 variable 확인"]
    Vars --> Appends["bitbake-layers show-appends<br/>layer 적용 확인"]
    Appends --> ReRun["bitbake <recipe> -c <task> -f"]
    ReRun --> Shell["필요하면 devshell에서 재현"]
    Shell --> Fix["recipe/layer 수정"]
    Fix --> History["buildhistory로 결과 변화 확인"]
```

## 대표 기본 variable

### Build/workspace variable

| variable | 의미 | 예시/비고 |
| --- | --- | --- |
| `TOPDIR` | 현재 build directory | 보통 `build` |
| `COREBASE` | Poky/OE-Core 기준 경로 | 이 프로젝트에서는 `poky` |
| `TMPDIR` | 대부분의 build 작업이 생기는 곳 | `${TOPDIR}/tmp` |
| `DL_DIR` | fetch한 source cache | `${TOPDIR}/downloads` 또는 mirror |
| `SSTATE_DIR` | shared state cache | `${TOPDIR}/sstate-cache` |
| `DEPLOY_DIR` | deploy output root | `${TMPDIR}/deploy` |
| `DEPLOY_DIR_IMAGE` | kernel/image output 위치 | `${TMPDIR}/deploy/images/${MACHINE}` |
| `BUILDHISTORY_DIR` | buildhistory 출력 위치 | 이 프로젝트는 `${TOPDIR}/buildhistory` |

확인:

```sh
bitbake-getvar TOPDIR
bitbake-getvar COREBASE
bitbake-getvar TMPDIR
bitbake-getvar DEPLOY_DIR_IMAGE
```

### Recipe 작업 경로 variable

| variable | 의미 |
| --- | --- |
| `WORKDIR` | recipe 하나의 workspace |
| `S` | source directory |
| `B` | build directory |
| `D` | install staging directory |
| `T` | task log/script가 있는 temp directory |

확인:

```sh
bitbake-getvar -r hello-makefile-application WORKDIR
bitbake-getvar -r hello-makefile-application S
bitbake-getvar -r hello-makefile-application B
bitbake-getvar -r hello-makefile-application D
```

### Recipe metadata variable

| variable | 의미 |
| --- | --- |
| `PN` | package/recipe base name |
| `PV` | version |
| `PR` | package revision |
| `PE` | epoch |
| `SUMMARY` | 짧은 설명 |
| `DESCRIPTION` | 긴 설명 |
| `LICENSE` | license |
| `LIC_FILES_CHKSUM` | license file checksum |
| `SRC_URI` | source, patch, local file 목록 |
| `SRCREV` | Git revision |
| `FILESPATH` | `file://` 검색 경로 |
| `FILESEXTRAPATHS` | layer에서 `FILESPATH` 확장 |

확인:

```sh
bitbake-getvar -r linux-textbook SRC_URI
bitbake-getvar -r linux-textbook PV
bitbake-getvar -r linux-textbook FILESPATH
```

### Dependency variable

| variable | 의미 |
| --- | --- |
| `DEPENDS` | build-time dependency |
| `RDEPENDS:${PN}` | runtime dependency |
| `RRECOMMENDS:${PN}` | 권장 runtime package |
| `PROVIDES` | build provider alias |
| `RPROVIDES:${PN}` | runtime provider alias |
| `PREFERRED_PROVIDER_virtual/kernel` | `virtual/kernel` provider 선택 |

예:

```bitbake
DEPENDS = "hello-cmake-library"
RDEPENDS:${PN} += "textbook-profile-service"
PREFERRED_PROVIDER_virtual/kernel = "linux-textbook"
```

### Machine/distro/image variable

| variable | 의미 |
| --- | --- |
| `MACHINE` | target machine |
| `MACHINE_FEATURES` | machine capability |
| `DISTRO` | distro policy |
| `DISTRO_FEATURES` | distro feature set |
| `IMAGE_INSTALL` | image에 설치할 package 목록 |
| `IMAGE_FEATURES` | image feature |
| `EXTRA_IMAGE_FEATURES` | 추가 image feature |
| `IMAGE_FSTYPES` | 생성할 image format |
| `PACKAGE_CLASSES` | rpm/ipk/deb package backend |

확인:

```sh
bitbake-getvar MACHINE
bitbake-getvar DISTRO
bitbake-getvar DISTRO_FEATURES
bitbake-getvar -r textbook-core-image IMAGE_INSTALL
bitbake-getvar -r textbook-core-image IMAGE_FSTYPES
```

### Layer variable

| variable | 의미 |
| --- | --- |
| `BBLAYERS` | 활성화된 layer 목록 |
| `BBPATH` | conf/class 검색 경로 |
| `BBFILES` | recipe 검색 pattern |
| `BBFILE_COLLECTIONS` | layer collection 이름 |
| `BBFILE_PRIORITY_*` | layer 우선순위 |
| `LAYERDEPENDS_*` | layer dependency |
| `LAYERSERIES_COMPAT_*` | 호환 Yocto release |

확인:

```sh
bitbake-getvar BBLAYERS
bitbake-layers show-layers
```

## variable 최종값 확인

단일 variable은 `bitbake-getvar`가 가장 읽기 쉽다.

```sh
bitbake-getvar MACHINE
bitbake-getvar DISTRO
bitbake-getvar -r linux-textbook SRC_URI
bitbake-getvar -r textbook-core-image IMAGE_INSTALL
```

전체 metadata dump가 필요하면 `bitbake -e`를 쓴다.

```sh
bitbake -e linux-textbook > /tmp/linux-textbook.env
grep '^SRC_URI=' /tmp/linux-textbook.env
grep '^do_compile' /tmp/linux-textbook.env
```

variable의 override까지 보고 싶을 때는 recipe를 지정해서 확인한다.

```sh
bitbake -e textbook-core-image | grep '^IMAGE_INSTALL='
bitbake -e packagegroup-textbook-core | grep '^RDEPENDS'
```

주의:

- `bitbake -e` 출력은 매우 크다.
- 단일 variable 확인은 `bitbake-getvar`, 전체 분석은 `bitbake -e`가 편하다.

## Recipe 최종 형태 확인

어떤 recipe가 있는지:

```sh
bitbake-layers show-recipes
bitbake-layers show-recipes hello-makefile-application
bitbake-layers show-recipes virtual/kernel
```

어떤 `.bbappend`가 적용되는지:

```sh
bitbake-layers show-appends
bitbake-layers show-appends | grep linux-textbook
bitbake-layers show-appends | grep packagegroup-textbook-core
```

같은 recipe가 여러 layer에 있을 때 어떤 것이 가려지는지:

```sh
bitbake-layers show-overlayed
```

layer 간 dependency:

```sh
bitbake-layers show-cross-depends
```

사용 가능한 machine:

```sh
bitbake-layers show-machines
```

사용 가능한 layer:

```sh
bitbake-layers show-layers
```

## Layer 추가와 제거

현재 활성 layer 확인:

```sh
bitbake-layers show-layers
```

layer 추가:

```sh
bitbake-layers add-layer ../layers/meta-textbook/meta-textbook-external
```

layer 제거:

```sh
bitbake-layers remove-layer ../layers/meta-textbook/meta-textbook-external
```

이 프로젝트의 helper:

```sh
source envsetup.sh
add_external_sources
remove_external_sources
```

직접 파일로 확인:

```sh
sed -n '1,160p' build/conf/bblayers.conf
```

주의:

- `bitbake-layers add-layer`는 `build/conf/bblayers.conf`의 `BBLAYERS`를 수정한다.
- `TEMPLATECONF`는 새 build directory를 만들 때 sample을 제공한다. 이미 만들어진 `build/conf/bblayers.conf`는 별도로 수정해야 한다.
- layer를 추가했는데 recipe가 안 보이면 `conf/layer.conf`, `BBFILES`, `LAYERSERIES_COMPAT`를 확인한다.

## error가 발생했을 때 기본 루틴

1. 실패한 task 이름을 확인한다.

예:

```text
ERROR: hello-makefile-application-1.0-r0 do_compile: ExecutionError
```

여기서 recipe는 `hello-makefile-application`, 실패 task는 `do_compile`이다.

2. log 파일을 연다.

```sh
bitbake-getvar -r hello-makefile-application T
ls $(bitbake-getvar -r hello-makefile-application --value T)
```

직접 찾기:

```sh
find build/tmp/work -path '*hello-makefile-application*' -path '*temp/log.do_compile*'
```

대표 로그:

```text
${WORKDIR}/temp/log.do_fetch
${WORKDIR}/temp/log.do_patch
${WORKDIR}/temp/log.do_configure
${WORKDIR}/temp/log.do_compile
${WORKDIR}/temp/log.do_install
${WORKDIR}/temp/log.do_package
${WORKDIR}/temp/log.do_rootfs
```

3. 실제 실행 script를 본다.

```sh
ls ${WORKDIR}/temp/run.do_compile*
```

`run.do_*` 파일은 BitBake가 실제로 실행한 shell script다. 로그만으로 부족하면 이 파일에서 어떤 command가 어떤 환경으로 실행됐는지 본다.

4. 해당 task만 강제로 다시 실행한다.

```sh
bitbake hello-makefile-application -c compile -f
```

5. devshell로 들어가 같은 환경에서 재현한다.

```sh
bitbake hello-makefile-application -c devshell
```

devshell 안:

```sh
echo $CC
echo $S
echo $B
oe_runmake -C ${S} O=${B}
```

## task별 자주 보는 원인

| 실패 task | 자주 보는 원인 | 확인할 것 |
| --- | --- | --- |
| `do_fetch` | URL error, branch error, 네트워크, checksum mismatch | `SRC_URI`, `SRCREV`, `DL_DIR`, mirror |
| `do_unpack` | archive 형식 문제, source directory 예상 불일치 | `S`, unpack 결과 |
| `do_patch` | patch context mismatch, patch 순서 문제 | `SRC_URI` patch 순서, `log.do_patch` |
| `do_configure` | dependency 누락, CMake/autotools 옵션 error | `DEPENDS`, `EXTRA_OECMAKE`, `PACKAGECONFIG` |
| `do_compile` | cross compile 옵션 누락, header/library 누락 | `CC`, `CFLAGS`, `DEPENDS`, sysroot |
| `do_install` | 잘못된 install path, `${D}` 미사용 | `install -d ${D}${bindir}` 형태 |
| `do_package` | `FILES` 누락, split package error | `FILES:${PN}`, `PACKAGES` |
| `do_package_qa` | rpath, already-stripped, dev-so, installed-vs-shipped | QA 메시지, `FILES`, build flags |
| `do_rootfs` | runtime dependency 미해결, package conflict | `RDEPENDS`, `IMAGE_INSTALL`, packagegroup |
| `do_image` | image size, filesystem tool 문제 | `IMAGE_ROOTFS_SIZE`, `IMAGE_FSTYPES` |

## clean 계열 command

recipe 작업 결과만 지우기:

```sh
bitbake hello-makefile-application -c clean
```

sstate까지 지우고 다시 build하게 하기:

```sh
bitbake hello-makefile-application -c cleansstate
```

download까지 지우기:

```sh
bitbake hello-makefile-application -c cleanall
```

주의:

- `cleanall`은 source download까지 지운다.
- fetch 문제가 아니라면 보통 `clean` 또는 `cleansstate`로 충분하다.

## buildhistory란 무엇인가

buildhistory는 build 결과의 변화 기록이다.

이 프로젝트 configuration:

```bitbake
INHERIT += "buildhistory"
BUILDHISTORY_COMMIT = "1"
BUILDHISTORY_COMMIT_AUTHOR = "JunKi Hong <dylanhong920509@gmail.com>"
BUILDHISTORY_DIR = "${TOPDIR}/buildhistory"
BUILDHISTORY_IMAGE_FILES = "/etc/passwd /etc/group"
```

남는 정보:

- image에 설치된 package 목록
- package version과 size
- image info
- dependency graph
- 특정 image file 내용
- metadata revision

이 프로젝트에서 확인한 파일:

```text
build/buildhistory/images/textbook/glibc/textbook-core-image/image-info.txt
build/buildhistory/images/textbook/glibc/textbook-core-image/installed-package-names.txt
build/buildhistory/images/textbook/glibc/textbook-core-image/files-in-image.txt
build/buildhistory/metadata-revs
```

확인:

```sh
git -C build/buildhistory log --oneline -n 10
sed -n '1,120p' build/buildhistory/images/textbook/glibc/textbook-core-image/image-info.txt
grep hello build/buildhistory/images/textbook/glibc/textbook-core-image/installed-package-names.txt
```

build 간 차이 확인:

```sh
git -C build/buildhistory diff HEAD~1 HEAD
buildhistory-diff build/buildhistory
```

핵심 메시지:

buildhistory는 “image에 무엇이 들어갔는지”와 “지난 build 이후 무엇이 바뀌었는지”를 추적하는 기록이다. 제품 개발에서는 image 크기 증가, package 추가/삭제, version 변경을 확인하는 데 중요하다.

## source, patch, file 검색 문제 확인

`file://`이 어디서 검색되는지:

```sh
bitbake-getvar -r linux-textbook FILESPATH
```

`FILESEXTRAPATHS`가 제대로 들어갔는지:

```sh
bitbake-getvar -r linux-textbook FILESEXTRAPATHS
```

source fetch 정보:

```sh
bitbake-getvar -r hello-cmake-application SRC_URI
bitbake-getvar -r hello-cmake-application SRCREV
```

## package 내용 확인

설치 staging directory 확인:

```sh
bitbake-getvar -r hello-makefile-application D
```

package split 결과는 buildhistory나 pkgdata로 확인한다.

```sh
oe-pkgdata-util list-pkgs | grep hello
oe-pkgdata-util list-pkg-files hello-makefile-application
oe-pkgdata-util find-path /usr/bin/hello-makefile-application
```

image에 실제로 들어갔는지:

```sh
grep hello-makefile-application build/buildhistory/images/textbook/glibc/textbook-core-image/installed-package-names.txt
```

## provider 문제 확인

`virtual/kernel`처럼 provider가 여러 개일 수 있는 항목은 provider 선택을 확인한다.

```sh
bitbake-getvar PREFERRED_PROVIDER_virtual/kernel
bitbake-layers show-recipes virtual/kernel
```

이 프로젝트:

```bitbake
PREFERRED_PROVIDER_virtual/kernel = "linux-textbook"
```

## override 확인

Yocto variable은 override로 machine, distro, package별 값을 바꿀 수 있다.

예:

```bitbake
RDEPENDS:${PN} += "textbook-profile-service"
IMAGE_ROOTFS_EXTRA_SPACE:append = " + 4096"
PACKAGECONFIG:append:pn-qemu-system-native = " sdl"
```

최종값 확인:

```sh
bitbake-getvar -r textbook-core-image IMAGE_ROOTFS_EXTRA_SPACE
bitbake-getvar -r packagegroup-textbook-core RDEPENDS
```

## dependency graph 확인

전체 image graph:

```sh
bitbake textbook-core-image -g
```

생성 파일:

```text
pn-buildlist
recipe-depends.dot
task-depends.dot
```

특정 recipe가 왜 build되는지 볼 때 유용하다.

## 실전 debugging 순서 요약

```text
1. ERROR에서 recipe와 task 이름을 확인한다.
2. ${WORKDIR}/temp/log.do_<task>를 읽는다.
3. bitbake-getvar -r <recipe>로 S/B/D/WORKDIR/SRC_URI/DEPENDS를 확인한다.
4. bitbake-layers show-appends로 .bbappend 적용 여부를 확인한다.
5. 필요한 task만 -f로 재실행한다.
6. devshell에서 같은 환경으로 수동 재현한다.
7. 수정이 맞으면 recipe/layer metadata에 반영한다.
8. buildhistory로 image/package 변화가 의도한 것인지 확인한다.
```

## 핵심 메시지

Yocto debugging의 핵심은 “최종 metadata를 확인하고, 실패한 task의 log/run script를 읽고, 필요하면 같은 환경에 들어가 재현하는 것”이다. 감으로 고치기보다 `bitbake-getvar`, `bitbake -e`, `bitbake-layers`, `buildhistory`를 사용해 현재 build 시스템이 무엇을 보고 있는지 먼저 확인한다.
