# 01. workspace와 environment entrypoint

[Back to Learning Path](../README.md#learning-path)

Related Commit:

- `30e7cf1 workspace: add initial environment setup files`

## When to Use

Yocto workspace를 처음 받은 사람이 복잡한 path와 environment variable을 외우지 않고 바로 build environment에 들어가게 하고 싶다면, workspace root에 entrypoint script를 둔다.

## What This Chapter Covers

이 chapter는 project root에서 시작하는 build workflow를 고정하는 방법을 설명한다. `envsetup.sh`가 workspace 위치를 계산하고, 사용자가 같은 command로 BitBake environment에 들어가게 만드는 entrypoint 역할을 한다.

## Required Additions

| 항목 | 역할 |
| --- | --- |
| `envsetup.sh` | workspace root에서 source하는 build environment entrypoint |
| `README.md` | 빠른 시작 command 안내 |
| `.gitignore` | 반복 build output 제외 |
| `.vscode/settings.json` | IDE가 workspace root를 이해하도록 설정 |
| repo manifest `linkfile` | layer 안의 공통 파일을 workspace root에 노출 |

## Project Implementation

```text
.
├── .repo
│   └── manifests
│       └── default.xml
└── layers
    └── meta-textbook
        ├── envsetup.sh
        ├── README.md
        └── settings.json
```

Core Workflow:

```sh
source envsetup.sh
bitbake textbook-core-image
bitbake linux-textbook -c deploy
runqemu textbook-core-image nographic slirp
```

`envsetup.sh`는 자신의 위치를 기준으로 `WORKSPACE_BASE`를 계산한다. 이 방식은 bash와 zsh에서 모두 source 가능한 형태로 작성되어 있다.

## Key Takeaway

Yocto는 경로와 environment variable이 많다. 그래서 첫 번째 commit의 목적은 feature 구현이 아니라 “사용자가 어디서 시작해야 하는지”를 고정하는 것이다.

## Verification Commands

```sh
readlink README.md
readlink envsetup.sh
sed -n '1,80p' envsetup.sh
```
