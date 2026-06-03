# 05. Distro와 systemd 전환

[학습 순서로 돌아가기](../README.md#추천-학습-순서)

관련 commit:

- `e99d22f meta-textbook-core: introduce textbook-core-distro configuration`
- `17998dd distro: introduction of systemd-enabled distribution and ecosystem updates`

## 필요한 상황

프로젝트만의 배포판 이름, 버전, init system, feature 정책을 고정하고 싶다면 distro configuration을 추가한다.

## 추가하면 되는 것

- `conf/distro/<name>.conf`
- `envsetup.sh`의 `DISTRO` 값
- systemd를 쓸 경우 `DISTRO_FEATURES`, virtual runtime provider configuration
- systemd가 요구하는 `usrmerge`

## 이 프로젝트의 구현

파일:

- `meta-textbook-core/conf/distro/textbook-core-distro.conf`
- `meta-textbook-core/conf/distro/textbook-systemd-distro.conf`
- `envsetup.sh`

핵심 configuration:

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

## 핵심 메시지

Machine이 하드웨어라면 Distro는 제품 정책이다. 같은 machine이라도 init system, libc 정책, 보안 feature, package 관리 방식은 distro에서 결정된다.

## 확인 command

```sh
source envsetup.sh
bitbake-getvar DISTRO
bitbake-getvar DISTRO_FEATURES
bitbake-getvar VIRTUAL-RUNTIME_init_manager
```

