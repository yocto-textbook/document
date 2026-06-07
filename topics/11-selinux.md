# 11. SELinux 통합

[Back to Learning Path](../README.md#learning-path)

Related Commit:

- `9fd2c01 meta-textbook-selinux: introduce SELinux integration layer`
- `ae8b16c meta-textbook-selinux: update initial SELinux mode variable to DEFAULT_ENFORCING`

## When to Use

image에 SELinux를 켜고, kernel 옵션, user-space tool, policy, rootfs labeling을 함께 적용하고 싶다면 SELinux 전용 layer를 추가한다.

## What This Chapter Covers

이 chapter는 SELinux를 image feature로 통합할 때 필요한 kernel, distro, policy, rootfs 요소를 함께 설명한다. `meta-selinux`, feature conf, kernel config fragment, `selinux-image` class가 서로 어떻게 맞물리는지 다룬다.

## Concept

SELinux(Security-Enhanced Linux)는 Linux access control을 강화하는 security framework다. 일반적인 Unix permission은 user/group/mode bit를 기준으로 접근을 판단하지만, SELinux는 process와 file에 붙은 security context와 policy를 함께 보고 “이 process가 이 file/socket/device에 접근해도 되는가”를 판단한다.

예를 들어 service process가 root 권한으로 실행되더라도, SELinux policy가 허용하지 않으면 특정 file이나 network port에 접근하지 못하게 막을 수 있다. 그래서 침해가 발생했을 때 피해 범위를 줄이는 Mandatory Access Control(MAC) 용도로 사용한다.

| 개념 | Description |
| --- | --- |
| security context | process/file/socket 등에 붙는 SELinux label |
| policy | 어떤 domain이 어떤 type에 접근할 수 있는지 정의한 rule set |
| enforcing mode | policy 위반을 실제로 차단 |
| permissive mode | policy 위반을 차단하지 않고 log로만 남김 |
| `meta-selinux` | SELinux userspace, policy, image class를 제공하는 Yocto layer |

Yocto에서 SELinux는 단일 package 추가로 끝나지 않는다. kernel option, filesystem xattr, userspace tool, policy provider, rootfs labeling이 함께 맞아야 target에서 정상 동작한다.

## Required Additions

| 항목 | 역할 |
| --- | --- |
| `meta-selinux` layer | SELinux userspace, policy, image class 제공 |
| 프로젝트 SELinux layer | 프로젝트별 SELinux 설정 격리 |
| feature conf | distro/machine/image feature를 한곳에 묶음 |
| kernel config fragment | SELinux kernel option 활성화 |
| kernel recipe `.bbappend` | kernel recipe에 SELinux config fragment 추가 |
| `IMAGE_CLASSES += "selinux-image"` | rootfs labeling과 SELinux image 처리 |
| `packagegroup-core-selinux` | SELinux userspace package 포함 |
| enforcing/permissive variable | 초기 SELinux mode 정책 지정 |

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

feature configuration:

```bitbake
MACHINE_FEATURES:append = " selinux"
DISTRO_FEATURES:append = " acl xattr pam selinux"

SELINUX_MODE = "targeted"
PREFERRED_PROVIDER_virtual/refpolicy = "refpolicy-targeted"
IMAGE_CLASSES += "selinux-image"
IMAGE_INSTALL:append = " packagegroup-core-selinux"

DEFAULT_ENFORCING = "permissive"
```

kernel configuration:

```bitbake
SRC_URI += "file://selinux.cfg"
FILESEXTRAPATHS:prepend := "${THISDIR}/files:"
```

## Key Takeaway

SELinux는 package 하나를 설치한다고 끝나지 않는다. kernel feature, filesystem xattr, PAM/ACL, policy provider, rootfs labeling이 동시에 맞아야 한다. 그래서 별도 layer로 묶어 설명하는 것이 좋다.

## Verification Commands

```sh
bitbake-getvar DISTRO_FEATURES | grep selinux
grep -E '^(packagegroup-core-selinux|refpolicy-targeted|selinux-init)' buildhistory/images/textbook/glibc/textbook-core-image/installed-package-names.txt
```
