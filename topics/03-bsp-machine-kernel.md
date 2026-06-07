# 03. BSP, Machine, and Kernel Provider

[Back to Learning Path](../README.md#learning-path)

Related Commit:

- `9b3c03e meta-textbook-core-bsp: introduce BSP layer and linux-textbook kernel`

## When to Use

Use a BSP layer when the project needs to define the target machine, kernel
provider, QEMU model, and kernel configuration.

## What This Chapter Covers

This chapter explains how a Yocto machine configuration connects hardware
selection to the kernel recipe. In this project, `textbook.conf` selects
`linux-textbook` as the provider for `virtual/kernel`.

## Concept

`MACHINE` answers the question: "What target am I building for?" It controls
tuning, kernel provider, QEMU settings, machine features, and machine-specific
runtime dependencies.

`virtual/kernel` is an abstract dependency. Recipes and images can depend on
`virtual/kernel` without knowing which concrete kernel recipe is used. The
machine configuration selects that concrete provider.

| concept | role |
| --- | --- |
| `MACHINE` | target hardware or QEMU machine selection |
| `PREFERRED_PROVIDER_virtual/kernel` | concrete kernel recipe for the target |
| `KERNEL_IMAGETYPE` | kernel image format such as `Image` |
| kernel config fragment | target-specific kernel configuration |
| BSP layer | machine and kernel metadata grouped by hardware ownership |

## Project Implementation

```text
.
└── layers
    └── meta-textbook
        └── meta-textbook-core-bsp
            ├── conf/machine/textbook.conf
            └── recipes-linux
                ├── linux/linux-textbook.bb
                └── linux/files/qemuarm64.cfg
```

Machine configuration:

```bitbake
KERNEL_IMAGETYPE = "Image"
PREFERRED_PROVIDER_virtual/kernel = "linux-textbook"
MACHINE_FEATURES:append = " qemu-usermode"
```

Kernel recipe:

```bitbake
require recipes-kernel/linux/linux-yocto.inc
SRC_URI += "file://qemuarm64.cfg"
```

## Key Takeaway

The BSP layer connects a target machine to the kernel metadata needed to boot
that target. Machine configuration is not only naming; it selects the concrete
kernel, image type, features, and runtime requirements.

## Verification Commands

```sh
source envsetup.sh
bitbake-getvar MACHINE
bitbake-getvar PREFERRED_PROVIDER_virtual/kernel
bitbake-layers show-layers | grep textbook-core-bsp
```
