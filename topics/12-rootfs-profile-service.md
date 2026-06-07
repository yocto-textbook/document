# 12. Rootfs 서비스 확장

[Back to Learning Path](../README.md#learning-path)

Related Commit:

- `a1f4e45 meta-textbook-rootfs: add new layer and textbook-profile-service`

## When to Use

image가 부팅된 뒤 특정 스크립트를 systemd 서비스로 실행하고, 결과 파일을 rootfs 안에 남기고 싶다면 rootfs 확장 layer와 systemd recipe를 추가한다.

## What This Chapter Covers

이 chapter는 boot 이후 target에서 실행되는 runtime feature를 image에 포함하는 방법을 설명한다. systemd unit과 script를 recipe로 설치하고, packagegroup을 통해 rootfs에 포함해 진단 결과를 남기는 흐름을 다룬다.

## Required Additions

| 항목 | 역할 |
| --- | --- |
| rootfs 전용 layer | runtime/rootfs 기능을 별도 layer로 분리 |
| systemd unit 파일 | boot 후 실행할 service 정의 |
| 실행 스크립트 | service가 호출할 실제 진단 logic |
| `inherit systemd` | systemd package install/enable 처리 |
| `SYSTEMD_SERVICE:${PN}` | package가 제공하는 unit 이름 지정 |
| `SYSTEMD_AUTO_ENABLE` | boot 시 자동 enable 여부 지정 |
| packagegroup `.bbappend` | image에 service package 포함 |

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

recipe:

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

서비스가 남기는 결과:

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

Rootfs 확장은 단순히 package를 더 넣는 것뿐 아니라, target이 부팅 후 스스로 진단 정보를 남기게 만드는 방식까지 포함한다. systemd recipe는 이런 운영 feature를 image에 녹이는 좋은 예제다.

## Verification Commands

```sh
grep textbook-profile buildhistory/images/textbook/glibc/textbook-core-image/installed-package-names.txt
bitbake linux-textbook -c deploy
runqemu textbook-core-image nographic slirp
```

QEMU login 이후 target 안에서 확인:

```sh
systemctl status textbook-profile.service
ls -l /var/log/textbook-profile/latest
```
