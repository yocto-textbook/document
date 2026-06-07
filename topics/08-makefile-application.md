# 08. Makefile 기반 application과 library

[Back to Learning Path](../README.md#learning-path)

Related Commit:

- `2e832f9 application: Introduce meta-textbook-application layer and recipes`
- `32f8693 application: Add external layer and bbappends for local app/lib development`

## When to Use

기존 Makefile 프로젝트를 Yocto image에 포함하고, application이 자체 library에 link되게 하고 싶다면 application layer와 recipe를 추가한다.

## What This Chapter Covers

이 chapter는 Makefile 기반 source를 Yocto recipe로 packaging하는 방법을 설명한다. `oe_runmake`로 compile을 실행하고, `do_install`에서 binary, shared library, header, pkg-config file을 target/sysroot에 맞게 배치한다.

## Required Additions

| 항목 | 역할 |
| --- | --- |
| application layer | application/library recipe 소유 layer |
| library recipe | shared library build 및 install |
| application recipe | executable build 및 install |
| header, shared object, pkg-config install | 다른 recipe가 library를 사용할 수 있게 sysroot 구성 |
| application `DEPENDS` | build-time library dependency 선언 |
| packagegroup `.bbappend` | image에 application/library package 포함 |
| `externalsrc` `.bbappend` | local source 기반 개발 build |

## Project Implementation

```text
.
├── meta-textbook-application
│   ├── recipes-library
│   │   └── hello-makefile-library
│   │       └── hello-makefile-library.bb
│   ├── recipes-application
│   │   └── hello-makefile-application
│   │       └── hello-makefile-application.bb
│   └── appends/packagegroups/packagegroup-textbook-core.bbappend
└── meta-textbook-external
    ├── recipes-library/hello-makefile-library/hello-makefile-library.bbappend
    └── recipes-application/hello-makefile-application/hello-makefile-application.bbappend
```

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

## Key Takeaway

Makefile 프로젝트는 Yocto가 build system을 자동으로 추측하지 않는다. 대신 `oe_runmake`와 `do_install`로 “어떻게 build하고 어떤 file을 rootfs에 넣을지”를 명확히 적는다.

## Verification Commands

```sh
bitbake hello-makefile-library hello-makefile-application
grep hello-makefile buildhistory/images/textbook/glibc/textbook-core-image/installed-package-names.txt
```
