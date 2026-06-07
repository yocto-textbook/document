# 14. devshell과 kernel configuration

[Back to Learning Path](../README.md#learning-path)

Related Topics:

| topic | 이 장에서 보는 지점 |
| --- | --- |
| `linux-textbook` kernel recipe | kernel build/debug 대상 recipe |
| `meta-textbook-core-bsp` | machine과 kernel config fragment가 있는 BSP layer |
| `meta-textbook-external` kernel externalsrc | local kernel source를 사용할 때 configuration 차이 |

## When to Use

BitBake가 실제로 구성한 cross build environment 안에 들어가서 직접 build command를 실행하거나, kernel configuration을 `menuconfig`로 바꾼 뒤 fragment/defconfig로 저장하고 싶다면 `devshell`, `menuconfig`, `savedefconfig`, `diffconfig`를 사용한다.

## What This Chapter Covers

이 chapter는 BitBake task가 실행되는 환경을 직접 열어 build 문제를 재현하는 방법을 설명한다. 또한 kernel `.config`를 interactive하게 수정한 뒤, 그 결과를 `defconfig`나 `.cfg` fragment로 layer에 남겨 reproducible configuration으로 만드는 흐름을 다룬다.

## devshell이 해주는 일

`devshell`은 지정한 recipe의 source directory인 `${S}` 안에서 shell을 열어준다. 이때 BitBake가 recipe task를 실행할 때 쓰는 cross compiler, sysroot, pkg-config, configure 관련 환경이 잡혀 있다.

```sh
source envsetup.sh
bitbake hello-makefile-application -c devshell
```

devshell 안에서 확인할 값:

| 확인값 | Description |
| --- | --- |
| `pwd` | devshell이 열린 현재 source/build 위치 |
| `$S` | recipe source directory |
| `$B` | recipe build directory |
| `$WORKDIR` | recipe work directory |
| `$CC` | BitBake가 구성한 cross compiler |
| `$CFLAGS`, `$LDFLAGS` | target build flag |
| `$PKG_CONFIG_PATH` | recipe sysroot의 pkg-config 검색 경로 |

## devshell에서 build debugging하기

Makefile 기반 recipe:

```sh
bitbake hello-makefile-application -c devshell
```

devshell 안:

```sh
oe_runmake -C ${S} O=${B}
```

CMake 기반 recipe:

```sh
bitbake hello-cmake-application -c devshell
```

devshell 안:

```sh
cmake --build ${B}
```

task script를 직접 다시 실행할 수도 있다.

```sh
ls ${WORKDIR}/temp/run.do_*
${WORKDIR}/temp/run.do_compile
${WORKDIR}/temp/run.do_install
```

주의할 점:

| 주의점 | Description |
| --- | --- |
| recipe 자동 수정 아님 | devshell은 build environment만 재현한다. |
| 성공한 command는 recipe에 반영 | `do_compile`, `do_install`, patch, CMake option 등으로 옮겨야 재현 가능하다. |
| task script가 없을 수 있음 | sstate 재사용 때문에 생성되지 않았을 수 있으므로 `bitbake <recipe> -c compile -f`처럼 task를 먼저 실행한다. |

## kernel menuconfig 사용

kernel configuration을 대화형으로 바꾸고 싶다면:

```sh
source envsetup.sh
bitbake linux-textbook -c kernel_configme -f
bitbake linux-textbook -c menuconfig
```

동작:

| command | 동작 |
| --- | --- |
| `kernel_configme` | 현재 machine, recipe, config fragment를 기준으로 `.config`를 다시 생성 |
| `menuconfig` | ncurses 기반 kernel configuration UI 실행 |
| 저장 후 종료 | build directory 안의 kernel `.config` 갱신 |

호스트에 ncurses 개발 package가 필요할 수 있다.

## savedefconfig 사용

전체 `.config`가 아니라 최소 configuration인 `defconfig`를 얻고 싶다면:

```sh
bitbake linux-textbook -c savedefconfig
```

결과는 보통 kernel build directory 쪽에 `defconfig`로 생성된다. 위치는 build 상태에 따라 달라질 수 있으므로 다음처럼 찾는다.

```sh
find tmp/work -path '*linux-textbook*' -name defconfig
```

이 파일을 recipe space에 넣고 싶다면 예를 들어:

```text
.
└── meta-textbook-core-bsp
    └── recipes-linux/linux/files/defconfig
```

그리고 kernel recipe나 `.bbappend`에 추가한다.

```bitbake
SRC_URI += "file://defconfig"
```

## diffconfig 사용

`menuconfig`에서 바꾼 configuration만 fragment로 뽑고 싶다면:

```sh
bitbake linux-textbook -c kernel_configme -f
bitbake linux-textbook -c menuconfig
bitbake linux-textbook -c diffconfig
```

`diffconfig`는 변경된 `CONFIG_` 항목만 담은 `fragment.cfg`를 `${WORKDIR}`에 만든다.

찾는 방법:

```sh
find tmp/work -path '*linux-textbook*' -name fragment.cfg
```

프로젝트 layer에 반영하는 예:

```text
.
└── meta-textbook-core-bsp
    └── recipes-linux/linux/files/my-feature.cfg
```

recipe에 등록:

```bitbake
SRC_URI += "file://my-feature.cfg"
FILESEXTRAPATHS:prepend := "${THISDIR}/files:"
```

이 프로젝트에는 이미 다음 config fragment가 있다.

```text
.
├── meta-textbook-core-bsp
│   └── recipes-linux/linux/files
│       ├── qemuarm64.cfg
│       └── qemuarm64-ext.cfg
└── meta-textbook-selinux
    └── recipes-linux/linux/files
        └── selinux.cfg
```

## Key Takeaway

`devshell`은 “BitBake가 실행하는 환경 안에 직접 들어가는 방법”이고, `menuconfig`는 “kernel configuration을 눈으로 보며 바꾸는 방법”이다. 하지만 최종 결과는 반드시 recipe layer에 `defconfig`나 `.cfg` fragment로 저장해야 재현 가능해진다.

## Verification Commands

```sh
bitbake linux-textbook -c kernel_configme -f
bitbake linux-textbook -c menuconfig
bitbake linux-textbook -c diffconfig
find tmp/work -path '*linux-textbook*' -name fragment.cfg

bitbake hello-makefile-application -c devshell
```

`diffconfig`는 `menuconfig`에서 저장한 변경분을 비교하므로, `menuconfig`를 실행하지 않은 상태에서 바로 실행하면 `.config.orig`가 없어 실패할 수 있다.
