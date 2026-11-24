---
title: YubiKey application capabilities and terminology
description: "Unreviewed field guide to YubiKey applications, smartcards, certificates, slots, and use cases"
type: note
tags: [security, yubikey, openpgp, piv, fido2, unreviewed]
created: 2026-11-24
updated: 2026-11-24
---

# YubiKey applications — field guide

> **UNREVIEWED — 2026-11-24.** Research notes for owner review. Explanations below summarize accessible vendor/GnuPG materials; they are not proof of what is configured on any particular physical key. No hardware changes were performed as part of this research.

## First principles

A multi-protocol YubiKey is a miniature cryptographic computer. Its *applications* are independent security systems, not accounts in one shared keychain. They generally have different PINs, key slots, and tooling. A FIDO passkey is **not** the same credential as an OpenPGP encryption subkey, and PIV is not another name for OpenPGP.

- **Public/private key pair:** A public key can be distributed; a private key proves identity, signs, or decrypts. A private key generated *on* the YubiKey generally cannot be exported. A private key imported from a file can be restored elsewhere from an independently saved backup.
- **Smartcard:** A protocol and secure computing model. A USB YubiKey acts as both the card and its reader, so no external card reader is required for ordinary USB use.
- **CCID:** USB smartcard transport used for PIV, OpenPGP, OATH and YubiHSM Auth.
- **HID FIDO:** USB interface for FIDO/U2F operations. **HID keyboard:** OTP can type text as if it were a keyboard. USB product ID showing OTP+FIDO+CCID describes enabled interfaces, not every configured credential.
- **PIN:** Knowledge factor authorizing operations in an application; not the same as a GPG private-key passphrase. **Touch:** Physical-presence authorization; may be required in addition to PIN.
- **Slot:** Reserved location for a particular type of credential/key. Applications have distinct slot models; slots are not universally interchangeable.
- **Certificate:** A signed statement associating a public key with an identity or claim. It contains no usable private key. PIV normally uses X.509 certificates; OpenPGP identities and signatures use another format. An *attestation certificate* instead vouches for aspects of how a device/key was produced.
- **CSR (certificate signing request):** Package containing a public key and requested identity data submitted to a certificate authority (CA) for signature. **Self-signed:** The certificate is signed with its own corresponding private key; not automatically trusted by other computers.

## What the six ykman application groups do

| Application | Core job | Example uses | Host communication / extra hardware |
| --- | --- | --- | --- |
| **FIDO (FIDO2/WebAuthn, U2F)** | Authenticate to a website or program using a cryptographic challenge | Passkeys for online accounts; hardware 2FA; Linux sudo via pam-u2f; FIDO2 SSH; LUKS2 unlock with systemd-cryptenroll | USB HID FIDO, NFC on compatible models/readers; no external HSM |
| **OpenPGP** | Hold keys used by GPG for decryption, signatures and authentication | gopass/pass, encrypted files, signed Git commits, OpenPGP mail, OpenPGP-derived SSH authentication | USB CCID smartcard + GnuPG / gpg-agent / scdaemon; no extra physical reader |
| **PIV** | Hold keys and X.509 certificates used by enterprise/certificate-based software | Smartcard desktop login, corporate VPN, EAP-TLS Wi-Fi, document signing, PKCS#11 clients, some SSH integrations | USB CCID + PIV middleware/PKCS#11 or OS smartcard support; a CA/trust configuration may be required |
| **OATH** | Store HOTP/TOTP secrets used to generate short verification codes | Six-digit 2FA codes for sites that support authenticator apps | USB CCID/NFC + Yubico Authenticator or compatible client; phone/PC supplies time for TOTP |
| **OTP** | Keyboard-style one-time strings or challenge-response | Yubico OTP, static-password suffix, HMAC-SHA1 challenge-response with an integrated app, legacy systems | USB HID keyboard for typed OTP; challenge-response via host software; two configurable touch slots |
| **YubiHSM Auth** | Authenticate a client to a separate hardware security module | An admin unlocking access to keys stored in a **YubiHSM 2** | USB CCID + YubiHSM client and an external YubiHSM 2; this does *not* turn a YubiKey into the full HSM |

Firmware and model matter. As documented for YubiKey 5 Series, firmware 5.7+ offers up to 100 discoverable FIDO2 credentials and 64 OATH credentials; older versions commonly support 25 and 32 respectively. FIDO also supports many non-discoverable registrations that do not occupy the discoverable inventory. Not every YubiKey model supports the listed applications.

## Application audit — 2026-11-24 (owner-reported)

The YubiKey 5 application list has **seven entries**: Yubico OTP, FIDO U2F, FIDO2, OATH, PIV, OpenPGP, and YubiHSM Auth. U2F and FIDO2 appear separately in device configuration, although both belong to the FIDO family.

| Application | Decision / observed status |
| --- | --- |
| Yubico OTP | Disabled; no confirmed relying service |
| FIDO U2F | Disabled; sudo, Tailscale, GitHub, Fastmail and Bitwarden were tested successfully afterward |
| FIDO2 | Retained; used for passwordless authentication |
| OATH | Retained; next area of work |
| PIV | Disabled; no current certificate/PKCS#11 use case |
| OpenPGP | Retained; used for GPG decryption |
| YubiHSM Auth | Disabled; no YubiHSM 2 use case |

**Lesson:** FIDO2 does not require the legacy U2F application to be enabled, but existing U2F-only registrations could still depend on it. The five successful login tests validate those workflows, not every possible registration. Application disablement is reversible and does not itself erase credentials. **Check Main and Spare independently**; the full seven-application state was not independently verified on both keys.

### OATH and the proposed `auth` CLI

OATH stores TOTP/HOTP secrets on the YubiKey; a client such as `ykman` generates codes. Unlike FIDO2, TOTP codes are phishable. No completed Government of Canada OATH enrollment was verified in this session.

The owner chose a minimal, terminal-first **`auth` wrapper** around `ykman` (specification written; **not implemented**):

- `auth` shows OATH codes by default; `auth <query>` selects one; `auth ls` lists accounts.
- `auth add`, `auth rename`, `auth rm` manage OATH credentials with clear feedback and safe secret input. **Do not enable `--touch` when adding.**
- `auth fido` lists **discoverable FIDO2 credentials only**; it cannot inventory non-discoverable website registrations.
- `auth pgp` inspects OpenPGP key slots and available metadata.
- The FIDO and PGP views are **strictly read-only**. No generic `ykman` passthrough or unnecessary features.
- **New requirements:** `auth` watches TOTP codes in place without clearing the terminal; `auth doctor` performs a read-only comparison against the approved USB/NFC application matrix (FIDO2, OATH, OpenPGP enabled; OTP, U2F, PIV, YubiHSM Auth disabled).

**Full implementation handoff:** [Auth CLI engineering prompt](auth-cli.md).

**Resume:** Implement/test `auth` using the session's Claude engineering prompt and `ykman` CLI reference; then enroll and verify OATH on both keys. Keep the public notebook free of TOTP secrets, PINs, recovery codes, and private registration details.

Reference: [YubiKey Manager OATH commands](https://docs.yubico.com/software/yubikey/tools/ykman/OATH_Commands.html).

## PIV is keys + certificates + data objects

**PIV** means *Personal Identity Verification*. It implements the smartcard credential system commonly used by operating systems and organizations. A normal PIV key is generated or imported into a numbered slot; an X.509 certificate links its public key to an identity trusted by the client.

**Main PIV slots:** `9a` authentication (computer or VPN login); `9c` digital signature (document signing); `9d` key management (decryption); `9e` card authentication (physical access or card-to-system identification). Additional retired-key slots can retain prior decryption keys. PIN and touch policies for PIV keys are generally chosen at generation/import time and cannot simply be changed afterwards.

Translate the user's subcommands:

- `ykman piv certificates generate`: make a *self-signed* X.509 certificate for a key; for instance a private test environment. A company's CA may instead issue a trusted certificate from a CSR made with `request`.
- `certificates import/export/delete`: manage the public certificate held in a PIV slot; exporting a certificate does **not** export the secret private key.
- `ykman piv keys generate`: create an on-device key pair (private key stays inside); `import`: load a pre-existing secret key (back up separately); `export`: return the public key, **not the private key**; `attest`: produce proof a key was generated on this YubiKey; `info` inspect metadata; `move` and `delete` alter card contents.
- `ykman piv objects`: read/write PIV *data objects* such as CHUID (Card Holder Unique Identifier), CCC, or implementation-defined fields. These are small structured smartcard fields, **not** a general-purpose secure file vault. `generate chuid` writes card-identity metadata. `import` alters persistent data; arbitrary objects are not automatically confidential.

For a practical scenario: an organization issues an employee an X.509 client certificate for company Wi-Fi (EAP-TLS). The certificate shows who the employee is; the YubiKey's PIV private key proves possession without exporting that private key.

## OpenPGP is the GPG-oriented smartcard application

**OpenPGP** is the public-key encryption/signature ecosystem used by GnuPG. A YubiKey's OpenPGP application has conventional slots:

| `ykman openpgp keys info` slot | Plain meaning | Usage |
| --- | --- | --- |
| `sig` | Sign | Sign files, messages, or Git commits |
| `dec` (called `enc` by some older tools/docs) | Decrypt | Decrypt GPG-encrypted files and a gopass store |
| `aut` | Authenticate | OpenPGP authentication, possibly SSH through gpg-agent |
| `att` | Attest | Vouch for generated card keys; advanced use |

The slot is **one key**, not unlimited credentials. An OpenPGP *primary key* typically certifies an identity; everyday encryption, signing, and authentication can use purpose-specific *subkeys*. For a single encryption-only goal, the existing encryption subkey can be considered without moving the primary key.

Translate the other commands:

- `ykman openpgp info`: inspect app version, PIN retry counters, signature-PIN policy and KDF status.
- `access change-pin/change-admin-pin/change-reset-code`: change the routine PIN, management PIN and recovery/unblocking code. `set-retries` alters brute-force protections; this is **not** a harmless query.
- `keys info sig/dec/aut/att`: inspect slot type, origin and touch policy. `keys set-touch` **changes** the persistent touch requirement; `fixed` and `cached-fixed` can be difficult or impossible to reverse without replacing key material.
- `keys attest`: make an attestation certificate for a card key. `keys import` in ykman is **for the attestation private key only**, not the routine way to import a GPG encryption subkey. GnuPG's `keytocard` is the routine approach for moving GPG subkeys.
- `certificates`: import/export/delete certificates associated with OpenPGP slots; separate from the normal OpenPGP public key's user ID/self-signatures.
- `reset`: **destructive** OpenPGP-app reset, not a harmless refresh. It does not imply a reset of FIDO/PIV/OATH.

**Reported card state from a local OpenPGP info display:** OpenPGP specification 3.4; three remaining User PIN and Admin PIN tries; reset-code retries zero; signature PIN policy `Once`; KDF disabled. Zero reset-code tries normally means no reset code configured; it must not be interpreted as a usable recovery method. Signature policy `Once` does **not** establish that decryption is PIN-free. Never guess current PINs from factory defaults.

## GPG and pinentry: a vital nuance

- A **software GPG secret key** is protected by its local passphrase.
- An **OpenPGP card secret key** is protected by card PIN and, optionally, touch; `pinentry` may still be the interface asking for the PIN.
- A card often retains an authorized PIN state while powered and inserted, so repeated decryptions may stop prompting until removal/reset. This is principally **card-side verification state**, not a guaranteed GnuPG PIN cache.
- Strongest straightforward goal: *reduce repeated prompts while keeping PIN+touch protection*. Completely suppressing PIN authentication is a separate security trade-off, **not** an automatic outcome of `keytocard`.

The OpenPGP application's PIN, PIV PIN, FIDO PIN, and GPG passphrase are separate. A FIDO2 credential does not directly serve as a GPG OpenPGP decryption subkey. Integrating a FIDO-based key-derivation design would be a separate architecture, not native GnuPG OpenPGP smartcard behavior.

## Procedure comparison: what matters before deploying gopass

The [Yubico OpenPGP support article](https://support.yubico.com/s/article/Using-Your-YubiKey-with-OpenPGP) introduces card use; the [drduh guide](https://github.com/drduh/YubiKey-Guide) presents a more comprehensive offline-master-key, subkey, backup, multi-card, and testing workflow. The [GnuPG smartcard wiki](https://wiki.gnupg.org/SmartCard) is a relevant index; its page was not directly accessible during this review, so corroboration comes from [GnuPG's official gpg-card documentation](https://www.gnupg.org/documentation/manuals/gnupg/gpg_002dcard.html). The Yubico support site's dynamic page also did not render in this session; current Yubico technical manuals and the Yubico PGP walk-through were used for authoritative technical details.

Most relevant stages, **not approved execution steps**:

1. Identify exact hardware model and firmware, available applications, current slot contents, PIN and touch policies.
2. Inventory *existing* gopass recipients and secret keys; verify that the proposed new key actually can decrypt existing data, or plan an explicit, safely tested recipient migration.
3. Create/test a recoverable secret-key backup in an isolated GnuPG home. Showing the fingerprint of an ASCII-armored secret-key export is **not** proof that a restoration and decryption test will succeed. Move backups off the daily-use machine.
4. Choose Main/Spare strategy: import the same backed-up encryption subkey to both cards, or encrypt the store to distinct recipients. Card-generated private keys cannot be copied out for replication.
5. Only after verified recovery, set OpenPGP PIN and touch policy, transfer the *intended subkey* to the correct hardware slot, and test decryption after removing/reinserting the card.
6. Independently test the spare card and recovery workflow. A GnuPG local card stub can reference a specific physical card; switching may require relearning the card.
7. Do not use destructive `reset`, `keytocard`, `piv keys delete`, `openpgp keys import att`, or slot overwrite as exploratory commands.

## Research-only local inspection (read-only intent)

- `ykman info`: physical key model, firmware and enabled apps. `ykman --version` is the **tool** version, not device firmware.
- `ykman fido info`, `ykman openpgp info`, `ykman piv info`: application summaries.
- `ykman openpgp keys info dec`: inspect decryption key slot in installed CLI that calls it `DEC`. Check `--help` for this installed version rather than assuming all online examples use the same slot spelling.
- `gpg --card-status`: read GnuPG's view of the OpenPGP card; may refresh local card metadata.

These inspections should not intentionally alter card credentials. Redact physical identifiers, private account details and any security-sensitive output before sharing publicly.

## Sources for user review

- [Yubico Technical Manual — applications](https://docs.yubico.com/hardware/yubikey/yk-tech-manual/yk5-apps-index.html)
- [Yubico — USB physical interfaces](https://docs.yubico.com/yesdk/users-manual/yubikey-reference/transports/overview.html)
- [Yubico — OpenPGP](https://docs.yubico.com/hardware/yubikey/yk-tech-manual/yk5-apps-openpgp.html)
- [Yubico — PIV](https://docs.yubico.com/hardware/yubikey/yk-tech-manual/yk5-apps-piv.html)
- [Yubico — firmware capability matrix](https://docs.yubico.com/hardware/yubikey/yk-tech-manual/yk5-firmware-overview.html)
- [Yubico — OpenPGP ykman commands](https://docs.yubico.com/software/yubikey/tools/ykman/OpenPGP_Commands.html)
- [GnuPG — gpg-card](https://www.gnupg.org/documentation/manuals/gnupg/gpg_002dcard.html)
- [drduh/YubiKey-Guide](https://github.com/drduh/YubiKey-Guide)
- [Yubico PGP walk-through](https://developers.yubico.com/PGP/PGP_Walk-Through.html)

## Unreviewed questions

- Which exact device model/firmware applies to Main and Spare? Does either already have OpenPGP or PIV keys occupying a slot?
- Is the actual goal *never see pinentry*, *do not enter a long GPG passphrase repeatedly*, or *PIN-free use solely on touch*? The security consequences differ.
- Should the new GPG key replace the recipient of an existing password store, supplement it, or remain a test identity?
- Which two-key recovery plan, user PIN/touch combination, and verified offline backup method should be approved?
