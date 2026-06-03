# 01. workspace와 환경 진입점

[학습 순서로 돌아가기](../README.md#추천-학습-순서)

관련 commit:

- `30e7cf1 workspace: add initial environment setup files`

## 필요한 상황

Yocto workspace를 처음 받은 사람이 복잡한 경로와 environment variable을 외우지 않고 바로 build environment에 들어가게 하고 싶다면, 루트에 환경 진입점 스크립트를 둔다.

## 추가하면 되는 것

- 루트에서 source할 `envsetup.sh`
- 빠른 시작을 담은 `README.md`
- 반복 build output을 제외할 `.gitignore`
- IDE가 루트 workspace를 이해하도록 `.vscode/settings.json`
- repo manifest의 `linkfile`로 위 파일들을 루트에 노출

## 이 프로젝트의 구현

파일:

- `layers/meta-textbook/envsetup.sh`
- `layers/meta-textbook/README.md`
- `layers/meta-textbook/settings.json`
- `.repo/manifests/default.xml`

핵심 workflow:

```sh
source envsetup.sh
bitbake textbook-core-image
runqemu textbook-core-image nographic
```

`envsetup.sh`는 자신의 위치를 기준으로 `WORKSPACE_BASE`를 계산한다. 이 방식은 bash와 zsh에서 모두 source 가능한 형태로 작성되어 있다.

## 핵심 메시지

Yocto는 경로와 environment variable이 많다. 그래서 첫 번째 commit의 목적은 feature 구현이 아니라 “사용자가 어디서 시작해야 하는지”를 고정하는 것이다.

## 확인 command

```sh
readlink README.md
readlink envsetup.sh
sed -n '1,80p' envsetup.sh
```
