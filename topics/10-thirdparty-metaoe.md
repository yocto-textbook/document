# 10. Third-party package와 meta-oe 연동

[학습 순서로 돌아가기](../README.md#추천-학습-순서)

관련 commit:

- `023abb3 meta-thirdparty: Add new layer for useful CLI utilities`
- `bb34ff7 meta-textbook: integrate meta-oe and append core testing utilities`

## 필요한 상황

제품 image에 외부 open source utility를 추가하거나, meta-openembedded의 package를 활용하고 싶다면 별도 third-party layer를 둔다.

## 추가하면 되는 것

- third-party layer
- 직접 관리할 open source recipe
- meta-openembedded layer 등록
- packagegroup `.bbappend`
- 외부 package의 license checksum, source URI, build class

## 이 프로젝트의 구현

파일:

- `meta-thirdparty/conf/layer.conf`
- `meta-thirdparty/recipes-btop/btop/btop.bb`
- `meta-thirdparty/recipes-nano/nano/nano.bb`
- `meta-thirdparty/appends/packagegroups/packagegroup-textbook-core.bbappend`
- `meta-thirdparty/recipes-openembedded/packagegroups/packagegroup-textbook-core.bbappend`
- `meta-textbook-core/conf/templates/default/bblayers.conf.sample`

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

## 핵심 메시지

Third-party layer는 “직접 packaging하는 외부 tool”과 “이미 meta-oe에 있는 tool을 image에 포함하는 정책”을 분리해 보여주기 좋다.

## 확인 command

```sh
bitbake-layers show-layers | grep -E 'meta-thirdparty|meta-oe'
grep -E '^(btop|nano)$' build/buildhistory/images/textbook/glibc/textbook-core-image/installed-package-names.txt
```
