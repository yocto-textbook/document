# 10. Third-Party Packages and meta-oe

[Back to Learning Path](../README.md#learning-path)

Related Commit:

- `023abb3 meta-thirdparty: Add new layer for useful CLI utilities`
- `bb34ff7 meta-textbook: integrate meta-oe and append core testing utilities`

## When to Use

Add a third-party layer when the image needs external open source utilities or packages from `meta-openembedded`.

## What This Chapter Covers

This chapter explains two ways to add external utilities to the project image. You can write and maintain a recipe yourself, or you can use a recipe that already exists in an upstream layer such as `meta-openembedded`.

## Concept

`meta-openembedded` is a collection of Yocto/OpenEmbedded layers maintained by the OpenEmbedded community. It contains many recipes that are not part of OE-Core: applications, libraries, networking tools, Python packages, filesystem utilities, and more. It is split into layers such as `meta-oe`, `meta-networking`, `meta-python`, and `meta-filesystems`.

There are two common ways to bring third-party packages into an image:

| Method | Use when | Example |
| --- | --- | --- |
| Write your own recipe | The package is not available in an upstream layer, or the project must control source, version, patches, and packaging details directly. | `btop`, `nano` recipes |
| Use an existing layer | A maintained recipe already exists in an upstream layer. | utilities from `meta-oe` |

Use the OpenEmbedded Layer Index to find recipes and layer dependencies. This project is based on Yocto `scarthgap`, so search that branch:

| What to check | Where to check |
| --- | --- |
| Which layer provides a recipe | `https://layers.openembedded.org/layerindex/branch/scarthgap/recipes/` |
| Which repository and branch a layer uses | `https://layers.openembedded.org/layerindex/branch/scarthgap/layers/` |
| Layer dependencies | The layer detail page in Layer Index. |
| Source to add to the workspace | The layer Git repository URL and branch. |

Adding a layer and installing a package are separate operations. The layer must be registered before BitBake can see its recipes. The package must also be added to a packagegroup or `IMAGE_INSTALL` before it appears in the rootfs.

## Required Additions

| Item | Description |
| --- | --- |
| third-party layer | Keeps external utility packaging policy separate from the core layer. |
| directly maintained open source recipes | Recipes such as `btop` and `nano` that this project owns. |
| `meta-openembedded` layer registration | Makes upstream recipes visible to BitBake. |
| packagegroup `.bbappend` | Selects which third-party utilities are installed in the image. |
| license checksum, source URI, build class | Keeps external source reproducible and license-checked. |

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

| Category | Example | Description |
| --- | --- | --- |
| Direct recipe | `btop`, `nano` | Source URI, license, and build class are managed in this project. |
| Upstream layer package | `meta-oe` package | The layer is registered in `bblayers.conf.sample`, then selected through a packagegroup. |

Direct recipe examples:

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

Packagegroup extension:

```bitbake
RDEPENDS:${PN} += "\
    btop \
    nano \
"
```

## Key Takeaway

A third-party layer is useful for separating external package policy from the product core. It also shows the difference between owning a recipe directly and selecting an already-maintained recipe from `meta-openembedded`.

## Verification Commands

```sh
bitbake-layers show-layers | grep -E 'meta-thirdparty|meta-oe'
grep -E '^(btop|nano)$' buildhistory/images/textbook/glibc/textbook-core-image/installed-package-names.txt
```
