---
title: GPG and gopass
description: "OpenPGP hardware security and key recovery planning"
type: note
tags: [security, gpg, gopass]
created: 2026-11-22
updated: 2026-11-23
---

# GnuPG, OpenPGP, and gopass — design pending

**Goal (confirmed 2026-11-23):** Decrypt the existing gopass password store and GPG-encrypted files using a YubiKey instead of repeatedly entering the software private-key passphrase, while preserving recovery access. No migration has yet been verified. **Postponed by owner on 2026-11-23** until graphical authentication and disk unlock are resolved.

## Session checkpoint

A fresh software GPG key was **proposed** for repeatable hardware provisioning, but its creation, secret-key backup, encrypted test, and YubiKey transfer have **not been confirmed**. Do not assume example commands from chat were executed. Revisit hardware capabilities, independent Main/Spare recovery and the active password-store recipients before generating or migrating keys.

## Starting point

A **reported existing RSA3072 OpenPGP key** has:
- A primary key with signing/certification capability.
- An encryption subkey.
- Creation date **2026-10-07** and stated expiry **2027-10-07**.

The full fingerprint and private-key state are intentionally omitted here under the public-notebook policy. The original keyring should be inspected locally.

## Critical distinctions

The YubiKey's **OpenPGP smartcard application** is independent of its **FIDO2/WebAuthn application**. A FIDO passkey cannot be assumed to decrypt gopass data or sign GPG messages.

Placing key material on a card, generating hardware-backed subkeys, and using a GPG agent are different operations with different recovery and portability implications.

## Preferred direction and remaining decisions

Explore the YubiKey **OpenPGP encryption/decipher subkey slot**, not its existing FIDO2 passkeys. GnuPG and gopass can use the hardware-held decryption subkey through `gpg-agent`/smartcard support. This typically exchanges the software key passphrase for the **OpenPGP PIN** (which may be cached) and, where configured and supported, a touch requirement. It is not automatically PIN-free.

Two-key strategies include importing an existing backed-up encryption subkey onto both compatible cards, or using different encryption recipients; the latter can require recipient or encryption changes. Carefully verify hardware model/firmware and RSA3072 support. **Neither strategy has been selected or executed.** Never make a YubiKey the only recoverable copy of a decryption key.

## No implementation decision yet

Open questions: reuse existing keys or generate card-backed subkeys; Main/Spare portability and backup strategy; PIN/touch policy; handling encryption/decryption and signing separately.

**Do not** perform keytocard, replace keys, delete any software secret key, or change recipients before:
1. Inspecting local public and secret key material.
2. Establishing tested backups and a decryption recovery route.
3. Selecting a two-key strategy.
4. Testing operations without risking existing gopass access.

No key migration is recorded as completed.
