---
title: "Passwordless web authentication"
description: "Pocket ID passkeys and downstream OIDC integration"
type: note
tags: [security, pocket-id, sso]
created: 2026-11-22
updated: 2026-11-22
---

# Passwordless self-hosted web authentication

**Goal:** Sign into self-hosted web services with YubiKey-backed passkeys, avoiding routine passwords.

## What is known

- **Pocket ID** is the selected existing identity provider.
- A **localhost passkey test succeeded**.
- This does not verify enrollment of both physical keys in a production deployment or adoption by downstream applications.

## Keep these layers separate

1. **WebAuthn/FIDO2 at the identity provider:** the browser and YubiKey authenticate the human to Pocket ID.
2. **OIDC at a relying application:** the application delegates login to the identity provider and enforces its own user/role authorization.
3. **Reverse-proxy authentication:** an alternative front-door enforcement strategy for compatible services; it does not necessarily replace application authorization.

Enrolling a YubiKey at Pocket ID does **not** automatically secure every self-hosted service.

## Resume

Inspect the actual Pocket ID deployment and target application capabilities; identify which applications support OIDC or require a proxy approach. Independently test Main, Spare and recovery before removing working passwords.
