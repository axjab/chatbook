---
title: sudo integration tasks
description: "Verified versus reported progress, outstanding tests, and next-session handoff"
type: todo
tags: [security, sudo, pam, tasks]
created: 2026-11-22
updated: 2026-11-23
---

# YubiKey-backed sudo — task checklist and handoff

**Owner update (2026-11-23):** sudo/FIDO is considered **complete by owner report**. This supersedes the earlier pending-deployment status below. The earlier experiments and test checkboxes remain historical evidence, **not** a current instruction to repeat enrollment or redeploy; independent negative-test records and per-device details were not provided. Do not restart production sudo work unless explicitly requested. The active resumption point is [integration checkpoint](integration.todo.md).

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

**Reported after real-hardware use:** the original insertion gate works. A known issue occurs if the sensor is held while inserting the key: first authentication may fail and the password prompt may join the touch line. The owner accepted this as a minor baseline defect. The former bounded-retry enhancement was later rejected; the current proposal below is simpler.

- [x] Prototype complete: POSIX shell, bounded 3-second default insertion wait, 250 ms polls, yellow single-line prompt, terminal clearing, and fallback exit status.
- [x] Eight mocked/pseudo-terminal scenarios passed: inserted, mid-wait insertion, timeout, removed while waiting, missing USB tool, invalid TTY, countdown and invalid timeout. Shell syntax passed `sh -n`.
- [x] Verify Linux-PAM `[success=ignore default=1]` skip semantics, `pam_exec` error mapping, and documented `lsusb -d 1050:` nonmatch exit.
- [ ] **Before deploying any enhancement:** Inspect the actual working host sudo service and includes; preserve a known-good independent recovery path; back up the exact PAM files to be changed.
- [ ] Test script with real YubiKey Main and Spare/unknown USB devices; validate `PAM_TTY`, actual `pam_exec` UID, terminal permissions and output in sudo.
- [ ] Validate PAM in a separate test service first, then sudo: no-key timeout/password fallback, correct key touch, wrong/unenrolled key fallback, failed program execution, and no authentication bypass.
- [ ] Test sudo-i and noninteractive sessions; confirm prompt cleanup and Ctrl+C/touch-wait behavior. A preflight script does not impose a timeout on `pam_u2f`.
- [ ] Record exact deployed configuration and observed test outcomes; preserve original `common-auth` and password recovery.

## Current change — rejection notice and Ctrl+C during insertion (2026-11-23)

**Decided:** One FIDO attempt only. On verifier failure display `WRONG KEY` in red and go straight to password. No removal/reinsertion prompts. Ctrl+C during insertion clears the line and should skip the FIDO attempt. The old retry design is explicitly superseded.

- [x] Implement `wrong` display-only mode: red `WRONG KEY` (`#EF4444`), newline before password; do not authenticate from display output.
- [x] Remove the optional second FIDO attempt and all `REMOVE THE KEY` prompts.
- [x] Trap SIGINT during the shell gate, clear prompt, and exit `130`; also handle termination/hangup as error paths.
- [x] Update proposed PAM to three lines: `[success=ignore default=2]` gate; `sufficient pam_u2f`; `[default=ignore]` rejection notice.
- [x] Run 13 local shell/pseudo-terminal checks (`sh -n` included), eight isolated Linux-PAM control-flow tests using mocked verifier and fallback, and a separate controlling-PTY literal Ctrl+C test; all passed.
- [ ] **NOT INSTALLED on the owner host:** Inspect exact current sudo and fido PAM configuration. Back up files and keep independent root recovery before making any changes.
- [ ] Test real unregistered key, recognized key, failed touch, password fallback, Ctrl+C during insertion and safe recovery on Debian 13.
- [ ] Test Ctrl+C separately during `pam_u2f` touch wait: the gate script is no longer running and cannot guarantee the desired fallback. This may cancel the sudo operation.
- [ ] Record the actual deployed state once verified. Do not assert that `WRONG KEY` proves an unregistered key: it is a generic message for any FIDO failure.

## Outstanding in recommended order

- [ ] **First next-session action:** Read actual current sudo service configuration and review which experimental lines remain; inspect only, do not assume prior proposed examples match live state. Keep a known-good admin route open.
- [ ] Record actual Main test results: fresh touch, no key fallback, wrong key fallback, no touch, no routine PIN, and cache behavior. Do not mark a test passed on predicted output alone.
- [ ] Verify sudo-i behavior and the independent recovery login; decide whether sudo-i needs separate treatment.
- [ ] Remove any unwanted experimental/debug output or PAM lab changes from production sudo after verification.
- [ ] Then register discoverable sudo credential on Spare, inspect listing, append its public mapping safely, and independently test correct/incorrect key behavior.
- [ ] Confirm final negative cases, document rollback, and write a manually reproducible per-host procedure.
- [ ] Consider rotating any FIDO PIN disclosed through command-line arguments and avoiding future PINs in shell history.
- [ ] Evaluate touch-wait cancellation separately; insertion gate alone cannot cancel a blocked `pam_u2f` attempt.

## Next-session instructions

Read [AGENTS.md](../AGENTS.md), [sudo overview](sudo-pam.md), this checklist, [PAM fundamentals](pam-fundamentals.md), [YubiKey fundamentals](yubikey-fundamentals.md), and [registrations](registrations.md). Check each against observed host state before changing files. Explain every operation's hardware, filesystem, PAM and recovery effects. Use Inspect -> Explain -> Implement -> Test -> Verify -> Record; make one consequential change at a time.

**Do not restart** at pamu2fcfg -r -v on Main, suggest shared RP IDs, or register Spare before confirming the outstanding baseline tests. No further changes to authentication configuration were made by this handoff update.
