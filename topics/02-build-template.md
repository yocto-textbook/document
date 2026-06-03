# 02. build template과 default configuration

[학습 순서로 돌아가기](../README.md#추천-학습-순서)

관련 commit:

- `8198bff meta-textbook: introduce template configuration for textbook build environment`

## 필요한 상황

새 build directory를 만들 때 항상 같은 `local.conf`, `bblayers.conf`가 생성되게 하고 싶다면 Yocto의 `TEMPLATECONF`를 사용한다.

## 추가하면 되는 것

- `conf/templates/default/local.conf.sample`
- `conf/templates/default/bblayers.conf.sample`
- `conf/templates/default/conf-notes.txt`
- `envsetup.sh`에서 `TEMPLATECONF` export

## 이 프로젝트의 구현

파일:

- `meta-textbook-core/conf/templates/default/local.conf.sample`
- `meta-textbook-core/conf/templates/default/bblayers.conf.sample`
- `meta-textbook-core/conf/templates/default/conf-notes.txt`
- `envsetup.sh`

핵심 configuration:

```sh
export TEMPLATECONF=${WORKSPACE_BASE}/layers/meta-textbook/meta-textbook-core/conf/templates/default/
source poky/oe-init-build-env ${WORKSPACE_BASE}/${BUILD_DIR}
```

`local.conf.sample`에는 mirror, sstate, buildhistory configuration이 들어간다.

```bitbake
INHERIT += "own-mirrors"
SOURCE_MIRROR_URL = "file://${HOME}/yocto/source-mirrors"
SSTATE_MIRRORS = "file://.* file://${HOME}/yocto/sstate-cache/PATH"
INHERIT += "buildhistory"
BUILDHISTORY_COMMIT = "1"
```

## 핵심 메시지

Yocto에서 재현성은 source code만으로 끝나지 않는다. 어떤 layer를 켜고, 어떤 mirror와 cache를 쓰고, 어떤 buildhistory를 남기는지도 프로젝트의 일부다.

## 확인 command

```sh
source envsetup.sh
bitbake-getvar TEMPLATECONF
sed -n '1,120p' build/conf/bblayers.conf
```
