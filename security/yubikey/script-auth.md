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

**Status:** Proposed experiment; no implementation verified.

## The crucial distinction

Detecting a USB device or matching a Yubico vendor/product identifier only establishes **apparent presence**. It is not proof that the hardware holds an authorized private key.

The experiment should use a genuine cryptographic challenge-response mechanism or a properly scoped existing authentication component. Decide what is being authorized (running a local command, gaining privileges, or releasing a secret) before choosing an implementation.

This should be tested separately from production PAM, sudo, SSH, and secret-management policy. Scripts must not imply that a successful presence check grants security.
