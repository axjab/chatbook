---
title: "YubiKey fundamentals"
description: "OTP, FIDO2 credential types, and key operations"
type: note
tags: [security, yubikey, fido2]
created: 2026-11-22
updated: 2026-11-22
---

# YubiKey fundamentals and discoveries

## One physical key, several independent applications

A YubiKey can expose Yubico OTP, FIDO U2F, FIDO2/WebAuthn, OATH HOTP/TOTP, OpenPGP, and PIV. These technologies have distinct credentials and interfaces. Enrolling a WebAuthn passkey does not automatically configure GPG, SSH, or PAM.

## Why touching the key types changing text

In a terminal experiment, touching the key caused a 44-character ModHex string followed by Enter. This is consistent with **Yubico OTP** sent through the USB HID keyboard interface.

A standard OTP has a **12-character public-ID prefix** plus a **32-character encrypted OTP part**. The prefix commonly stays constant; counters, timing, and other encrypted data cause later OTPs to differ. The token is not the AES secret key, but an unused OTP may still function as an authentication factor; do not publish live tokens.

Do **not** conflate the Yubico OTP public ID, hardware serial number, FIDO credential ID, and OpenPGP fingerprint. They identify different things.

## FIDO2 credential models

| | Non-discoverable | Discoverable (resident) |
|---|---|---|
| Individual credential record can be listed on device | No | Yes |
| Consumes discoverable-credential capacity | No | Yes |
| Supports hardware-backed cryptographic authentication | Yes | Yes |
| Automatically appears on the Spare key | No | No |

**Decided:** For new sudo authentication credentials, prioritize **discoverable** records so the physical key itself gives a partial inventory. The **-r / --resident** registration option creates a new resident credential; it does not convert an earlier one.

Discoverability is **not** a complete list of all service enrollments. Non-discoverable credentials and non-FIDO applications require separate tracking.

Creating or managing a discoverable credential may require a FIDO PIN even if the ultimate authentication policy requires touch but no PIN.

## The registration operation we learned from

**Reported executed:** **pamu2fcfg -v**.

- The **-v** option means verbose, not version.
- It requested FIDO credential registration from the physical authenticator.
- It created a **non-discoverable** credential, not an enumerable resident record.
- It printed a mapping containing a credential ID, public key, ES256 type, and presence requirement.
- It did **not** save a mapping file or change PAM/sudo configuration.
- It did **not** consume a discoverable credential slot.
- Authenticator-internal changes not observable from output (such as counters) should not be guessed at.

Accurate language distinguishes **a real cryptographic registration operation** from **persistent storage of an individually listable credential record**. Saying simply that the command “did not modify the key” is misleading.

## Relevant references

- [Yubico pam-u2f](https://developers.yubico.com/pam-u2f/)
- [pamu2fcfg manual](https://developers.yubico.com/pam-u2f/Manuals/pamu2fcfg.1.html)
- [Yubico discoverable credentials](https://developers.yubico.com/WebAuthn/WebAuthn_Developer_Guide/Resident_Keys.html)
