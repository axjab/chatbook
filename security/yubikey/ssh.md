---
title: Hardware-backed SSH
description: "YubiKey-backed SSH options and unresolved decisions"
type: note
tags: [security, ssh, yubikey]
created: 2026-11-22
updated: 2026-11-23
---

# Hardware-backed SSH — design pending

**Goal:** Authenticate to trusted Linux hosts through possession of a physical YubiKey, preferably with a deliberate touch. Existing SSH access must remain usable while testing.

## Two different approaches

1. **OpenSSH FIDO security-key authentication** (such as ed25519-sk or ecdsa-sk): an SSH credential is generated for use with a FIDO authenticator. The hardware performs signing. Discoverable credentials are an option, but their behavior and tooling should be verified before enrollment.
2. **GPG-agent/OpenPGP-backed SSH**: expose an OpenPGP authentication key through gpg-agent's SSH-agent support. This uses a different YubiKey application and key-management workflow.

**No approach has been selected or configured** in the available project record.

## Decisions and safeguards

- Test Main and Spare independently; credentials do not automatically copy across physical keys.
- Record which remote account/host trusts which public credential in the private inventory.
- Keep a recovery login and at least one working SSH session until tests pass.
- Separate host authorization of a public SSH key from key creation on the local authenticator.
- Do not confuse Tailscale network access with SSH cryptographic authentication.

## Resume

Inspect current OpenSSH version, existing working identities, and supported FIDO operations. Compare OpenSSH -sk and GPG-agent architectures, then select one before creating credentials or changing authorized keys.
