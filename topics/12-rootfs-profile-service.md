# 12. Rootfs Service Extension

[Back to Learning Path](../README.md#learning-path)

Related Commit:

- `a1f4e45 meta-textbook-rootfs: add new layer and textbook-profile-service`

## When to Use

Add a rootfs service recipe when the target should run a script after boot and leave diagnostic output inside the rootfs.

## What This Chapter Covers

This chapter explains how to include a runtime feature in the image. A recipe installs a systemd unit and a script, the packagegroup pulls that package into the image, and the target records boot/profile information after it starts.

## Required Additions

| Item | Description |
| --- | --- |
| rootfs layer | Keeps runtime/rootfs features separate from build-only metadata. |
| systemd unit file | Defines the service that runs after boot. |
| executable script | Contains the diagnostic logic called by the service. |
| `inherit systemd` | Adds systemd packaging and enablement behavior. |
| `SYSTEMD_SERVICE:${PN}` | Names the unit provided by the package. |
| `SYSTEMD_AUTO_ENABLE` | Controls whether the service is enabled at boot. |
| packagegroup `.bbappend` | Includes the service package in the image. |

## Project Implementation

```text
.
└── meta-textbook-rootfs
    ├── conf/layer.conf
    ├── recipes-rootfs
    │   └── systemd
    │       ├── textbook-profile-service.bb
    │       └── files
    │           ├── textbook-profile.service
    │           └── textbook-profile.sh
    └── appends/packagegroups/packagegroup-textbook-core.bbappend
```

Recipe:

```bitbake
inherit systemd
SYSTEMD_SERVICE:${PN} = "textbook-profile.service"
SYSTEMD_AUTO_ENABLE = "enable"

RDEPENDS:${PN} += "systemd-analyze"

do_install() {
    install -d ${D}${systemd_system_unitdir}
    install -m 0644 ${S}/textbook-profile.service ${D}${systemd_system_unitdir}

    install -d ${D}${bindir}
    install -m 0755 ${S}/textbook-profile.sh ${D}${bindir}/textbook-profile.sh
}
```

Runtime output:

```text
/var/log/textbook-profile
├── latest -> <timestamp>/
└── <timestamp>
    ├── summary.txt
    ├── blame.txt
    ├── critical-chain.txt
    └── security.txt
```

## Key Takeaway

A rootfs extension is not limited to adding static packages. It can also install services that run on the target and produce operational data. This systemd recipe is a compact example of turning a boot-time diagnostic script into an image feature.

## Verification Commands

```sh
grep textbook-profile buildhistory/images/textbook/glibc/textbook-core-image/installed-package-names.txt
bitbake linux-textbook -c deploy
runqemu textbook-core-image nographic slirp
```

After QEMU reaches the login prompt, check inside the target:

```sh
systemctl status textbook-profile.service
ls -l /var/log/textbook-profile/latest
```
