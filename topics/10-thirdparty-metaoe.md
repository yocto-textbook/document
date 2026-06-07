# 10. Third-party package와 meta-oe 연동

[Back to Learning Path](../README.md#learning-path)

Related Commit:

- `023abb3 meta-thirdparty: Add new layer for useful CLI utilities`
- `bb34ff7 meta-textbook: integrate meta-oe and append core testing utilities`

## When to Use

제품 image에 외부 open source utility를 추가하거나, meta-openembedded의 package를 활용하고 싶다면 별도 third-party layer를 둔다.

## What This Chapter Covers

이 chapter는 외부 utility를 project image에 넣는 두 가지 방식을 설명한다. 직접 recipe를 작성해 source/license/build class를 관리하는 경우와, meta-openembedded에 이미 있는 package를 layer와 packagegroup으로 끌어오는 경우를 구분한다.

## Concept

`meta-openembedded`는 OpenEmbedded community가 관리하는 Yocto/OE용 layer collection이다. OE-Core에 포함되지 않은 많은 application, library, network tool, Python package, system utility recipe가 이 layer collection 안에 들어 있다. 보통 `meta-oe`, `meta-networking`, `meta-python`, `meta-filesystems`처럼 목적별 layer로 나뉜다.

외부 package를 image에 넣는 방법은 크게 두 가지다.

| 방식 | 언제 사용하나 | 예 |
| --- | --- | --- |
| 직접 recipe 작성 | 원하는 package가 upstream layer에 없거나 project에서 version/source/patch를 직접 통제해야 할 때 | `btop`, `nano` recipe |
| 기존 layer 활용 | 이미 검증된 recipe가 `meta-openembedded` 같은 upstream layer에 있을 때 | `meta-oe`에 있는 utility package |

필요한 recipe나 layer를 찾을 때는 OpenEmbedded Layer Index를 먼저 확인한다. 이 프로젝트는 Yocto `scarthgap` 기준이므로 branch도 `scarthgap`으로 맞춰 검색한다.

| 확인할 것 | 어디서 확인하나 |
| --- | --- |
| package recipe가 어느 layer에 있는지 | `https://layers.openembedded.org/layerindex/branch/scarthgap/recipes/` |
| layer가 어떤 repository와 branch를 쓰는지 | `https://layers.openembedded.org/layerindex/branch/scarthgap/layers/` |
| 특정 layer의 dependency | Layer Index의 layer 상세 페이지 |
| workspace에 받을 source | 해당 layer의 Git repository URL과 branch |

필요한 layer를 찾은 뒤에는 repo manifest나 `bblayers.conf.sample`에 layer를 추가하고, packagegroup `.bbappend`에서 필요한 package를 image에 포함한다. layer를 추가하는 것과 package를 image에 설치하는 것은 별개다. layer를 등록해야 recipe가 보이고, packagegroup에 넣어야 rootfs에 설치된다.

## Required Additions

| 항목 | 역할 |
| --- | --- |
| third-party layer | 외부 utility packaging 정책을 core layer와 분리 |
| 직접 관리할 open source recipe | btop, nano처럼 직접 recipe를 작성하는 package |
| meta-openembedded layer 등록 | 이미 upstream layer에 있는 package 활용 |
| packagegroup `.bbappend` | image에 third-party utility 포함 |
| license checksum, source URI, build class | 외부 source의 재현성과 license 검증 |

## Project Implementation

```text
.
└── layers
    └── meta-textbook
        ├── meta-thirdparty
        │   ├── conf/layer.conf
        │   ├── recipes-btop/btop/btop.bb
        │   ├── recipes-nano/nano/nano.bb
        │   ├── appends/packagegroups/packagegroup-textbook-core.bbappend
        │   └── recipes-openembedded/packagegroups/packagegroup-textbook-core.bbappend
        └── meta-textbook-core/conf/templates/default/bblayers.conf.sample
```

| 구분 | 예 | 설명 |
| --- | --- | --- |
| 직접 recipe | `btop`, `nano` | source URI, license, build class를 recipe에서 직접 관리 |
| upstream layer 활용 | meta-oe package | `bblayers.conf.sample`에 layer를 등록하고 packagegroup에서 선택 |

직접 recipe:

```bitbake
# btop
SRC_URI = "git://github.com/aristocratos/btop.git;protocol=https;branch=main"
SRCREV = "v1.4.0"
do_compile() {
    oe_runmake
}
do_install() {
    install -d ${D}${bindir}
    install -m 0755 ${S}/bin/btop ${D}${bindir}/btop
}
```

```bitbake
# nano
SRC_URI = "https://ftp.gnu.org/gnu/nano/nano-9.0.tar.xz"
DEPENDS = "ncurses"
inherit gettext pkgconfig autotools
```

packagegroup 확장:

```bitbake
RDEPENDS:${PN} += "\
    btop \
    nano \
"
```

## Key Takeaway

Third-party layer는 “직접 packaging하는 외부 tool”과 “이미 meta-oe에 있는 tool을 image에 포함하는 정책”을 분리해 보여주기 좋다.

## Verification Commands

```sh
bitbake-layers show-layers | grep -E 'meta-thirdparty|meta-oe'
grep -E '^(btop|nano)$' buildhistory/images/textbook/glibc/textbook-core-image/installed-package-names.txt
```
