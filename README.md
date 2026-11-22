# Chatbook

A public notebook maintained from conversations. Its purpose is to preserve useful technical discoveries, decisions, and current work so a discussion can resume without reconstructing its history.

**Last reviewed:** 2026-11-22  
**Scope of this section:** Host Doctor and the related YubiKey/authentication discussions.

## Start here

- [Current state and where to resume](STATUS.md) — short, actionable handoff.
- [Working agreements](WORKFLOW.md) — how we inspect, change, test, and publish.
- [Host Doctor](host-doctor/README.md) — local health checking and USB presence detection.
- [YubiKey fundamentals](security/yubikey-fundamentals.md) — OTP, FIDO2, discoverability, and hardware state.
- [Registration inventory](security/registrations.md) — what is enrolled and what still needs verification.
- [sudo and PAM](security/sudo-pam.md) — decisions, prior experiment, and staged checklist.
- [SSH](security/ssh.md) — hardware-backed SSH alternatives and unresolved design.
- [GPG and gopass](security/gpg-gopass.md) — existing key material and migration safeguards.
- [Passwordless web applications](security/web-sso.md) — Pocket ID and downstream services.
- [Authentication in scripts](security/script-auth.md) — experimental command gating.

## How to read this notebook

Entries deliberately distinguish **Verified**, **Reported**, **Decided**, **Proposed**, and **Pending**. A command appearing here is not proof that it has been run. Check the date and state before relying on operational status.

This is a **public** reference, not a credential vault or full host inventory. It intentionally omits secrets, private-network details, sensitive authentication material, and exact private device identifiers. Machine-specific state must be checked on the actual host.

The notebook is organized by topic, not by chat transcript. When new facts supersede old ones, update the relevant page and its status rather than adding a duplicate session recap.
