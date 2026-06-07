# 05. Distro와 systemd 전환

[Back to Learning Path](../README.md#learning-path)

Related Commit:

- `e99d22f meta-textbook-core: introduce textbook-core-distro configuration`
- `17998dd distro: introduction of systemd-enabled distribution and ecosystem updates`

## When to Use

프로젝트만의 distro 이름, 버전, init system, feature 정책을 고정하고 싶다면 distro configuration을 추가한다.

## What This Chapter Covers

이 chapter는 hardware 선택과 별개로 product policy를 고정하는 distro configuration을 설명한다. systemd 전환 과정에서 `DISTRO_FEATURES`, virtual runtime provider, `usrmerge`가 어떤 의미를 갖는지 다룬다.

## Concept

Yocto에서 `machine`은 target hardware를 고르는 configuration이고, `distro`는 제품 운영 정책을 고르는 configuration이다. 같은 `textbook` machine을 쓰더라도 어떤 init system을 쓸지, 어떤 libc/toolchain 정책을 둘지, 어떤 feature를 기본으로 켤지는 distro에서 결정한다.

`systemd`는 Linux system과 service를 시작하고 관리하는 init/service manager다. 전통적인 SysV init처럼 boot script를 순서대로 실행하는 방식보다, unit file을 기준으로 service dependency, parallel startup, socket activation, logging integration을 관리한다. Yocto에서는 systemd를 쓰려면 package 하나만 추가하는 것이 아니라 distro feature와 virtual runtime provider를 함께 바꿔야 한다.

| 개념 | 역할 | 이 chapter에서 보는 지점 |
| --- | --- | --- |
| `distro` | 제품 전체의 runtime/build policy 선택 | `DISTRO`, `DISTRO_FEATURES`, version, codename |
| `DISTRO_FEATURES` | distro가 지원할 capability 목록 | `systemd`, `usrmerge` 추가, `sysvinit` 제거 |
| `VIRTUAL-RUNTIME_*` | virtual runtime dependency의 실제 provider 선택 | init manager를 `systemd`로 전환 |
| `systemd` | boot와 service lifecycle을 관리하는 init/service manager | service 기반 rootfs 운영 정책 |

정리하면 machine은 “어떤 board/QEMU target인가”를 말하고, distro는 “그 target에서 어떤 Linux 제품 정책으로 동작할 것인가”를 말한다.

## Required Additions

| 항목 | 역할 |
| --- | --- |
| `conf/distro/<name>.conf` | distro 이름, 버전, feature 정책 정의 |
| `envsetup.sh`의 `DISTRO` | 기본 distro 선택 |
| `DISTRO_FEATURES` | systemd, usrmerge 같은 distro capability 지정 |
| virtual runtime provider | init manager와 initscripts provider 전환 |
| `usrmerge` | systemd 사용 시 필요한 filesystem layout feature |

## Project Implementation

```text
.
└── layers
    └── meta-textbook
        ├── envsetup.sh
        └── meta-textbook-core
            └── conf/distro
                ├── textbook-core-distro.conf
                └── textbook-systemd-distro.conf
```

Key Configuration:

```bitbake
DISTRO ?= "textbook-systemd-distro"
DISTRO_NAME ?= "Textbook Systemd Distro (Textbook Systemd Distro)"
DISTRO_VERSION ?= "1.0.0"
DISTRO_CODENAME = "scarthgap"

DISTRO_FEATURES:append = " systemd usrmerge"
DISTRO_FEATURES:remove = " sysvinit"
VIRTUAL-RUNTIME_init_manager = "systemd"
VIRTUAL-RUNTIME_initscripts = "systemd-compat-units"
DISTRO_FEATURES_BACKFILL_CONSIDERED = "sysvinit"
```

## Key Takeaway

Machine이 하드웨어라면 Distro는 제품 정책이다. 같은 machine이라도 init system, libc 정책, 보안 feature, package 관리 방식은 distro에서 결정된다.

## Verification Commands

```sh
source envsetup.sh
bitbake-getvar DISTRO
bitbake-getvar DISTRO_FEATURES
bitbake-getvar VIRTUAL-RUNTIME_init_manager
```
