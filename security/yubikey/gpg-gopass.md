---
title: GPG and gopass
description: "OpenPGP hardware security, new-key checkpoint, and recovery planning"
type: note
tags: [security, gpg, gopass, yubikey, unreviewed]
created: 2026-11-22
updated: 2026-11-24
---

# GnuPG, OpenPGP, and gopass — configuration paused

> **UNREVIEWED 2026-11-24 UPDATE.** This checkpoint distinguishes user-reported local operations from independently tested results. Hardware changes were stopped at the owner's request; the current focus is learning the device's full capability.

## Goal

Unlock an existing GPG-encrypted password store using the hardware-held OpenPGP encryption/decryption key without repeatedly typing the passphrase for a software private key. Whether the owner also wishes to eliminate **all** PIN prompts is not yet decided: OpenPGP card PIN, GPG software-key passphrase, pinentry interface, and optional touch are different mechanisms.

## Progress reported during 2026-11-24 session

- **Reported:** Created a new Ed25519 signing/certification primary key and cv25519 encryption subkey using `gpg --full-gen-key`, with one-year expiration. The account label, private identifiers, and actual fingerprint are excluded from this public notebook.
- **Reported and shown:** `gpg --list-secret-keys` showed the new locally stored secret primary key (`sec`) and encryption subkey (`ssb`).
- **Reported and shown:** An armored secret-key backup was exported to a local file, and `gpg --show-keys --with-fingerprint` on that file showed the expected primary key and subkey. This **does not establish** that private-key restoration/decryption works; isolated restore testing and off-machine storage are pending.
- **Reported:** `ykman openpgp info` showed OpenPGP specification version 3.4, application version 5.8.0, three User PIN and three Admin PIN retries, zero reset-code retries, signature PIN policy `Once`, and KDF disabled. This does **not** establish that decryption is already PIN-free or that any encryption key is installed in the card's `dec` slot.
- **Not performed/verified:** A YubiKey key transfer, gopass recipient migration, smartcard decryption test, spare-card provisioning, or offline recovery test.
- **Explicit decision:** **Abort all configuration commands** while reviewing authoritative sources and the six distinct YubiKey applications.

## Legacy GPG identity

Before this session a separate **reported existing RSA3072 OpenPGP key** had signing/certification and encryption subkeys, created in October 2026. It may be relevant to the existing gopass recipients. **Do not assume the newly created key is a recipient of any pre-existing encrypted password entry.** The private-key and recipient inventory is not published here.

## What to understand before resuming

- The YubiKey's **OpenPGP** card is separate from **FIDO2** and **PIV**. OpenPGP's encryption/decryption subkey slot (`dec`) is the direct fit for GnuPG/gopass.
- In the conventional OpenPGP card model, GnuPG calls into `gpg-agent` / `scdaemon` to request a private-key operation. After `keytocard`, the usable local secret subkey is replaced by a card reference (stub). The card private key is not exportable.
- A GPG **passphrase** protecting a software key is not a card **PIN**. On-card verified PIN state may persist until card removal/reset; it does not guarantee zero prompts and must not be confused with PIN caching by the agent.
- OpenPGP signature policy `Once` only describes signature PIN checking. It does not prove a configured `dec` key or a decryption PIN policy.
- The **OpenPGP Admin PIN**, **User PIN**, and optional **Reset Code** are distinct. Zero reset-code retries does not provide an available recovery channel.
- For a Main/Spare design, an offline copy of an imported encryption subkey could be transferred to both cards, or distinct subkeys could be configured as separate recipients. Each path has different recovery and migration consequences.
- Existing store readability must be validated independently of generating a new GPG identity.

See [YubiKey application capabilities — unreviewed](applications.md) for plain-English app and command terminology. See [drduh/YubiKey-Guide](https://github.com/drduh/YubiKey-Guide) for the offline-backup, dedicated-subkey and dual-card workflow.

## Before any hardware change

- [ ] Establish exact physical device models, firmware versions, and current OpenPGP key slots (without exposing serials publicly).
- [ ] Inspect the actual existing gopass public-key recipient(s) and verify the old/new key relationship.
- [ ] Verify private-key backup via **restoration and a decryption test in an isolated GnuPG home**; store backup offline in safe custody.
- [ ] Decide the Main/Spare approach and recovery method.
- [ ] Understand and select PIN, signature policy, touch settings and acceptable prompt behavior.
- [ ] Only then consider controlled `keytocard`, secret-key stub verification, and a real gopass decryption test.

**Do not** perform `keytocard`, replace keys, delete the existing private key, reset OpenPGP, change recipients, or change authentication policies until these prerequisites are reviewed.

## Sources

- [Yubico OpenPGP application technical documentation](https://docs.yubico.com/hardware/yubikey/yk-tech-manual/yk5-apps-openpgp.html)
- [GnuPG gpg-card manual](https://www.gnupg.org/documentation/manuals/gnupg/gpg_002dcard.html)
- [drduh/YubiKey-Guide](https://github.com/drduh/YubiKey-Guide)
