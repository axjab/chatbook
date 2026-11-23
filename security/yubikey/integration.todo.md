---
title: Authentication and host identity next tasks
description: "Upcoming desktop, username, GPG and script authentication work after sudo"
type: todo
tags: [security, yubikey, ssh, gpg, hosts, tasks]
created: 2026-11-23
updated: 2026-11-23
---

# Next authentication and identity tasks

**Scope:** Follow-up work after the owner reported sudo/FIDO working on 2026-11-23. These are **decisions and requests**, not proof of deployment. The separate [sudo test checklist](sudo-pam.todo.md) and [registration inventory](registrations.md) may still need evidence-based reconciliation.

## Priorities

- [ ] **Dux/DankMaterialShell graphical sign-in:** Determine the installed Dux version and whether the target is the **greetd login screen**, the **DMS lock screen**, or both. Official Dux documentation supports U2F for each through different PAM policies. Inspect existing greetd, dankshell and dankshell-u2f services first. Test a key-only path with password recovery; do not blindly import the sudo-specific terminal helper.
- [ ] **Universal human username:** Adopt one short lowercase username for the primary human administrator across owned servers and desktops. The exact chosen name and host mappings are recorded privately, not in this public notebook. Keep service identities separate. Audit processes, ownership, home directories, sudoers and SSH before migrating an existing account.
- [ ] **Hardware decryption for gopass and encrypted files:** Inspect the existing secret key and hardware OpenPGP capabilities read-only. Test independent encrypted backups and recovery; decide on Main/Spare strategy, PIN/touch behavior and whether an existing encryption subkey can be imported onto both. Only then migrate. See [GPG/gopass](gpg-gopass.md).
- [ ] **Low-effort FIDO gates for Bash, Python and Go:** Prototype a single reusable key-only authentication interface with PAM or a verified challenge-response CLI; return nonzero on failure/cancellation. Show simple invocations in each language and test honest negative cases. A user-editable script is not itself a security boundary. See [script authentication](script-auth.md).
- [ ] **Ordinary SSH:** On WAN, deploy tested Ed25519 public-key authentication without requiring YubiKeys, preferably with local private-key passphrase protection. For Tailnet, distinguish Tailscale SSH from ordinary OpenSSH. Keep existing recovery until tested. See [SSH plan](ssh.md).
- [ ] **Enrollment audit:** Confirm each YubiKey independently for Bitwarden, Tailscale, GitHub and other services; reconcile outdated sudo Main/Spare outcomes without assuming unrecorded tests passed.

## Immediate read-only inspection

For Dux, inspect the installed version and existing PAM services before any changes. For GPG, inspect `gpg --list-secret-keys --keyid-format LONG` and `gpg --card-status` (the latter queries the connected card without changing its credentials). Do not paste private identifiers or encryption secrets into this public notebook.

## Recovery principles

No blanket modifications to `common-auth`, no untested greetd changes, no deletion of software GPG secrets, and no replacement of working SSH login before validation. Preserve independent recovery access.
