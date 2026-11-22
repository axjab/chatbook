# GnuPG, OpenPGP, and gopass — design pending

**Goal:** Use physical YubiKeys for relevant GPG operations while preserving access to existing secrets managed through gopass.

## Starting point

A **reported existing RSA3072 OpenPGP key** has:
- A primary key with signing/certification capability.
- An encryption subkey.
- Creation date **2026-10-07** and stated expiry **2027-10-07**.

The full fingerprint and private-key state are intentionally omitted here under the public-notebook policy. The original keyring should be inspected locally.

## Critical distinctions

The YubiKey's **OpenPGP smartcard application** is independent of its **FIDO2/WebAuthn application**. A FIDO passkey cannot be assumed to decrypt gopass data or sign GPG messages.

Placing key material on a card, generating hardware-backed subkeys, and using a GPG agent are different operations with different recovery and portability implications.

## No implementation decision yet

Open questions: reuse existing keys or generate card-backed subkeys; Main/Spare portability and backup strategy; PIN/touch policy; handling encryption/decryption and signing separately.

**Do not** perform keytocard, replace keys, delete any software secret key, or change recipients before:
1. Inspecting local public and secret key material.
2. Establishing tested backups and a decryption recovery route.
3. Selecting a two-key strategy.
4. Testing operations without risking existing gopass access.

No key migration is recorded as completed.
