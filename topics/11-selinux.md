# 11. SELinux Integration

[Back to Learning Path](../README.md#learning-path)

Related Commit:

- `9fd2c01 meta-textbook-selinux: introduce SELinux integration layer`
- `ae8b16c meta-textbook-selinux: update initial SELinux mode variable to DEFAULT_ENFORCING`

## When to Use

Add a SELinux layer when the image must enable SELinux together with the required kernel options, userspace tools, policy, and rootfs labeling.

## What This Chapter Covers

This chapter explains how SELinux is integrated as an image feature. It connects `meta-selinux`, feature configuration, kernel config fragments, and the `selinux-image` class.

## Concept

SELinux, or Security-Enhanced Linux, is a security framework that strengthens Linux access control. Normal Unix permissions use users, groups, and mode bits. SELinux also looks at security context labels and policy rules to decide whether a process can access a file, socket, device, or port.

For example, even if a service runs as root, SELinux can still block access to files or ports that the policy does not allow. This is why SELinux is used as a Mandatory Access Control (MAC) mechanism: it limits the damage if a process is compromised.

| Concept | Description |
| --- | --- |
| security context | Label attached to a process, file, socket, or device. |
| policy | Rule set that defines which domains may access which types. |
| enforcing mode | Blocks policy violations. |
| permissive mode | Logs policy violations without blocking them. |
| `meta-selinux` | Yocto layer that provides SELinux userspace, policy, and image support. |

In Yocto, SELinux is not enabled by installing one package. Kernel options, filesystem xattrs, userspace tools, the policy provider, and rootfs labeling must all line up.

## Required Additions

| Item | Description |
| --- | --- |
| `meta-selinux` layer | Provides SELinux userspace, policy, and image classes. |
| project SELinux layer | Keeps project-specific SELinux configuration isolated. |
| feature configuration | Groups distro, machine, and image feature changes. |
| kernel config fragment | Enables SELinux kernel options. |
| kernel recipe `.bbappend` | Adds the SELinux config fragment to the kernel recipe. |
| `IMAGE_CLASSES += "selinux-image"` | Enables rootfs labeling and SELinux image handling. |
| `packagegroup-core-selinux` | Installs SELinux userspace packages. |
| enforcing/permissive variable | Sets the initial SELinux mode. |

## Project Implementation

```text
.
└── layers
    └── meta-textbook
        ├── meta-textbook-selinux
        │   ├── conf/feature/textbook-selinux-feature.conf
        │   └── recipes-linux/linux
        │       ├── linux-textbook.bbappend
        │       └── files/selinux.cfg
        └── meta-textbook-core/conf/templates/default/bblayers.conf.sample
```

Feature configuration:

```bitbake
MACHINE_FEATURES:append = " selinux"
DISTRO_FEATURES:append = " acl xattr pam selinux"

SELINUX_MODE = "targeted"
PREFERRED_PROVIDER_virtual/refpolicy = "refpolicy-targeted"
IMAGE_CLASSES += "selinux-image"
IMAGE_INSTALL:append = " packagegroup-core-selinux"

DEFAULT_ENFORCING = "permissive"
```

Kernel configuration:

```bitbake
SRC_URI += "file://selinux.cfg"
FILESEXTRAPATHS:prepend := "${THISDIR}/files:"
```

## Key Takeaway

SELinux requires coordinated kernel, distro, policy, userspace, and rootfs changes. Keeping those changes in a dedicated layer makes the feature easier to enable, review, and remove.

## Verification Commands

```sh
bitbake-getvar DISTRO_FEATURES | grep selinux
grep -E '^(packagegroup-core-selinux|refpolicy-targeted|selinux-init)' buildhistory/images/textbook/glibc/textbook-core-image/installed-package-names.txt
```
