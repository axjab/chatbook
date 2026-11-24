---
title: GPG and gopass
description: "Verified YubiKey-backed OpenPGP decryption, PIN and touch behavior, and gopass handoff"
type: note
tags: [security, gpg, gopass, yubikey, openpgp]
created: 2026-11-22
updated: 2026-11-24
---

# GnuPG, OpenPGP, and gopass — hardware decryption verified

## Goal and current state (2026-11-24)

Replace repeated software-private-key passphrase prompts with YubiKey-backed GnuPG decryption, retaining a recoverable key backup and eventually integrating with an existing `gopass` store.

**Verified from direct terminal output on the main YubiKey:**

- A new Ed25519 signing/certification primary key and a Curve25519 encryption subkey were created outside the card. Their actual identifiers are intentionally omitted from this public notebook.
- The owner exported an armored secret-key backup and confirmed its displayed public fingerprint/subkey. **Reported:** backup stored somewhere safe off the machine. **Pending:** prove private-key restoration from this file in a separate environment; inspecting public fingerprints is not a recovery test.
- In GnuPG's `--edit-key` editor, selecting `key 1` marked only the encryption subkey (`ssb*`). `keytocard`, destination **2 (Encryption)**, successfully installed the subkey in the YubiKey's OpenPGP `dec` slot.
- GnuPG reported that the local private key would be deleted only by `save`. The owner exited with `quit` and chose **not to save**, preserving the local private-key material for future backup/spare-card provisioning. The transfer to the card remained persistent.
- `gpg --card-status` and `ykman openpgp keys info DEC` showed a matching card encryption-key fingerprint; the card reported key algorithm X25519 and origin **IMPORTED**. Signature and authentication slots were empty at the time of inspection.
- A fresh, temporary GnuPG home was created, and **only the public key** was imported. `gpg --card-status` discovered the card, and `gpg --list-secret-keys` displayed `sec#` (primary private key unavailable) and `ssb>` (card-backed encryption subkey).
- An independently encrypted test message was decrypted successfully inside that isolated GnuPG home. Because no software private key was imported there, this verified **hardware-backed decryption**.
- **Reported:** The OpenPGP User PIN and Admin PIN were changed from their defaults; the actual PIN values are not recorded. Decryption requested a PIN after insertion, then skipped repeated PIN prompts while the card's verification state remained active.
- `ykman openpgp keys info DEC` verified touch policy **Cached**, meaning one physical touch is required, followed by a 15-second touch cache for subsequent operations. A second decryption within that window needed no further touch.
- [yubikey-touch-detector](https://github.com/maximbaz/yubikey-touch-detector) detected a real OpenPGP touch wait (`GPG_1`, then `GPG_0`). Its desktop notification and an independent socket subscriber's existing terminal-broadcast notification were both observed. See [YubiKey touch notifications](touch-notifications.md).

**Not completed:** Actual `gopass` recipient inspection/migration, decryption of an existing password entry using the card, spare YubiKey provisioning, tested offline recovery, and demonstrated persistent systemd operation of the touch detector.

## Critical distinctions

- The **software GPG private-key passphrase** and the **OpenPGP User PIN** are different secrets. Pinentry may still be the UI for requesting the PIN; this setup eliminates repeated software-key passphrase entry, **not** all possible prompts.
- PIN verification state and touch caching are separate. A fresh card insertion may require a PIN; the `cached` touch setting allows repeat private-key operations without another touch for 15 seconds.
- `sec#` means the primary secret key is not available in a given GnuPG home. `ssb>` means an encryption secret subkey is represented by a card stub; it is **not** an imported software private key.
- Installing an encryption subkey on the card does **not** automatically make the new public key a recipient of existing `gopass` secrets.
- The existing store may be encrypted to a separate, older OpenPGP identity. **Do not remove an old recipient or local private key** before proving access and recovery.

## Reproducible approach and observed behavior

### Prepare the card (already done on the main key)

Use the vendor's [OpenPGP guide](https://support.yubico.com/s/article/Using-Your-YubiKey-with-OpenPGP) for the underlying `keytocard` workflow. The vendor walkthrough uses an RSA example, while the observed configuration uses Curve25519. Transfer **only the intended encryption subkey** to slot 2. Saving editor changes would remove the local private subkey; quitting without saving retained it for subsequent cards.

`scdaemon` is needed for GnuPG smartcard access. In the observed setup, `dpkg-query -W scdaemon` listed a package name even though the package was not installed. `dpkg -L scdaemon` confirmed the absence, and installation resolved the `No SmartCard daemon` error. **Reported:** `scdaemon` was added as a dependency of the `core-sec` metapackage for new hosts.

Prefer `ykman` for YubiKey hardware configuration and `gpg` for cryptographic operations. Examples (commands that change card PINs or touch policies are **persistent**):

```bash
ykman openpgp access change-pin
ykman openpgp access change-admin-pin
ykman openpgp keys set-touch dec cached
ykman openpgp keys info DEC
```

Avoid `fixed` and `cached-fixed` touch policies without explicitly accepting their key-replacement implications.

### Isolated hardware-only decryption test (performed successfully)

Use a fresh temporary GnuPG directory and **only a public-key export**, never the secret-key backup, for a hardware-only test:

```bash
GNUPGHOME="$(mktemp -d)"
export GNUPGHOME
gpg --import /path/to/public-key.asc
gpg --card-status
gpg --list-secret-keys --keyid-format LONG
```

Expected: `sec#` and `ssb>`. Encrypt a disposable test message to the imported public key and decrypt it in this temporary home. The observed test returned the original plaintext, including after one PIN verification and one touch. `unset GNUPGHOME` restores the normal shell environment; deleting the temporary directory requires a separate explicit cleanup.

**Safety:** Transferring a private key with `keytocard` changes the YubiKey persistently. The editor's `quit`/do-not-save choice only governs pending local-key changes; it does not roll back the card write. Backups must remain recoverable and protected.

## Next steps

1. **Inspect existing `gopass` recipients** and confirm which public key(s) currently encrypt entries. Do not assume the new card key grants access to the existing store.
2. Plan and test an additive recipient transition, with a verified rollback and offline private-key recovery, before removing any old recipient.
3. Provision and independently test the **spare** YubiKey; retain an independently protected private-key backup and consider GnuPG card-stub switching behavior.
4. Inspect and enable the detector's packaged user service; separately daemonize the tested socket listener and validate after login/card reinsertion. See [touch notifications](touch-notifications.md).
5. Verify real `gopass` decryption using a card-backed-only GnuPG environment, while preserving access to the original store and its former recipients.

## References

- [Yubico — Using Your YubiKey with OpenPGP](https://support.yubico.com/s/article/Using-Your-YubiKey-with-OpenPGP)
- [Yubico OpenPGP technical manual](https://docs.yubico.com/hardware/yubikey/yk-tech-manual/yk5-apps-openpgp.html)
- [GnuPG smartcard documentation](https://www.gnupg.org/documentation/manuals/gnupg/gpg_002dcard.html)
- [drduh YubiKey Guide](https://github.com/drduh/YubiKey-Guide)
- [YubiKey touch detector](https://github.com/maximbaz/yubikey-touch-detector)
