# Resolved Scope Decisions

These decisions were confirmed by the user and should be treated as current
project direction unless changed later.

## 1. Target stack scope

In scope:

- LIO target support

Not yet in scope as a broader target framework commitment:

- generic support for other target implementations

## 2. `iscsi-offload`

Deferred.

Do not design the first implementation around hardware/offload transports.
Keep the architecture extensible so `iscsi-offload` can be added later without
reworking the core role split.

## 3. `iscsi-firmware`

Deferred.

Do not include first-class iBFT, flashnode, or firmware/OpenFirmware boot
features in the first implementation.

## 4. Cinder scope

The requirement is compatibility with what the current
`interfaces/builtin/iscsi_initiator.go` was trying to cover for Cinder-related
usage. The immediate goal is:

- do not break that expected behavior during refactoring
- preserve any currently intended Cinder-related support while reworking the
  architecture

This should not be interpreted as a mandate to add broad new Cinder-specific
surface area in the first pass.

## 5. Host stack model

The intended model is:

- mediation of the host iSCSI stack

Not required at this stage:

- a snap shipping and running its own iSCSI userspace stack

## 6. Testing

Testing is explicitly required.

At minimum, ongoing work should include:

- built-in interface unit tests
- AppArmor snippet assertions
- kmod assertions where relevant
- tests covering any changed role split and non-regression behavior for the
  current `iscsi-initiator` interface

## Remaining judgment call

The only notable implementation choice still left to engineering judgment is
how aggressively to separate current `iscsi-initiator` behavior from any
Cinder-motivated allowances while keeping compatibility in the first pass.
