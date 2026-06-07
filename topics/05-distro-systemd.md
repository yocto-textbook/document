# 05. Distro and systemd

[Back to Learning Path](../README.md#learning-path)

Related Commit:

- `e99d22f meta-textbook-core: introduce textbook-core-distro configuration`
- `17998dd distro: introduction of systemd-enabled distribution and ecosystem updates`

## When to Use

Use distro configuration when the project needs to fix product policy such as
distro name, version, init system, feature set, and runtime providers.

## What This Chapter Covers

This chapter explains distro configuration independently from hardware
selection. The machine chooses the target. The distro chooses how the Linux
product should behave.

## Concept

`machine` selects hardware or the QEMU target. `distro` selects product policy.
The same machine can boot with different init systems, feature sets, runtime
providers, and package management policy depending on the distro.

`systemd` is an init and service manager. It starts the system, manages service
dependencies, runs unit files, and integrates with logging and service state.
In Yocto, switching to systemd is not only about installing one package. The
distro feature set and virtual runtime providers must also be changed.

| concept | role |
| --- | --- |
| `DISTRO` | selected product policy |
| `DISTRO_FEATURES` | capabilities enabled by the distro |
| `VIRTUAL-RUNTIME_*` | concrete runtime providers for abstract dependencies |
| `systemd` | init and service manager |
| `usrmerge` | filesystem layout expected by systemd-based distros |

## Project Implementation

```text
.
└── layers
    └── meta-textbook
        ├── envsetup.sh
        └── meta-textbook-core
            └── conf/distro
                ├── textbook-core-distro.conf
                └── textbook-systemd-distro.conf
```

Key configuration:

```bitbake
DISTRO ?= "textbook-systemd-distro"
DISTRO_NAME ?= "Textbook Systemd Distro (Textbook Systemd Distro)"
DISTRO_VERSION ?= "1.0.0"
DISTRO_CODENAME = "scarthgap"

DISTRO_FEATURES:append = " systemd usrmerge"
DISTRO_FEATURES:remove = " sysvinit"
VIRTUAL-RUNTIME_init_manager = "systemd"
VIRTUAL-RUNTIME_initscripts = "systemd-compat-units"
DISTRO_FEATURES_BACKFILL_CONSIDERED = "sysvinit"
```

## Key Takeaway

Machine configuration is hardware policy. Distro configuration is product
runtime policy.

## Verification Commands

```sh
source envsetup.sh
bitbake-getvar DISTRO
bitbake-getvar DISTRO_FEATURES
bitbake-getvar VIRTUAL-RUNTIME_init_manager
```
