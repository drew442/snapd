# iSCSI In Snapd: Session Brief

## Goal

Design and implement a maintainable snapd interface architecture for iSCSI that:

- supports initiator-only use
- supports target-only use
- supports optional Cinder-specific additions
- handles transport and feature variants without collapsing everything into one broad interface

## Current Conclusions

### Recommended interface family

1. `iscsi-initiator`
2. `iscsi-target`
3. `cinder-support`
4. `iscsi-offload` (additive, deferred)
5. `iscsi-firmware` (additive, deferred)

### Role split

- `iscsi-initiator`: Open-iSCSI initiator role, discovery, login/logout, session management
- `iscsi-target`: target-side stack for LIO
- `cinder-support`: preserve current Cinder-motivated behavior without making it a replacement for initiator/target
- `iscsi-offload`: hardware/HBA/offload additions such as `iscsiuio` IPC and offload transports
- `iscsi-firmware`: iBFT, flashnode, firmware/OpenFirmware boot-related additions

### Why this shape

snapd's interface framework is connection-driven and additive. Separate built-in
interfaces align better with least privilege, reviewability, and the existing
patterns used by `block-devices`, `dm-multipath`, and `mount-control`.

Avoid a single giant `iscsi` interface with boolean role flags unless a very
small scope attribute is genuinely helpful for narrowing an otherwise stable
role.

## Important Packaging Context

For the Ubuntu Noble `open-iscsi` package that was checked during discussion,
the compiled-in paths observed from `/usr/sbin/iscsiadm` were:

- `HOMEDIR = /etc/iscsi`
- `DBROOT = /etc/iscsi`

Observed persistent subdirectories:

- `/etc/iscsi/ifaces`
- `/etc/iscsi/send_targets`
- `/etc/iscsi/fw`
- `/etc/iscsi/static`
- `/etc/iscsi/isns`
- `/etc/iscsi/nodes`

That means snapd work should be evaluated against the packaged Ubuntu behavior,
not only the upstream Meson defaults in the open-iscsi repo.

## Snapd-Specific Guidance

- Reuse existing generic interfaces where possible:
  - `block-devices`
  - `dm-multipath`
  - `mount-control`
- Keep iSCSI-specific policy in iSCSI-specific interfaces
- Use privileged built-in interface posture:
  - `allow-installation: false`
  - slot limited to `core`/system snap
  - `deny-auto-connection: true`
- Start with static built-ins via `commonInterface` where possible
- Use custom interface code only if needed for validation or generated policy
- Work against host-stack mediation, not a snapped iSCSI stack
- Do not include offload or firmware support in the first implementation

## Confirmed Scope Decisions

- target support in scope: LIO
- offload support: deferred
- firmware/iBFT/flashnode support: deferred
- host-stack mediation: yes
- preserve current `iscsi-initiator` Cinder-related intent in the first pass
- testing is required

## Expected First Changes

1. Fix and narrow `iscsi-initiator`
2. Remove target-side assumptions from it unless strongly justified by current compatibility needs
3. Plan or introduce `iscsi-target` for LIO
4. Preserve current Cinder-related behavior while moving toward a cleaner additive model
5. Include tests with every interface change

## Files Likely To Matter

- `interfaces/builtin/iscsi_initiator.go`
- `interfaces/builtin/iscsi_initiator_test.go`
- `interfaces/builtin/common.go`
- `interfaces/builtin/block_devices.go`
- `interfaces/builtin/dm_multipath.go`
- `interfaces/builtin/mount_control.go`
- `interfaces/builtin/README.md`
