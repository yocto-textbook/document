# 02. build template과 default configuration

[Back to Learning Path](../README.md#learning-path)

Related Commit:

- `8198bff meta-textbook: introduce template configuration for textbook build environment`

## When to Use

새 build directory를 만들 때 항상 같은 `local.conf`, `bblayers.conf`가 생성되게 하고 싶다면 Yocto의 `TEMPLATECONF`를 사용한다.

## What This Chapter Covers

이 chapter는 새 build directory가 생성될 때 어떤 default configuration이 들어가는지 설명한다. `TEMPLATECONF`를 통해 layer 목록, mirror, sstate, buildhistory 같은 build policy를 project 기준으로 고정한다.

## Required Additions

| 항목 | 역할 |
| --- | --- |
| `conf/templates/default/local.conf.sample` | 새 build directory의 기본 local configuration |
| `conf/templates/default/bblayers.conf.sample` | 기본 enabled layer 목록 |
| `conf/templates/default/conf-notes.txt` | `oe-init-build-env` 진입 후 보여줄 안내 |
| `envsetup.sh`의 `TEMPLATECONF` | 사용할 template directory 지정 |

## Project Implementation

```text
.
└── layers
    └── meta-textbook
        ├── envsetup.sh
        └── meta-textbook-core
            └── conf/templates/default
                ├── local.conf.sample
                ├── bblayers.conf.sample
                └── conf-notes.txt
```

Key Configuration:

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

## Key Takeaway

Yocto에서 재현성은 source code만으로 끝나지 않는다. 어떤 layer를 켜고, 어떤 mirror와 cache를 쓰고, 어떤 buildhistory를 남기는지도 프로젝트의 일부다.

## Verification Commands

```sh
source envsetup.sh
echo "$TEMPLATECONF"
sed -n '1,120p' conf/bblayers.conf
```
