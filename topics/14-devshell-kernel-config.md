# 14. devshell and Kernel Configuration

[Back to Learning Path](../README.md#learning-path)

Related Topics:

| Topic | What to look at in this chapter |
| --- | --- |
| `linux-textbook` kernel recipe | Kernel recipe used for build and debug examples. |
| `meta-textbook-core-bsp` | BSP layer that owns machine and kernel config fragments. |
| `meta-textbook-external` kernel externalsrc | Configuration differences when local kernel source is used. |

## When to Use

Use `devshell` when you want to enter the cross-build environment that BitBake prepared for a recipe. Use `menuconfig`, `savedefconfig`, and `diffconfig` when you want to adjust kernel configuration interactively and save the result as reproducible metadata.

## What This Chapter Covers

This chapter explains how to reproduce build issues inside the same environment used by BitBake tasks. It also shows how to turn interactive kernel `.config` changes into a `defconfig` or `.cfg` fragment that can be stored in a layer.

## What devshell Provides

`devshell` opens a shell for a specific recipe, usually in that recipe's source directory `${S}`. The shell includes the cross compiler, sysroot, pkg-config paths, and build variables that BitBake uses for the recipe tasks.

```sh
source envsetup.sh
bitbake hello-makefile-application -c devshell
```

Useful values inside devshell:

| Value | Description |
| --- | --- |
| `pwd` | Current source/build location opened by devshell. |
| `$S` | Recipe source directory. |
| `$B` | Recipe build directory. |
| `$WORKDIR` | Recipe work directory. |
| `$CC` | Cross compiler configured by BitBake. |
| `$CFLAGS`, `$LDFLAGS` | Target build flags. |
| `$PKG_CONFIG_PATH` | pkg-config search path for the recipe sysroot. |

## Debugging Builds in devshell

Makefile recipe:

```sh
bitbake hello-makefile-application -c devshell
```

Inside devshell:

```sh
oe_runmake -C ${S} O=${B}
```

CMake recipe:

```sh
bitbake hello-cmake-application -c devshell
```

Inside devshell:

```sh
cmake --build ${B}
```

You can also inspect or rerun generated task scripts:

```sh
ls ${WORKDIR}/temp/run.do_*
${WORKDIR}/temp/run.do_compile
${WORKDIR}/temp/run.do_install
```

| Caution | Description |
| --- | --- |
| devshell does not edit recipes automatically | It only reproduces the build environment. |
| Successful commands must be moved into metadata | Put the fix into `do_compile`, `do_install`, patches, or CMake options. |
| Task scripts may not exist yet | If sstate was reused, force a task first, for example `bitbake <recipe> -c compile -f`. |

## Using Kernel menuconfig

Run:

```sh
source envsetup.sh
bitbake linux-textbook -c kernel_configme -f
bitbake linux-textbook -c menuconfig
```

| Command | Description |
| --- | --- |
| `kernel_configme` | Regenerates `.config` from the current machine, recipe, and fragments. |
| `menuconfig` | Opens the ncurses kernel configuration UI. |
| save and exit | Updates the kernel `.config` in the build directory. |

The host may need ncurses development packages for `menuconfig`.

## Using savedefconfig

Run `savedefconfig` when you want a minimal `defconfig` instead of a full `.config`:

```sh
bitbake linux-textbook -c savedefconfig
```

Find the generated file:

```sh
find tmp/work -path '*linux-textbook*' -name defconfig
```

Example location in a layer:

```text
.
└── meta-textbook-core-bsp
    └── recipes-linux/linux/files/defconfig
```

Register it in the kernel recipe or `.bbappend`:

```bitbake
SRC_URI += "file://defconfig"
```

## Using diffconfig

Run `diffconfig` when you want only the changed `CONFIG_` entries from `menuconfig`:

```sh
bitbake linux-textbook -c kernel_configme -f
bitbake linux-textbook -c menuconfig
bitbake linux-textbook -c diffconfig
```

`diffconfig` writes a `fragment.cfg` file under `${WORKDIR}`.

Find it:

```sh
find tmp/work -path '*linux-textbook*' -name fragment.cfg
```

Example layer placement:

```text
.
└── meta-textbook-core-bsp
    └── recipes-linux/linux/files/my-feature.cfg
```

Register it:

```bitbake
SRC_URI += "file://my-feature.cfg"
FILESEXTRAPATHS:prepend := "${THISDIR}/files:"
```

Existing project fragments:

```text
.
├── meta-textbook-core-bsp
│   └── recipes-linux/linux/files
│       ├── qemuarm64.cfg
│       └── qemuarm64-ext.cfg
└── meta-textbook-selinux
    └── recipes-linux/linux/files
        └── selinux.cfg
```

`diffconfig` compares the saved `menuconfig` result against the previous config. If you run it without first running and saving `menuconfig`, it can fail because `.config.orig` does not exist.

## Key Takeaway

`devshell` is the way to work inside BitBake's recipe environment. `menuconfig` is the way to inspect and change kernel configuration interactively. The final result should always be saved as layer metadata, such as `defconfig` or a `.cfg` fragment, so the build stays reproducible.

## Verification Commands

```sh
bitbake linux-textbook -c kernel_configme -f
bitbake linux-textbook -c menuconfig
bitbake linux-textbook -c diffconfig
find tmp/work -path '*linux-textbook*' -name fragment.cfg

bitbake hello-makefile-application -c devshell
```
