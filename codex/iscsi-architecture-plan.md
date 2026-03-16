# iSCSI Interface Architecture Plan For snapd

## Objective

Provide a snapd-native iSCSI interface model that supports:

- initiator-only deployments
- target-only deployments using LIO
- optional Cinder support
- optional offload and firmware/boot features

while preserving least privilege and keeping each interface reviewable.

## Architectural Decision

Use composable built-in interfaces rather than one large interface with
multiple role flags.

## Proposed Interface Family

### 1. `iscsi-initiator`

Scope:

- Open-iSCSI initiator role
- discovery
- login/logout
- session management
- initiator config and persistent DB
- initiator-side sysfs and daemon IPC

Should not include:

- target-side configfs/LIO permissions
- Cinder-specific deltas
- broad offload/firmware extras by default

### 2. `iscsi-target`

Scope:

- target-side iSCSI management
- LIO/configfs surface
- target kernel modules and target-specific state/config

Should not include:

- initiator DB
- Cinder-only extras

### 3. `cinder-support`

Scope:

- only the Cinder/os-brick delta on top of one or both iSCSI roles
- Cinder-specific config/state/helper access

First-pass goal:

- preserve the current `iscsi-initiator` Cinder-motivated intent without
  broadening scope unnecessarily

Should not include:

- the complete initiator role
- the complete target role
- generic storage admin powers already modeled elsewhere in snapd

### 4. `iscsi-offload`

Additive optional interface for:

- hardware offload transports
- `iscsiuio` abstract socket access
- extra HBA/NIC-specific sysfs
- transport-specific kernel modules where justified

Status:

- deferred

### 5. `iscsi-firmware`

Additive optional interface for:

- iBFT
- flashnode management
- firmware/OpenFirmware boot metadata

Status:

- deferred

## Composition Model

Examples:

- initiator only:
  - `iscsi-initiator`
- target only:
  - `iscsi-target`
- initiator plus Cinder:
  - `iscsi-initiator`
  - `cinder-support`
- target plus Cinder:
  - `iscsi-target`
  - `cinder-support`
- initiator plus offload:
  - `iscsi-initiator`
  - `iscsi-offload`
- boot/firmware-enabled initiator:
  - `iscsi-initiator`
  - `iscsi-firmware`

## Existing snapd Facilities To Reuse

Do not absorb generic storage features into the iSCSI family if snapd already
has them.

Existing interfaces likely to compose with this work:

- `block-devices`
- `dm-multipath`
- `mount-control`

## Implementation Strategy

### Phase 1

Refine `iscsi-initiator` to the actual packaged Open-iSCSI initiator surface.
Preserve current intended behavior relied on for Cinder-related usage.

### Phase 2

Design and implement `iscsi-target` for LIO.

### Phase 3

Add `cinder-support` as a narrow additive interface, or at minimum separate
current Cinder-related allowances from the initiator role in a way that does
not regress behavior.

### Phase 4

Add `iscsi-offload` and/or `iscsi-firmware` if those features are in scope for
the first delivery.

Current decision:

- both are deferred

## Policy Style

Recommended posture for each privileged interface:

- plug `allow-installation: false`
- slot limited to system snap (`core`/`snapd`)
- `deny-auto-connection: true`

Start with static policy via `commonInterface` where possible.
Move to custom interface implementations only when:

- plug attributes need validation
- policy must be generated
- compatibility rules need custom enforcement

## Testing Requirements

- unit tests for each interface change
- AppArmor snippet assertions
- kmod assertions where relevant
- explicit non-regression coverage for current `iscsi-initiator`
- if interface splitting begins, tests covering intended composition behavior

## Current Known Decisions

See `../.codex/open-questions.md`.
