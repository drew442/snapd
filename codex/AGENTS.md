# Codex Guidance For iSCSI Work In Snapd

Read these first:

1. `../AGENTS.md`
2. `../.github/copilot-instructions.md`
3. `../.codex/session-brief.md`
4. `../.codex/open-questions.md`
5. `iscsi-architecture-plan.md`

## Task Focus

Continue snapd work for iSCSI interfaces with these constraints:

- least privilege first
- role separation over giant umbrella policy
- reuse existing generic storage interfaces where they already solve the problem
- keep Cinder additive rather than folding it into initiator or target

## Preferred Architecture

- `iscsi-initiator`
- `iscsi-target`
- `cinder-support`
- `iscsi-offload` (optional additive, deferred)
- `iscsi-firmware` (optional additive, deferred)

## Practical Development Order

1. Correct `iscsi-initiator`
2. Add or design `iscsi-target` for LIO
3. Preserve current Cinder-related behavior while planning `cinder-support`
4. Defer `iscsi-offload` and `iscsi-firmware`

## Validation Expectations

At minimum:

- built-in interface unit tests
- policy snippet assertions
- kmod assertions where applicable
- do not widen interfaces without explicit justification in comments/tests
- preserve non-regression coverage for current `iscsi-initiator` behavior

## Notes

- The Ubuntu Noble packaged `open-iscsi` layout observed during discussion uses
  `/etc/iscsi` as both config root and DB root.
- Do not assume upstream Meson defaults without checking the target deployment.
- The intended model is host iSCSI stack mediation.
- LIO is the in-scope target implementation for now.
