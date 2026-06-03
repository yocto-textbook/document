# 11. SELinux 통합

[학습 순서로 돌아가기](../README.md#추천-학습-순서)

관련 commit:

- `9fd2c01 meta-textbook-selinux: introduce SELinux integration layer`
- `ae8b16c meta-textbook-selinux: update initial SELinux mode variable to DEFAULT_ENFORCING`

## 필요한 상황

image에 SELinux를 켜고, kernel 옵션, user-space tool, policy, rootfs labeling을 함께 적용하고 싶다면 SELinux 전용 layer를 추가한다.

## 추가하면 되는 것

- `meta-selinux` layer
- 프로젝트 SELinux layer
- feature conf
- kernel config fragment
- kernel recipe `.bbappend`
- `IMAGE_CLASSES += "selinux-image"`
- `packagegroup-core-selinux`
- 초기 enforcing/permissive 정책 variable

## 이 프로젝트의 구현

파일:

- `meta-textbook-selinux/conf/feature/textbook-selinux-feature.conf`
- `meta-textbook-selinux/recipes-linux/linux/files/selinux.cfg`
- `meta-textbook-selinux/recipes-linux/linux/linux-textbook.bbappend`
- `meta-textbook-core/conf/templates/default/bblayers.conf.sample`

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

## 핵심 메시지

SELinux는 package 하나를 설치한다고 끝나지 않는다. kernel feature, filesystem xattr, PAM/ACL, policy provider, rootfs labeling이 동시에 맞아야 한다. 그래서 별도 layer로 묶어 설명하는 것이 좋다.

## 확인 command

```sh
bitbake-getvar DISTRO_FEATURES | grep selinux
grep -E '^(packagegroup-core-selinux|refpolicy-targeted|selinux-init)' build/buildhistory/images/textbook/glibc/textbook-core-image/installed-package-names.txt
```

