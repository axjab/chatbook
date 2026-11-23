---
title: Script authentication
description: "Experimental cryptographic authentication for local scripts"
type: note
tags: [security, scripts, yubikey]
created: 2026-11-22
updated: 2026-11-23
---

# Authentication for arbitrary scripts — experimental

**Goal:** Require an actual YubiKey authentication operation before executing a chosen script or command.

**Status:** Owner requested low-effort integration from Bash, Python and Go (2026-11-23). No reusable authorization helper or script-PAM service is verified.

## The crucial distinction

Detecting a USB device or matching a Yubico vendor/product identifier only establishes **apparent presence**. It is not proof that the hardware holds an authorized private key.

The experiment should use a genuine cryptographic challenge-response mechanism or a properly scoped existing authentication component. Decide what is being authorized (running a local command, gaining privileges, or releasing a secret) before choosing an implementation.

**Low-effort architecture:** Prefer one carefully scoped authentication helper shared by all three languages rather than independent FIDO implementations. A dedicated, key-only PAM service backed by `pam_u2f` can be exercised through a PAM-aware CLI such as `pamtester`; each language checks its exit status. The helper and mapping must be readable in its actual privilege context. Reusing a root-only sudo mapping from an unprivileged caller will not necessarily work.

A normal user can edit or bypass a script running as that same user; a positive check in such a script is an interaction gate, **not** a privilege or secret-access security boundary. For protected actions, enforce the authorization in a trusted privileged service or controlled executable, and do not accept a password fallback if the purpose is specifically YubiKey presence.

This must be tested separately from production PAM, sudo, SSH, and secret-management policy. Require a real cryptographic FIDO assertion; USB presence alone is insufficient.
