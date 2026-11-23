---
title: "YubiKey fundamentals"
description: "OTP, FIDO2 credential types, relying-party identity and hardware operations"
type: note
tags: [security, yubikey, fido2]
created: 2026-11-22
updated: 2026-11-23
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

**Decided:** For new sudo authentication credentials, prioritize **discoverable** records so the physical key itself gives a partial inventory. The -r / --resident registration option creates a new resident credential; it does not convert an earlier one.

Discoverability is **not** a complete list of all service enrollments. Non-discoverable credentials and non-FIDO applications require separate tracking.

Creating or managing a discoverable credential may require a FIDO PIN even if the ultimate authentication policy requires touch but no PIN.

## What registration actually changes

**Reported executed, older trial:** pamu2fcfg -v requested a real FIDO credential registration and printed an authorization mapping. It created a non-discoverable credential, not an enumerable resident record. It did not save a mapping file, change PAM/sudo, or consume a discoverable credential slot. Authenticator-internal state not observable from output should not be guessed at.

**Observed, newer Main registration (2026-11-22/23):** pamu2fcfg -r -v requested PIN authorization and generated a **new persistent discoverable credential**. ykman fido credentials list subsequently showed a matching PAM entry on Main. The command still printed the mapping without saving it to a file or editing PAM. Hardware registration, saved public mapping and PAM policy are three separate operations.

Do not paste actual credential IDs, public-key mappings, PINs or device serials into this public notebook.

## RP ID, username and display name

A PAM FIDO credential has a relying-party identifier (RP ID), a username scoped to that party, and a human-readable display name. The display name is **metadata**, not a login account, password, or authorization rule.

By default pam-u2f uses pam://<hostname> as its RP ID. Despite the URI-like notation, this is a logical label, not a web endpoint or an online connection to the named host. The same RP ID must be requested at authentication as was used during registration; a matching public-key mapping is also needed.

A local Linux username can differ from the username embedded as credential metadata, or from usernames on other hosts. PAM's local authorization mapping determines which user can authenticate with which registered public credential.

**Decided:** Start with a separate RP ID/credential for each host (approach A). A shared RP across many hosts is possible when explicitly configured but changes isolation and inventory characteristics, and does **not** create a centralized authentication server.

Registration and routine authentication are separate: FIDO PIN prompts during discoverable-credential creation do not automatically mean every subsequent sudo touch requires a PIN. Check actual module settings and credential policy.

## Relevant references

- [Yubico pam-u2f](https://developers.yubico.com/pam-u2f/)
- [pamu2fcfg manual](https://developers.yubico.com/pam-u2f/Manuals/pamu2fcfg.1.html)
- [pam_u2f manual](https://developers.yubico.com/pam-u2f/Manuals/pam_u2f.8.html)
- [Yubico discoverable credentials](https://developers.yubico.com/WebAuthn/WebAuthn_Developer_Guide/Resident_Keys.html)
- [sudo status and handoff](sudo-pam.todo.md)
