# 07. kernel module을 image에 포함하기

[학습 순서로 돌아가기](../README.md#추천-학습-순서)

관련 commit:

- `d8df46c textbook: Add hello-module recipe and include it as an essential dependency`
- `2bfa3fc external: Append hello-module to support local external source tree`

## 필요한 상황

제품 image에 kernel module을 포함하고, 부팅 시 자동 로드되게 하고 싶다면 kernel module recipe를 추가한다.

## 추가하면 되는 것

- kernel module source repo
- `inherit module`을 사용하는 recipe
- `RPROVIDES:${PN}`로 `kernel-module-*` 이름 제공
- `KERNEL_MODULE_AUTOLOAD`
- machine 또는 packagegroup에서 runtime dependency 추가
- local module 개발이 필요하면 `externalsrc` `.bbappend`

## 이 프로젝트의 구현

파일:

- `meta-textbook-core-bsp/recipes-linux/hello-module/hello-module.bb`
- `meta-textbook-core-bsp/conf/machine/textbook.conf`
- `meta-textbook-external/recipes-linux/hello-module/hello-module.bbappend`
- `external/hello-module`

recipe:

```bitbake
inherit module

SRC_URI = "git://github.com/yocto-textbook/hello-module.git;protocol=https;branch=main"
SRCREV = "${AUTOREV}"
S = "${WORKDIR}/git"

RPROVIDES:${PN} += "kernel-module-hello-module"
KERNEL_MODULE_AUTOLOAD += "hello-module"
ALLOW_EMPTY:${PN} = "1"
```

machine dependency:

```bitbake
MACHINE_ESSENTIAL_EXTRA_RDEPENDS += "\
    kernel-module-hello-module \
"
```

## 핵심 메시지

kernel module은 application package와 다르게 kernel version에 묶인다. Yocto의 `module` class를 쓰면 현재 build 중인 kernel과 맞는 module package를 만들 수 있다.

## 확인 command

```sh
bitbake hello-module
grep kernel-module-hello build/buildhistory/images/textbook/glibc/textbook-core-image/installed-package-names.txt
```
