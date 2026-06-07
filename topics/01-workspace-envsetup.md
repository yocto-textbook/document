# 01. Workspace and Environment Entrypoint

[Back to Learning Path](../README.md#learning-path)

Related Commit:

- `30e7cf1 workspace: add initial environment setup files`

## When to Use

Use a workspace entrypoint when users should be able to enter the Yocto build
environment without memorizing paths, build directories, or environment
variables.

## What This Chapter Covers

This chapter explains how `envsetup.sh` fixes the starting point of the
workspace. It calculates the workspace root, selects the build directory,
exports key variables, and sources `poky/oe-init-build-env`.

## Required Additions

| item | role |
| --- | --- |
| `envsetup.sh` | build environment entrypoint sourced from the workspace root |
| `README.md` | quick-start command guide |
| `.gitignore` | ignore repeated build outputs |
| `.vscode/settings.json` | help the IDE understand the workspace root |
| repo manifest `linkfile` | expose layer-owned files at the workspace root |

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

Core workflow:

```sh
source envsetup.sh
bitbake textbook-core-image
bitbake linux-textbook -c deploy
runqemu textbook-core-image nographic slirp
```

`envsetup.sh` computes `WORKSPACE_BASE` from its own location. It then exports
`TEMPLATECONF`, `MACHINE`, and `DISTRO` before entering the build directory.
After sourcing it, the current directory is `build/`.

## Key Takeaway

Yocto has many paths and environment variables. The first workspace commit is
therefore about fixing where users start, not about adding a product feature.

## Verification Commands

```sh
source envsetup.sh
pwd
echo "$WORKSPACE_BASE"
echo "$MACHINE"
echo "$DISTRO"
```
