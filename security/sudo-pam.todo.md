---
title: "sudo integration tasks"
description: "Checklist and next action for YubiKey-backed sudo"
type: todo
tags: [security, sudo, pam, tasks]
created: 2026-11-22
updated: 2026-11-22
---

# YubiKey-backed sudo — task checklist

**Decided:** Main and Spare will each have a discoverable FIDO2 credential for touch-only sudo authentication. The desired normal flow requires no Linux password or mandatory FIDO PIN. Registration may still require the key's FIDO PIN.

**Last reviewed:** 2026-11-22. This is a conversation-based snapshot, not a live inspection.

## Completed or decided

- [x] Install pam-u2f registration and PAM tooling (reported).
- [x] Select a root-controlled authorization mapping path.
- [x] Decide on touch-only sudo authentication.
- [x] Choose discoverable credentials for new registrations.
- [x] Understand the earlier non-discoverable registration experiment.

## Pending in order

- [ ] Confirm **Main** is inserted; register a new discoverable credential with `pamu2fcfg -r -v` after explicitly explaining hardware changes, PIN/touch, and recovery.
- [ ] Inspect the discoverable credential on Main, e.g. with `ykman fido credentials list`.
- [ ] Register a separate discoverable credential on **Spare**, then inspect it.
- [ ] Preserve both public credential mappings; securely create the root-controlled central authorization file.
- [ ] Inspect the actual sudo, sudo-i, and relevant shared PAM configuration on the host.
- [ ] Test PAM/FIDO authentication separately without risking working sudo access.
- [ ] Apply only the narrowly scoped sudo PAM change.
- [ ] Verify successful passwordless sudo independently with Main and Spare.
- [ ] Verify negative cases (absent or incorrect key, failed touch) and working recovery.
- [ ] Document a manually reproducible procedure for other Debian hosts.

## Resume

**The next registration command has been recommended but is not confirmed executed.** Stop after registering and inspecting Main. Do not assume a command shown in a discussion has run.

Read [the sudo/PAM explanation](sudo-pam.md) for context and safety requirements.
