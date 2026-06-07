# 08. Makefile Application and Library

[Back to Learning Path](../README.md#learning-path)

Related Commit:

- `2e832f9 application: Introduce meta-textbook-application layer and recipes`
- `32f8693 application: Add external layer and bbappends for local app/lib development`

## When to Use

Add application and library recipes when an existing Makefile project must be included in the Yocto image and the application must link against its own library.

## What This Chapter Covers

This chapter shows how to package Makefile-based source with Yocto recipes. The library recipe installs a shared library, header, and pkg-config file. The application recipe declares a build-time dependency on that library and installs the executable into the target rootfs.

## Required Additions

| Item | Description |
| --- | --- |
| application layer | Owns application and library recipes. |
| library recipe | Builds and installs the shared library. |
| application recipe | Builds and installs the executable. |
| header, `.so`, pkg-config file | Allows other recipes to use the library through the recipe sysroot. |
| application `DEPENDS` | Declares build-time dependency on the library recipe. |
| packagegroup `.bbappend` | Includes the application/library packages in the image. |
| `externalsrc` `.bbappend` | Allows local source builds during development. |

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

Library recipe:

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

Application recipe:

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

`DEPENDS` is important because it makes the library headers, shared object, and pkg-config metadata available in the application recipe sysroot before `do_configure` and `do_compile` run.

## Key Takeaway

Yocto does not infer arbitrary Makefile behavior. For a Makefile project, the recipe should clearly state how to build the source and which files belong in the target rootfs. `oe_runmake` handles the Makefile invocation, and `do_install` defines the final packaging input.

## Verification Commands

```sh
bitbake hello-makefile-library hello-makefile-application
grep hello-makefile buildhistory/images/textbook/glibc/textbook-core-image/installed-package-names.txt
```
