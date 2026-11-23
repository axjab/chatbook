---
title: sudo integration tasks
description: "Verified versus reported progress, outstanding tests, and next-session handoff"
type: todo
tags: [security, sudo, pam, tasks]
created: 2026-11-22
updated: 2026-11-23
---

# YubiKey-backed sudo — task checklist and handoff

**Last reviewed:** 2026-11-23 from conversation evidence. **Owner decision: sudo work is complete; move on.** The early-touch issue is documented as a known minor defect, not a blocker. **Status:** Main's touch-only sudo works (owner reported); no exhaustive negative-test output recorded. Spare intentionally deferred until Main is tested. Choose one RP ID **per host** (decision A), not a shared fleet-wide RP.

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

## 2026-11-23 PAM lockout and recovery — verified report

- During experiments to extract the sudo FIDO2 rule into a reusable PAM service and customize prompts, both `sudo` and `su` became unusable from the normal user session.
- User recovered root access by temporarily editing the GRUB Linux command line to boot with `init=/bin/bash`, repaired configuration, and returned to normal boot.
- **Exact failing rule and root cause were not established.** Do not infer that a particular include, prompt module, or control flag caused the failure without inspecting the saved configurations.
- The GRUB edit made using `e` is temporary; `init=/bin/bash` must not be persisted. This recovery route depends on bootloader and disk-encryption access.
- Do not assume the current on-disk PAM configuration matches any earlier suggested example; inspect it first.

### Mandatory safety procedure for authentication changes

1. **Inspect** live PAM stacks, includes, sudoers, fallback paths and the precise command's effects.
2. **Explain** all persistent changes, hardware effects, PAM control flow and rollback before executing.
3. **Prepare recovery:** preserve known-good configuration, a working independent root session, and tested rollback instructions. A root password via `su -` is a useful emergency path, but **not independent of PAM** when both share `common-auth`.
4. **Test in isolation** with a separate PAM service and appropriate test harness (for example `pamtester`); test both success and failure paths.
5. **Change one production variable at a time**; do not combine refactoring with UI/notification customization.
6. **Verify immediately** in a separate session: correct key, absent/wrong key, cancellation, timeout, fallback, `sudo-i`, and recovery. Keep the root session open until verification completes.
7. **Record** exact observed behavior, rollback and final configuration; separate confirmed facts from hypotheses.

### Specific design lessons

- PAM `sufficient`, `required`, `requisite`, and `include`/ `substack` semantics must be checked in the context of the *complete* stack; syntactic validity is not sufficient.
- Presentation (`pam_echo`, `pam_exec`, `tty-notify`) must never grant access or make authentication depend on successful notification delivery. Verify sudo's PAM message-suppression behavior and execution UID rather than assuming either.
- Prefer service-specific PAM experiments over changing `common-auth` globally, especially because shared files can affect `su`, login, SSH and unattended services.
- Keep a securely stored, tested emergency root password while pursuing passwordless **daily** authentication; avoid turning the recovery route into another untested FIDO-only dependency.
- Recovery through GRUB is a last resort, not a substitute for independent access and tested backups.

## Conditional insertion gate (2026-11-23)

**Reported after real-hardware use:** the original insertion gate works. A known issue occurs if the sensor is held while inserting the key: first authentication may fail and the password prompt may join the touch line. The owner accepted this and previously declared the baseline sudo task complete. The later bounded-retry enhancement below is a separate, not-yet-deployed prototype.

- [x] Prototype complete: POSIX shell, bounded 3-second default insertion wait, 250 ms polls, yellow single-line prompt, terminal clearing, and fallback exit status.
- [x] Eight mocked/pseudo-terminal scenarios passed: inserted, mid-wait insertion, timeout, removed while waiting, missing USB tool, invalid TTY, countdown and invalid timeout. Shell syntax passed `sh -n`.
- [x] Verify Linux-PAM `[success=ignore default=1]` skip semantics, `pam_exec` error mapping, and documented `lsusb -d 1050:` nonmatch exit.
- [ ] **Before deploying any enhancement:** Inspect the actual working host sudo service and includes; preserve a known-good independent recovery path; back up the exact PAM files to be changed.
- [ ] Test script with real YubiKey Main and Spare/unknown USB devices; validate `PAM_TTY`, actual `pam_exec` UID, terminal permissions and output in sudo.
- [ ] Validate PAM in a separate test service first, then sudo: no-key timeout/password fallback, correct key touch, wrong/unenrolled key fallback, failed program execution, and no authentication bypass.
- [ ] Test sudo-i and noninteractive sessions; confirm prompt cleanup and Ctrl+C/touch-wait behavior. A preflight script does not impose a timeout on `pam_u2f`.
- [ ] Record exact deployed configuration and observed test outcomes; preserve original `common-auth` and password recovery.

## Optional enhancement — retry after unsuccessful FIDO (2026-11-23)

- [x] Updated POSIX shell script with `retry` mode: `KEY NOT VERIFIED`, wait for removal and reinsertion, then `TOUCH THE KEY`; bounded waits default to three seconds.
- [x] Added `rejected` mode: clear the prior touch line and print `KEY NOT VERIFIED` with a newline before password fallback.
- [x] Verified 14 shell/pseudo-terminal cases using mocked USB status, including successful simulated replacement, removal/insertion timeouts, and missing terminal; syntax passed `sh -n`.
- [x] Verified 7 branch scenarios using Linux-PAM's real control interpreter via a **temporary, isolated PAM service**; simulated `pam_permit` and `pam_deny` stand in for FIDO and fallback. A display-only `[default=ignore]` rule never supplies authentication.
- [ ] **NOT DEPLOYED:** Keep the owner's working two-rule gate unchanged until an independent root recovery session and rollback are ready.
- [ ] Inspect and test an optional two-attempt PAM fragment; `default=4` on the first gate skips all retry modules if no key, `default=1` on the second gate skips only its FIDO module on timeout.
- [ ] Test incorrect/unenrolled key then correct key, missed touch, absent key, fallback, timeout, sudo-i, and no-auth-bypass with actual hardware on Debian 13.
- [ ] Distinguish generic authentication rejection from a proven wrong key. USB vendor detection cannot determine which credential the key holds.

## Outstanding in recommended order

- [ ] **First next-session action:** Read actual current sudo service configuration and review which experimental lines remain; inspect only, do not assume prior proposed examples match live state. Keep a known-good admin route open.
- [ ] Record actual Main test results: fresh touch, no key fallback, wrong key fallback, no touch, no routine PIN, and cache behavior. Do not mark a test passed on predicted output alone.
- [ ] Verify sudo-i behavior and the independent recovery login; decide whether sudo-i needs separate treatment.
- [ ] Remove any unwanted experimental/debug output or PAM lab changes from production sudo after verification.
- [ ] Then register discoverable sudo credential on Spare, inspect listing, append its public mapping safely, and independently test correct/incorrect key behavior.
- [ ] Confirm final negative cases, document rollback, and write a manually reproducible per-host procedure.
- [ ] Consider rotating any FIDO PIN disclosed through command-line arguments and avoiding future PINs in shell history.
- [ ] Evaluate touch-wait cancellation separately; insertion gate alone cannot cancel a blocked `pam_u2f` attempt.

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
