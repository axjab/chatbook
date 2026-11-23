---
title: "sudo integration tasks"
description: "Verified versus reported progress, outstanding tests, and next-session handoff"
type: todo
tags: [security, sudo, pam, tasks]
created: 2026-11-22
updated: 2026-11-23
---

# YubiKey-backed sudo — task checklist and handoff

**Last reviewed:** 2026-11-23 from conversation evidence. **Status:** Main's touch-only sudo works (owner reported); no exhaustive negative-test output recorded. Spare intentionally deferred until Main is tested. Choose one RP ID **per host** (decision A), not a shared fleet-wide RP.

## Completed or decided

- [x] Debian 13 host; pam-u2f and related tools reported installed.
- [x] Decide touch-only sudo without Linux password or routine FIDO PIN; preserve password fallback and admin recovery.
- [x] Decide separate discoverable credentials for Main and Spare, independent per-host relying-party IDs.
- [x] Main registered with pamu2fcfg -r -v; user supplied output showing PIN prompt and credential mapping.
- [x] Main's discoverable FIDO credential observed in ykman listing.
- [x] User reported creating root-controlled authorization mapping and verifying metadata.
- [x] Original sudo and sudo-i PAM stacks inspected; both included common-auth.
- [x] User tested a sudo-specific pam_u2f rule and reported touch authentication working.
- [x] A no-key test was performed (exact result not recorded).
- [x] PAM educational lessons captured in [PAM fundamentals](pam-fundamentals.md).
- [x] Cancellation issue reproduced: Ctrl+C during touch wait did not promptly stop sudo.

## Outstanding in recommended order

- [ ] **First next-session action:** Read actual current sudo service configuration and review which experimental lines remain; inspect only, do not assume prior proposed examples match live state. Keep a known-good admin route open.
- [ ] Record actual Main test results: fresh touch, no key fallback, wrong key fallback, no touch, no routine PIN, and cache behavior. Do not mark a test passed on predicted output alone.
- [ ] Verify sudo-i behavior and the independent recovery login; decide whether sudo-i needs separate treatment.
- [ ] Remove any unwanted experimental/debug output or PAM lab changes from production sudo after verification.
- [ ] Then register discoverable sudo credential on Spare, inspect listing, append its public mapping safely, and independently test correct/incorrect key behavior.
- [ ] Confirm final negative cases, document rollback, and write a manually reproducible per-host procedure.
- [ ] Consider rotating any FIDO PIN disclosed through command-line arguments and avoiding future PINs in shell history.
- [ ] Optionally evaluate a separate custom insertion-and-cancellation UI only **after** the secure sudo baseline is stable.

## Separate proposed project: responsive YubiKey interaction

**Not approved for deployment.** Requested behavior:

- Correct registered key already inserted: immediately ask for touch.
- Correct key absent or wrong key present: ask for insertion, wait for hotplug, then request touch.
- Permit reliable cancellation during *both* insertion wait and touch wait; enforce a bounded timeout.
- Cancel/failure must not grant access; preserve recovery and password policy.

Built-in pam_u2f does **not** combine these behaviors: cue handles touch, interactive prompts unconditionally, and nodetect skips detection. A preflight wrapper is insufficient to guarantee interruption of a blocked pam_u2f call. Explore an isolated PAM-aware test harness or module/library changes only after threat modeling, research and fail-closed tests. No implementation or production change has happened.

## Next-session instructions

Read [AGENTS.md](../AGENTS.md), [sudo overview](sudo-pam.md), this checklist, [PAM fundamentals](pam-fundamentals.md), [YubiKey fundamentals](yubikey-fundamentals.md), and [registrations](registrations.md). Check each against observed host state before changing files. Explain every operation's hardware, filesystem, PAM and recovery effects. Use Inspect -> Explain -> Implement -> Test -> Verify -> Record; make one consequential change at a time.

**Do not restart** at pamu2fcfg -r -v on Main, suggest shared RP IDs, or register Spare before confirming the outstanding baseline tests. No further changes to authentication configuration were made by this handoff update.
