# 02. Build Template and Default Configuration

[Back to Learning Path](../README.md#learning-path)

Related Commit:

- `8198bff build: add default build template configuration`

## When to Use

Use a build template when every new build directory should start with the same
`local.conf` and `bblayers.conf`.

## What This Chapter Covers

This chapter explains how `TEMPLATECONF` fixes the default build policy for the
project. The template defines enabled layers, source mirrors, sstate cache,
buildhistory, and other defaults that should be shared by all users.

## Concept

`TEMPLATECONF` points to a directory containing `local.conf.sample`,
`bblayers.conf.sample`, and optional notes. When `oe-init-build-env` creates a
new build directory, it copies those samples into `build/conf/`.

| file | role |
| --- | --- |
| `local.conf.sample` | local build policy, machine defaults, mirrors, sstate, buildhistory |
| `bblayers.conf.sample` | default layer list |
| `conf-notes.txt` | message shown after entering the build environment |

## Project Implementation

```text
.
└── layers
    └── meta-textbook
        └── meta-textbook-core
            └── conf/templates/default
                ├── bblayers.conf.sample
                ├── conf-notes.txt
                └── local.conf.sample
```

`envsetup.sh` selects the template.

```sh
export TEMPLATECONF=${WORKSPACE_BASE}/layers/meta-textbook/meta-textbook-core/conf/templates/default/
source poky/oe-init-build-env ${WORKSPACE_BASE}/${BUILD_DIR}
```

## Key Configuration

```bitbake
SOURCE_MIRROR_URL ?= "file://${TOPDIR}/downloads"
SSTATE_MIRRORS ?= "file://.* file://${TOPDIR}/sstate-cache/PATH"
INHERIT += "buildhistory"
BUILDHISTORY_COMMIT = "1"
```

## Key Takeaway

Reproducibility is not only about source code. The selected layers, mirrors,
caches, and buildhistory policy are also part of the project.

## Verification Commands

```sh
source envsetup.sh
echo "$TEMPLATECONF"
sed -n '1,120p' conf/bblayers.conf
```
