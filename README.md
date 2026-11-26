---
title: README.md
description: "Index of the public conversational knowledge base"
type: note
tags: [chatbook, index]
created: 2026-11-22
updated: 2026-11-26
---

# Chatbook

A **public, human-friendly shared knowledge base** maintained from conversations. It preserves useful discoveries, decisions, open questions, and the next steps needed to resume work. This is a reference notebook, not a transcript archive or a source-code repository.

**AI contributors:** Read [AGENTS.md](AGENTS.md) first. It is the single top-level set of publishing and editing rules.

## Quick setup cheatsheets

- [Passwordless sudo — FIDO2](SUDO-FIDO.md) — register Main/Spare, install all four authentication components, add `@include fido`, verify fallback and roll back safely.
- [Passphraseless GPG and gopass](GPG-YUBIKEY.md) — generate and back up a key once, provision both YubiKeys, then decrypt on any Debian/Ubuntu host with only the public key.
- [Touch-key notifications](TOUCH-NOTIFICATIONS.md) — deploy one systemd user service for GPG/FIDO touch prompts in desktop and terminal sessions.

## Browse by subject

- [AI mastery and agent infrastructure](ai-mastery/overview.md) — objectives, agent fundamentals, hosting and memory considerations, and the proposed curriculum.
- [AI mastery next steps](ai-mastery/ai-mastery.todo.md) — prioritized infrastructure decisions and experiment checklist.
- [SFTPGo 2.7 Getting Started](ai-mastery/sftpgo-getting-started.md) — comprehensive notes on user accounts, permissions, virtual folders, groups, storage backends and optional protocols.

- [Host Doctor](host-doctor/README.md) — local health checks and USB presence detection.
- [YubiKey fundamentals](security/yubikey/yubikey-fundamentals.md) — OTP, FIDO2, credential discoverability, and hardware effects.
- [YubiKey application capabilities — **UNREVIEWED**](security/yubikey/applications.md) — FIDO, OpenPGP, PIV, OATH, OTP, YubiHSM Auth; smartcards, certificates and practical use cases.
- [Auth CLI engineering prompt](security/yubikey/auth-cli.md) — complete Claude handoff for watching OATH codes, safe account management, read-only FIDO/PGP views and configuration doctor.
- [Registration inventory](security/yubikey/registrations.md) — reported enrollment, verification gaps, and recordkeeping.
- [PAM fundamentals and experiments](security/pam/fundamentals.md) — PAM services, auth/account/password/session, module stacking, output, and an isolated lab.
- [sudo and PAM](security/yubikey/sudo-pam.md) — touch-only sudo design, evidence, problems, and safeguards.
- [sudo task checklist and handoff](security/yubikey/sudo-pam.todo.md) — verified/reported progress and the exact next-session actions.
- [Hardware-backed SSH](security/yubikey/ssh.md) — design alternatives and next decisions.
- [GPG and gopass](security/yubikey/gpg-gopass.md) — earlier card-backed decryption milestone and remaining independent backup-restoration checks.
- [YubiKey touch notifications](security/yubikey/touch-notifications.md) — observed GPG touch events, Unix socket hooks, and initial service deployment research.
- [Passwordless web applications](security/yubikey/web-sso.md) — Pocket ID and downstream integration.
- [Script authentication](security/yubikey/script-auth.md) — the experimental challenge-response idea.
- [Disk encryption and FIDO2](security/yubikey/luks-fido2.md) — LUKS2 enrollment prerequisites and recovery.
- [Authentication/host identity checkpoint](security/yubikey/integration.todo.md) — pending greeter, disk unlock and integration tasks.

## How to use these notes

Read the relevant topic page and its task list (if present). Entries distinguish **Verified, Reported, Decided, Proposed, Pending,** and **Unknown**. Dates are snapshots, not live system monitoring. Material marked **UNREVIEWED** has not yet been approved by the owner.

**Privacy:** This is public. It intentionally excludes secrets, sensitive host configurations, private-network addressing, and exact private hardware identifiers. The complete operational inventory stays outside Chatbook.
