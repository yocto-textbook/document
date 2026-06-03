# 15. devtool 기반 recipe 개발

[학습 순서로 돌아가기](../README.md#추천-학습-순서)

관련 topic:

- `meta-textbook-application`
- `meta-textbook-external`
- SDK workflow

## 필요한 상황

새 software recipe를 빠르게 만들거나, 기존 recipe의 source를 workspace로 꺼내 수정하고, build/배포/patch 반영까지 한 workflow로 처리하고 싶다면 `devtool`을 사용한다.

## devtool을 언제 쓰나

`devtool`은 다음 상황에 잘 맞는다.

- 새 application을 Yocto에 처음 넣을 때
- 기존 recipe를 수정하고 patch를 만들 때
- recipe build 결과를 QEMU나 target에 빠르게 배포해 볼 때
- workspace에서 개발한 변경을 정식 layer로 반영할 때

`externalsrc`와 비교:

- `externalsrc`: 이미 프로젝트가 정한 외부 source tree를 Yocto build에 직접 연결한다.
- `devtool`: 임시 workspace를 만들어 recipe 개발, patch 생성, 배포, 정식 반영을 도와준다.

## 기본 준비

```sh
source envsetup.sh
devtool --help
devtool status
```

`devtool`을 처음 쓰면 build directory 아래에 workspace layer가 생긴다.

```text
build/workspace/
  appends/
  recipes/
  sources/
```

이 workspace layer는 개발용이다. 최종 결과는 `devtool finish`로 프로젝트 layer에 반영한다.

## 새 recipe 추가: devtool add

이미 local source tree가 있는 경우:

```sh
source envsetup.sh
devtool add my-hello external/hello-makefile-application
devtool edit-recipe my-hello
devtool build my-hello
```

remote source에서 시작하는 경우:

```sh
devtool add my-hello https://example.com/my-hello.tar.gz
```

별도 source directory로 checkout하고 싶다면:

```sh
devtool add my-hello /tmp/my-hello-src https://example.com/my-hello.git
```

## 기존 recipe 수정: devtool modify

`hello-makefile-application`을 수정하고 싶다면:

```sh
source envsetup.sh
devtool modify hello-makefile-application
devtool status
```

source 위치는 보통 다음 경로에 생긴다.

```text
build/workspace/sources/hello-makefile-application
```

수정 후:

```sh
cd build/workspace/sources/hello-makefile-application
git status
git add .
git commit -m "hello: update message"
```

## build와 image 반영

recipe만 build:

```sh
devtool build hello-makefile-application
```

workspace package를 포함한 image build:

```sh
devtool build-image textbook-core-image
```

`devtool build`는 recipe의 sysroot populate까지 확인하는 빠른 개발 build에 가깝고, image에 넣어 확인하려면 `devtool build-image`를 쓴다.

## target에 바로 배포

QEMU나 보드에서 SSH가 켜져 있다면:

```sh
devtool deploy-target hello-makefile-application root@192.168.7.2
```

되돌리기:

```sh
devtool undeploy-target hello-makefile-application root@192.168.7.2
```

주의:

- `deploy-target`은 package manager를 통해 설치하는 것이 아니라 install 결과물을 target에 복사한다.
- runtime dependency를 자동으로 설치하지 않는다.
- 이 프로젝트의 SDK layer가 `ssh-server-openssh`, `openssh-sftp-server`를 image에 넣는 이유가 이런 배포 workflow에도 도움이 된다.

## 변경을 정식 layer에 반영

수정한 source commit을 patch와 `.bbappend`로 남기고 싶다면:

```sh
devtool finish hello-makefile-application layers/meta-textbook/meta-textbook-application
```

또는 recipe 자체 업데이트만 하고 싶다면:

```sh
devtool update-recipe hello-makefile-application
```

작업을 버리고 workspace에서 제거:

```sh
devtool reset hello-makefile-application
```

## kernel에도 devtool을 쓸 수 있다

kernel source를 devtool workspace로 꺼내 수정:

```sh
devtool modify linux-textbook
```

build:

```sh
devtool build linux-textbook
devtool build-image textbook-core-image
```

정식 layer 반영:

```sh
devtool finish linux-textbook layers/meta-textbook/meta-textbook-core-bsp
```

주의:

- kernel은 build 시간이 길고 source tree가 크다.
- 이 프로젝트는 이미 `external/linux`와 `meta-textbook-external`의 `externalsrc` workflow를 제공하므로, kernel을 장기간 개발할 때는 `externalsrc`, patch 실험과 recipe 반영에는 `devtool`이 더 어울린다.

## 핵심 메시지

`devshell`은 recipe 안으로 들어가 보는 tool이고, `devtool`은 recipe 개발 과정을 관리하는 tool이다. `devtool add/modify`로 workspace를 만들고, `devtool build/deploy-target`으로 빠르게 확인하고, `devtool finish`로 정식 layer에 반영한다.

## 확인 command

```sh
source envsetup.sh
devtool status
devtool modify hello-makefile-application
devtool build hello-makefile-application
devtool reset hello-makefile-application
```

