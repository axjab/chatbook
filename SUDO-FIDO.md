---
title: Passwordless sudo with YubiKey on Debian and Ubuntu
description: "End-to-end FIDO2 sudo setup using the maintained security components, with enrollment, PAM changes, tests and recovery"
type: note
tags: [security, sudo, pam, fido2, cheatsheet, debian, ubuntu]
created: 2026-11-24
updated: 2026-11-24
---

# Passwordless sudo — YubiKey FIDO2

**Goal:** `sudo` succeeds with an enrolled YubiKey **touch instead of a Linux password**. The normal Linux password remains a fallback. This is **not** `NOPASSWD` in sudoers. Repeat on **each host**; FIDO relying-party credentials are host-specific in this design.

**Source implementation:** [axjab/security](https://github.com/axjab/security) — specifically [fido-prompt](https://github.com/axjab/security/blob/master/fido-prompt), [fido-auth](https://github.com/axjab/security/blob/master/fido-auth), [fido](https://github.com/axjab/security/blob/master/fido), [fido-worker](https://github.com/axjab/security/blob/master/fido-worker), [installer](https://github.com/axjab/security/blob/master/install.sh) and [test matrix](https://github.com/axjab/security/blob/master/TESTING.md). Read the current code before deployment: this cheatsheet describes that revision and is not an assertion that every future revision is identical.

**Safety:** Keep a separately opened and **tested root recovery session** for the entire PAM edit. Back up the current files. Do not change `common-auth`, `login`, `sshd`, or `su`; do not assume a current remote session alone is recovery. This installation changes privileged authentication immediately, without a daemon restart.

## 1. Install dependencies

~~~bash
sudo apt update
sudo apt install git libpam-u2f pamu2fcfg usbutils fido2-tools pamtester python3
~~~

**Changes:** APT package database and installed packages. Check that both `pamtester` and `fido2-token` exist. On distributions with different package names, resolve the equivalent packages before continuing.

## 2. Enroll each YubiKey for this host

Work as the **target local Unix user**, not root; use a secure terminal. Make sure `hostname` is stable: the default relying party is `pam://<hostname>`. **Registration persists on the hardware**, consuming one discoverable FIDO credential per card. Registration may prompt for the **FIDO2 PIN**; normal touch-only sudo is designed not to ask for it.

~~~bash
hostname
fido2-token -L                    # read-only: confirm the inserted FIDO device
umask 077
pamu2fcfg -r -u "$USER" > "$HOME/u2f-main.map"
# Remove Main, insert Spare. If enrolling one key only, omit the next two lines.
pamu2fcfg -r -n > "$HOME/u2f-spare.fragment"
printf '%s:%s\n' "$(cat "$HOME/u2f-main.map")" \
  "$(cat "$HOME/u2f-spare.fragment")" > "$HOME/u2f-combined.map"
~~~

For **one card only**, use `$HOME/u2f-main.map` instead of the combined map. Each authorized Unix user needs their own mapping line. For further cards, append `:<credential-fragment>` to that user's **same line**, not a second line for the same user. `pamu2fcfg -n` prints a credential fragment without the username.

**Do not print, publish, or commit the mapping.** It contains public authorization material, not private card keys, but is locally sensitive operational configuration.

## 3. Install the authorization mapping

For a host **without an existing** `/etc/u2f_mappings`:

~~~bash
sudo install -o root -g root -m 0600 \
  "$HOME/u2f-combined.map" /etc/u2f_mappings
sudo stat -c '%U %G %a %n' /etc/u2f_mappings
~~~

For a **single card**, replace `u2f-combined.map` with `u2f-main.map`. If `/etc/u2f_mappings` **already exists, stop and merge** into a private backup rather than overwriting another user's credentials. Preserve one line per user.

**Changes:** Root-owned authorization file `/etc/u2f_mappings`; persistent and reversible using its backup. No further YubiKey changes.

## 4. Install all FOUR FIDO components

~~~bash
git clone https://github.com/axjab/security.git
cd security
# Inspect fido, fido-worker, fido-auth, fido-prompt, install.sh, TESTING.md.
sh -n install.sh fido-prompt
python3 -c 'import ast; ast.parse(open("fido-auth").read())'
sudo ./install.sh --apply
~~~

The installer verifies dependencies and the mapping, backs up any replaced component, and installs:

| Source | Installed destination | Purpose |
| --- | --- | --- |
| `fido-prompt` | `/usr/local/libexec/fido-prompt` | Key presence/countdown and terminal notices |
| `fido-auth` | `/usr/local/libexec/fido-auth` | Cancellable FIDO authentication supervisor |
| `fido` | `/etc/pam.d/fido` | PAM entry point imported by sudo |
| `fido-worker` | `/etc/pam.d/fido-worker` | Isolated `pam_u2f.so` verification service |

The installer **does not** modify `/etc/pam.d/sudo`, `/etc/u2f_mappings`, `common-auth`, or the YubiKey. The worker must **not** be included directly from sudo.

## 5. Edit the sudo PAM service — REQUIRED MANUAL STEP

~~~bash
sudo install -d -o root -g root -m 0700 /var/backups/fido-sudo
sudo cp -a /etc/pam.d/sudo /var/backups/fido-sudo/sudo.before-fido
sudoedit /etc/pam.d/sudo
~~~

Immediately **before** the existing `@include common-auth`, add this exact line:

~~~text
@include fido
@include common-auth
~~~

**Do not replace the entire file or remove any existing account/session directives.** Inspect `/etc/pam.d/sudo-i` separately if `sudo -i` uses another policy. New PAM transactions read the change immediately; no service reload.

## 6. Verify success AND fallback

~~~bash
sudo -k
sudo -v                            # Insert registered key; touch; no Linux password
sudo -k
sudo -v                            # No key: countdown then normal password fallback
~~~

Also test: **spare card**, unregistered key (must reject and offer password), incorrect Linux password (must reject), Ctrl+C during insertion and touch waits, no touch timeout, `sudo -i` if applicable, a new SSH terminal, and the independent root recovery method. Follow [TESTING.md](https://github.com/axjab/security/blob/master/TESTING.md).

**Caveat:** USB presence only identifies Yubico hardware; it does not authorize a credential. The custom helper uses a terminal and the separate `pamtester` worker; do not reuse this terminal-only policy unchanged for a graphical greeter or unattended process.

## Roll back

Using the **already open recovery root session**, restore the saved policy and, if necessary, the files from the installer-reported backup directory:

~~~bash
cp -a /var/backups/fido-sudo/sudo.before-fido /etc/pam.d/sudo
~~~

This restores the original sudo policy **if the backup was taken immediately before this procedure**. Remove/revert only components you actually installed; restore any pre-existing `/etc/u2f_mappings` rather than deleting other credentials. Existing FIDO card credentials persist until explicitly deleted on-device.

## References

- [Yubico pamu2fcfg manual](https://developers.yubico.com/pam-u2f/Manuals/pamu2fcfg.1.html)
- [Yubico pam_u2f manual](https://developers.yubico.com/pam-u2f/Manuals/pam_u2f.8.html)
- [Chatbook sudo/PAM findings](security/yubikey/sudo-pam.md)
- [Touch-key notifications cheatsheet](TOUCH-NOTIFICATIONS.md)

**Status:** The owner reported working FIDO sudo on a Debian host; the complete cross-distro and negative-case matrix is a **procedure to verify**, not a claim of validation on every Debian/Ubuntu release.
