---
title: "YubiKey sudo and PAM"
description: "Touch-only sudo architecture, observed progress, pitfalls and recovery"
type: note
tags: [security, linux, pam, sudo]
created: 2026-11-22
updated: 2026-11-23
---

# Passwordless sudo with YubiKey and PAM

**Status (2026-11-23):** Main's discoverable credential and touch-based sudo authentication **reported working** on Debian 13. The exact currently deployed PAM file has not been independently re-read after the experiments. Spare has deliberately not been enrolled for sudo.

## Chosen behavior

- Authentication for sudo: a recognized YubiKey touch, with neither a Linux password nor a routine FIDO PIN.
- This is hardware-backed cryptographic verification, **not** a sudoers NOPASSWD rule.
- Preserve a Linux-password fallback and independent administrator recovery access during setup.
- Limit the initial change to sudo's PAM service; do not edit shared common-auth.
- Give Main and Spare separate credentials, and test Main thoroughly before enrolling Spare.
- **Decided:** one relying-party identity **per host** for now (approach A). Do not roll out one shared RP across hosts merely to save credential slots.

## What actually happened

**Reported and observed in conversation:**
1. An earlier pamu2fcfg -v registration generated a non-discoverable credential; it did not save a mapping or configure sudo.
2. Main was enrolled with pamu2fcfg -r -v. The command requested the FIDO PIN and printed a mapping. A subsequent ykman fido credentials list showed a discoverable PAM credential for the local host and account.
3. The user reported saving an authorization mapping in a root-controlled file and verifying its ownership and permissions. The mapping contains verification material, not the private signing key.
4. Before editing, the sudo and sudo-i service files were inspected. Each included common-auth. A narrowly scoped pam_u2f authentication rule was proposed and tried only for sudo.
5. The user subsequently reported that touch-based sudo **worked**. A no-key test was executed, but the exact outcome was not pasted. Do not claim every negative/fallback scenario passed.
6. When sudo waited for a touch, repeated Ctrl+C produced ^C without promptly cancelling the operation. Signal-based workarounds, timeout behavior and custom cancellation have **not** been verified.

**Pending:** Independent Spare registration, thorough recorded positive/negative cases and recovery verification, and a reproducible procedure for other hosts. See [current task list and handoff](sudo-pam.todo.md).

## Why this works

~~~text
sudo -> PAM service "sudo"
  -> pam_u2f.so -> local authorization mapping
  -> FIDO challenge for the chosen RP ID
  -> private credential signs on YubiKey after touch
  -> PAM verifies using public credential material
  -> sudo accepts authentication or follows its password fallback
~~~

The RP ID defaults to pam://<hostname> unless explicitly configured. This is a **logical relying-party label**, not an HTTP URL, network connection or central authentication server. A credential bound to one RP ID cannot automatically serve another. A local Unix account name is distinct from the YubiKey credential's display name and from the RP ID.

A discoverable credential's **display name** is human-readable metadata; its **username** names the user within the RP, not an automatic mapping to every Unix username. Different hosts may have different local usernames. The local authorization file controls which account accepts which public credential.

For simplicity, use per-host RP IDs. A shared RP is technically possible with compatible authorization mappings on each host, but has cross-host isolation and inventory trade-offs. Do not silently change origin during future registration.

## PAM configuration lessons

Example syntax only, **not an assertion about the current host file**:

~~~text
auth sufficient pam_u2f.so authfile=/path/to/root-controlled-mapping cue debug
@include common-auth
~~~

- sufficient: a successful FIDO authentication can short-circuit the remaining auth stack; a failed attempt falls through to password authentication.
- authfile=: point to the mapping actually saved; without it, pam_u2f looks in its per-user default location.
- cue: ask the user to touch a matching inserted authenticator; debug: diagnostic logging.
- A custom multiword cue may use [cue_prompt=Touch your key] as **one bracketed argument**. This is documented PAM syntax.
- The sudo-i PAM service can be different from sudo; do not assume one edit covers both.

Unlike interactive, which prompts **before** testing for the key, normal detection first checks whether an authorized authenticator is inserted. cue typically does not display when no matching key is detected. nodetect disables this precheck and is not the desired solution to conditional insertion prompts.

Changing a PAM file does not require restarting a daemon; it affects new PAM operations. A caller such as sudo controls which messages it displays and may request PAM_SILENT. A failed pam_echo greeting was an observed example, even while the pam_u2f touch cue appeared. See [PAM fundamentals](pam-fundamentals.md).

## Testing and known limits

For a fresh auth attempt, sudo -k invalidates a cached authentication timestamp and sudo -v requests validation. This does not edit PAM or YubiKey credentials. The authentication cache otherwise allows later sudo operations without repeating touch until its policy requires authentication again.

Expected test cases: correct Main/touch, no key/password fallback, wrong key/password fallback, correct key/no touch, authentication without a routine FIDO PIN, cache behavior after removing the key, and root/admin recovery. Document **actual observations**, not just expected outcomes.

There is no documented pam_u2f timeout= setting in the consulted manual. The request for (1) show an insertion prompt **only if** the correct key is absent, (2) wait for hotplug, and (3) reliably cancel a pending touch is **proposed follow-up engineering**, not an existing production feature. Built-in interactive prompts unconditionally; cue is touch-specific; nodetect skips precheck. An external pre-PAM script alone cannot guarantee cancellation inside pam_u2f. A custom component would require a separate fail-closed implementation and isolated testing before any production use.

Never put FIDO PINs in command-line arguments or publish credential mappings, PINs, IDs or debug output. If a PIN was exposed in an argument or transcript, treat it as exposed, review shell history safely, and rotate it.

## Recovery and references

Maintain a known-good administrative login while changing PAM. Rollback should remove **only** the newly added sudo PAM line, using the preserved access; keep common-auth unchanged. Never deploy experimental pam_debug success, alwaysok or nouserok into a privileged authentication path.

- [Yubico pam_u2f manual](https://developers.yubico.com/pam-u2f/Manuals/pam_u2f.8.html)
- [Yubico pamu2fcfg manual](https://developers.yubico.com/pam-u2f/Manuals/pamu2fcfg.1.html)
- [PAM fundamentals and experiments](pam-fundamentals.md)
- [Current checklist and resume point](sudo-pam.todo.md)
