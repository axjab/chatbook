---
title: Authentication and host identity checkpoint
description: "Resumption point for graphical authentication, universal account identity and disk unlock"
type: todo
tags: [security, yubikey, pam, luks, hosts, tasks]
created: 2026-11-23
updated: 2026-11-23
---

# Authentication checkpoint — 2026-11-23

**This session is closed.** The owner reported YubiKey-backed sudo complete. Do not infer completion of every negative test from that report. This page is the active resumption point.

## Decided

- **One universal human login name:** The owner selected the same established personal handle for systems under personal control and external services. For privacy, this public notebook does **not** reproduce the exact account name or internal per-host mappings. Existing shorter local account names remain until a deliberate migration; do not mass-rename accounts. Service accounts remain separate.
- **Graphical sign-in:** Enable a YubiKey option for Dux/DankMaterialShell's greetd login while preserving a working password fallback. The user enabled a Dux security-key option but reported that insertion was not recognized; the location of a documented Sync control was unclear.
- **Sudo:** Owner reports complete. Its custom `fido` stack is built for terminal use; do **not** assume `@include fido` works in a graphical greeter. In particular, its helper requires a tty. Use and validate the greeter's own PAM-compatible `pam_u2f` flow.
- **Disk encryption:** Investigate unlocking LUKS2 with a YubiKey during early boot, retaining the current passphrase as recovery. Nothing has been enrolled or modified for disk unlock.
- **GPG/gopass hardware decryption:** Postponed until graphical login and disk-unlock issues are resolved. Do not generate or migrate keys as a continuation of this checkpoint; previous example commands were suggestions, not evidence of execution.

## Outstanding work in order

- [ ] **Diagnose Dux security-key detection:** Identify Dux/DMS greeter version and actual greeter binary. Inspect the current `/etc/pam.d/greetd` and related Dux PAM service and verify which mapping file the greeter uses. Compare the reported detection failure with the working terminal FIDO path. Determine why the documented Sync control is absent in the installed version. Do not overwrite PAM rules blindly.
- [ ] **Resolve Dux PAM ownership:** Decide whether Dux manages `/etc/pam.d/greetd` through its greeter sync, or the operating system manages it. Test a real key and password fallback from a recoverable session before relying on graphical login.
- [ ] **Provide an appropriate Dux setup video** only after verifying that it covers the same greeter version and security-key authentication; do not label an unrelated appearance/setup video as a PAM tutorial.
- [ ] **Inspect the encrypted root device and initramfs:** Determine LUKS version, mapped block device, actual `/etc/crypttab`, boot initramfs tooling and whether FIDO2 unlocking is supported at boot. Distinguish boot-time unlock from user-session PAM authentication.
- [ ] **Plan safe LUKS2 enrollment:** Once compatibility and backups are verified, enroll Main and Spare as independent unlock methods; keep an existing verified passphrase/recovery slot; test reboots and removal/failure scenarios before retiring any method.
- [ ] **Plan username reconciliation:** Inventory home paths, ownership/UIDs, services, sudo policies, user-level configuration and identity-dependent app state privately; migrate host-by-host only with tested recovery.
- [ ] **Reconcile stale sudo and service registration notes:** Record evidence of actual Main/Spare enrollments and tests, rather than assuming successful deployment equals exhaustive testing.
- [ ] **Later — GPG:** Return to new-key planning and hardware-held decryption only after higher priorities are resolved. Verify independent recovery copies before `keytocard` or recipient changes.

## Confirmed lessons

1. An approved universal account name is a **convention**, not a completed migration. A one-letter user is convenient but looks cryptic; an established personal handle is a better durable identity than a role or project name.
2. The YubiKey's FIDO2 authenticator is not the same as its OpenPGP or PIV smartcard application. A Dux security-key prompt normally means FIDO/U2F, not smartcard insertion.
3. A terminal PAM integration does not transfer unchanged to a graphical greeter. PAM service, device access, conversation handling, authorization mapping and fallback all matter.
4. USB detection proves presence only, not successful enrollment or FIDO authorization. An inserted key can precede a usable HID/FIDO interface.
5. A documented UI control or command may differ by installed Dux version. Inspect locally before running sync operations that rewrite PAM rules.
6. Early-boot LUKS unlocking uses a separate key enrollment and boot integration. Existing sudo/greeter registration does not configure encrypted disk unlock.
7. Preserve independent recovery and make one authentication change at a time. Never remove the remaining working disk passphrase without verified alternate unlock paths.

## Authoritative reference material

- [Dux greeter authentication](https://duxlabs.dev/docs/dms-greeter/authentication) — security key, PAM policy ownership and greeter sync.
- [Dux greeter project README](https://github.com/duxlabs-dev/dux-greeter) — documents `dms-greeter sync --auth`; command changes PAM when Dux manages authentication.
- [systemd-cryptenroll](https://www.freedesktop.org/software/systemd/man/latest/systemd-cryptenroll.html) — FIDO2 LUKS2 enrollment, recovery keys and risks.
- [crypttab](https://www.freedesktop.org/software/systemd/man/latest/crypttab.html) — `fido2-device=` unlock settings.
- [Disk-encryption research and prerequisites](luks-fido2.md).

## Safe resume point

1. **Read-only greeter inspection:** `cat /etc/pam.d/greetd`, inspect existing Dux-related PAM policies, `fido2-token -L`, `dms-greeter --help`. Examine output before touching login policy.
2. **Read-only disk inspection:** `lsblk -o NAME,TYPE,FSTYPE,MOUNTPOINTS`, `cat /etc/crypttab`, check actual initramfs packages. Do not run `systemd-cryptenroll --fido2-device=auto` yet.
3. After diagnosing both, select one controlled, reversible change and validate a full recovery path.

**Safety:** No YubiKey enrollment, key creation, PAM edit, username migration, disk-header change, or boot configuration change was performed by this Chatbook checkpoint.
