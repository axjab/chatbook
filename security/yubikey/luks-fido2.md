---
title: LUKS2 disk unlock using a YubiKey
description: "Boot-time FIDO2 disk decryption prerequisites, documentation and safe test plan"
type: note
tags: [security, yubikey, luks, fido2, linux]
created: 2026-11-23
updated: 2026-11-23
---

# LUKS2 disk unlocking with a YubiKey — research, not configured

**Owner request (2026-11-23):** Replace routine disk-unlock passphrase entry with a YubiKey, while retaining a tested passphrase as recovery. The disk currently asks for a passphrase before the graphical greeter. Its actual LUKS version, encrypted device and boot tooling have **not yet been inspected**, so compatibility is not confirmed.

## What technology is involved

- LUKS2 supports multiple unlock mechanisms or key slots; a YubiKey unlock can be **added alongside** a passphrase rather than replacing it immediately.
- `systemd-cryptenroll` can enroll a **FIDO2 hmac-secret** authenticator on a LUKS2 volume. Its FIDO2 enrollment is **separate** from sudo `pam_u2f` and Dux's graphical login. A key registered for one is not automatically enrolled for the others.
- At boot, **systemd's** unlocking path needs to recognize the enrolled token. The `fido2-device=` setting is documented in `crypttab(5)`. A traditional boot initramfs may need separate integration; do not assume that installing `fido2-tools` enables FIDO2 root unlocking.
- Hardware behavior may include touch and/or a FIDO2 PIN, depending on authenticator and enrollment policy.

## Read-only first step

~~~sh
lsblk -o NAME,TYPE,FSTYPE,MOUNTPOINTS
cat /etc/crypttab
dpkg -l initramfs-tools dracut systemd-cryptsetup 2>/dev/null | grep '^ii'
systemd-cryptenroll --fido2-device=list
~~~

These commands inspect disk topology, crypttab configuration, installed boot/cryptsetup packages and connected FIDO2 authenticators. They **do not** enroll a token or modify key slots. Redact device UUIDs and sensitive host details before sharing outside the machine.

## Safe implementation sequence — not yet executed

1. Identify the exact encrypted block device and confirm it uses LUKS2. Read its header and available key slots without modifying them.
2. Verify a **separate** recovery path including the existing unlock passphrase, a suitable LUKS header backup and tested access to that backup. A header backup is sensitive security material and should not be published.
3. Determine whether the actual boot initramfs implements systemd's FIDO2 unlocking flow; explicitly establish any needed packages and regeneration process before attempting enrollment.
4. Enroll **one YubiKey** as an *additional* method using the documented `systemd-cryptenroll` options. This changes persistent LUKS metadata; it does not replace the existing passphrase unless slots are deliberately removed.
5. Configure the boot-time unlock integration. Verify full reboot, token present/absent, PIN/touch prompts and passphrase fallback.
6. Independently enroll and validate a Spare device only after the first path works. Keep at least one proven fallback independent of both hardware keys.

**Do not copy an example `--fido2-device=auto` command onto a guessed disk path.** Mistakes in keyslot selection, initramfs or crypttab can cause a boot-time lockout.

## Official documentation

- [systemd-cryptenroll(1)](https://www.freedesktop.org/software/systemd/man/latest/systemd-cryptenroll.html): FIDO2 token enrollment and recovery, including `--fido2-device=`, `--fido2-with-client-pin=`, `--fido2-with-user-presence=`.
- [crypttab(5)](https://www.freedesktop.org/software/systemd/man/latest/crypttab.html): `fido2-device=` option for token-based LUKS2 unlocking.
- [cryptsetup(8)](https://man7.org/linux/man-pages/man8/cryptsetup.8.html): LUKS operation and header recovery precautions.

## Resume

Start by inspecting the user's actual LUKS header version and initramfs. Do not touch disk slots, root boot configuration or recovery passphrases until compatibility is confirmed and a recovery plan is tested.
