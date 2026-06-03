# 08. Makefile 기반 application과 library

[학습 순서로 돌아가기](../README.md#추천-학습-순서)

관련 commit:

- `2e832f9 application: Introduce meta-textbook-application layer and recipes`
- `32f8693 application: Add external layer and bbappends for local app/lib development`

## 필요한 상황

기존 Makefile 프로젝트를 Yocto image에 포함하고, application이 자체 library에 link되게 하고 싶다면 application layer와 recipe를 추가한다.

## 추가하면 되는 것

- application layer
- library recipe
- application recipe
- library의 header, shared object, pkg-config file install
- application recipe의 `DEPENDS`
- packagegroup `.bbappend`
- local 개발용 `externalsrc` `.bbappend`

## 이 프로젝트의 구현

파일:

- `meta-textbook-application/recipes-library/hello-makefile-library/hello-makefile-library.bb`
- `meta-textbook-application/recipes-application/hello-makefile-application/hello-makefile-application.bb`
- `meta-textbook-application/appends/packagegroups/packagegroup-textbook-core.bbappend`
- `meta-textbook-external/recipes-library/hello-makefile-library/hello-makefile-library.bbappend`
- `meta-textbook-external/recipes-application/hello-makefile-application/hello-makefile-application.bbappend`

library recipe:

```bitbake
do_compile() {
    oe_runmake -C ${S} O=${B} LDFLAGS="${LDFLAGS}"
}

do_install() {
    install -d ${D}/${libdir}
    install -m 0755 ${B}/libhello-makefile.so.1.0 ${D}/${libdir}
    ln -s libhello-makefile.so.1.0 ${D}/${libdir}/libhello-makefile.so.1
    ln -s libhello-makefile.so.1 ${D}/${libdir}/libhello-makefile.so

    install -d ${D}/${includedir}
    install -m 0644 ${S}/hello-makefile-library.h ${D}/${includedir}

    install -d ${D}${libdir}/pkgconfig
    install -m 0644 ${S}/hello-makefile-library.pc ${D}${libdir}/pkgconfig/
}
```

application recipe:

```bitbake
inherit pkgconfig
DEPENDS = "hello-makefile-library"

do_compile() {
    oe_runmake -C ${S} O=${B}
}

do_install() {
    install -d ${D}/${bindir}
    install -m 0755 ${B}/hello-makefile-application ${D}/${bindir}
}
```

## 핵심 메시지

Makefile 프로젝트는 Yocto가 build system을 자동으로 추측하지 않는다. 대신 `oe_runmake`와 `do_install`로 “어떻게 build하고 어떤 file을 rootfs에 넣을지”를 명확히 적는다.

## 확인 command

```sh
bitbake hello-makefile-library hello-makefile-application
grep hello-makefile build/buildhistory/images/textbook/glibc/textbook-core-image/installed-package-names.txt
```
