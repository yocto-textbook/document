# 12. Rootfs 서비스 확장

[학습 순서로 돌아가기](../README.md#추천-학습-순서)

관련 commit:

- `a1f4e45 meta-textbook-rootfs: add new layer and textbook-profile-service`

## 필요한 상황

image가 부팅된 뒤 특정 스크립트를 systemd 서비스로 실행하고, 결과 파일을 rootfs 안에 남기고 싶다면 rootfs 확장 layer와 systemd recipe를 추가한다.

## 추가하면 되는 것

- rootfs 전용 layer
- systemd unit 파일
- 실행 스크립트
- `inherit systemd`
- `SYSTEMD_SERVICE:${PN}`
- `SYSTEMD_AUTO_ENABLE`
- packagegroup `.bbappend`

## 이 프로젝트의 구현

파일:

- `meta-textbook-rootfs/conf/layer.conf`
- `meta-textbook-rootfs/recipes-rootfs/systemd/textbook-profile-service.bb`
- `meta-textbook-rootfs/recipes-rootfs/systemd/files/textbook-profile.service`
- `meta-textbook-rootfs/recipes-rootfs/systemd/files/textbook-profile.sh`
- `meta-textbook-rootfs/appends/packagegroups/packagegroup-textbook-core.bbappend`

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
/var/log/textbook-profile/<timestamp>/summary.txt
/var/log/textbook-profile/<timestamp>/blame.txt
/var/log/textbook-profile/<timestamp>/critical-chain.txt
/var/log/textbook-profile/<timestamp>/security.txt
/var/log/textbook-profile/latest
```

## 핵심 메시지

Rootfs 확장은 단순히 package를 더 넣는 것뿐 아니라, target이 부팅 후 스스로 진단 정보를 남기게 만드는 방식까지 포함한다. systemd recipe는 이런 운영 feature를 image에 녹이는 좋은 예제다.

## 확인 command

```sh
grep textbook-profile build/buildhistory/images/textbook/glibc/textbook-core-image/installed-package-names.txt
runqemu textbook-core-image nographic
systemctl status textbook-profile.service
ls -l /var/log/textbook-profile/latest
```

