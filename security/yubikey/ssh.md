---
title: SSH over Tailnet and WAN
description: "Recorded SSH access preference, key-based authentication and optional hardware protection"
type: note
tags: [security, ssh, yubikey]
created: 2026-11-22
updated: 2026-11-23
---

# SSH over Tailnet and WAN

**Owner preference (2026-11-23):** Do not require YubiKeys for ordinary SSH. For WAN-facing OpenSSH, an ordinary asymmetric SSH key pair is acceptable. This is a design choice, **not** evidence that host SSH configuration has been changed or tested.

## Distinguish connectivity from authentication

- **Tailscale networking** provides a private connectivity fabric; it does not automatically replace ordinary OpenSSH login authentication.
- **Tailscale SSH**, when specifically enabled and configured with the appropriate tailnet policies, has its own identity and authorization flow.
- **Ordinary OpenSSH over Tailnet or WAN** still authenticates the remote user using its configured SSH mechanisms.

## WAN OpenSSH recommendation

Use an Ed25519 public/private key pair for interactive administration; authorize its public key for the intended remote account. Prefer encrypting the private key with a strong passphrase and caching it in `ssh-agent`. A passphrase on the **local SSH key** is different from a remote Linux password; a remote login can still be passwordless.

After confirming working access and a tested recovery path, consider disabling SSH password and keyboard-interactive login and direct root login for exposed hosts. Never remove the only working access path before successful testing. Use narrowly scoped separate credentials for unattended automation where applicable.

A hardware-backed OpenSSH `ed25519-sk` identity is **optional**, potentially useful for more sensitive interactive access. Touch requirements can conflict with unattended orchestration. GPG-agent/OpenPGP SSH is another option but is not selected for normal SSH.

## Host account naming

The owner wants **one short lowercase username for the primary human administrator account on owned servers and desktops**. The exact name and per-host account mappings are intentionally kept out of this public repository. Distinct service accounts remain separate. Migration of existing accounts requires an ownership, services, SSH, home-directory and recovery audit.

## Resume

1. Inventory existing SSH identities, enabled server authentication methods, and Tailscale SSH versus ordinary OpenSSH.
2. Verify WAN SSH public-key authentication without changing live recovery settings.
3. Prepare and test rollbacks before turning off any password-based SSH access.
4. Make the selected primary-account naming consistent only through planned host-by-host migrations.

## References

- [OpenSSH sshd_config manual](https://man.openbsd.org/sshd_config)
- [OpenSSH ssh-keygen manual](https://man.openbsd.org/ssh-keygen)
- [Yubico pam-u2f](https://developers.yubico.com/pam-u2f/)
