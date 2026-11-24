---
title: YubiKey touch notifications on Debian and Ubuntu
description: "Install and run a single user service for GPG and FIDO touch requests, with desktop and terminal notifications"
type: note
tags: [security, yubikey, notifications, systemd, gpg, fido2, cheatsheet, debian, ubuntu]
created: 2026-11-24
updated: 2026-11-24
---

# Touch-key notifications — desktop and terminal

**Goal:** Show a visible notification when GPG or FIDO is waiting for a physical YubiKey touch. **One user systemd service** runs the detector and forwards its events; supports desktop notifications and terminal broadcasts.

**Upstream:** [max-baz/yubikey-touch-detector](https://github.com/max-baz/yubikey-touch-detector) (formerly maximbaz). The upstream Go installation command still uses the module path `github.com/maximbaz/yubikey-touch-detector`. It detects `GPG_1`, `U2F_1`, `MAC_1` on operation start; suffix `_0` means wait ended. GPG touch detection is heuristic because the agent does not expose the card's actual touch state.

**Observed 2026-11-24:** A **user** service named `yubikey-touch.service` ran a custom helper, delivering a real terminal `GPG_1` notification while `gpg --decrypt` waited for touch; both Main and Spare subsequently decrypted. This page offers a **reproducible generic implementation** of that architecture, not an assertion that the owner's deployed helper exactly matches every line below.

## 1. Install detector dependencies and build

~~~bash
sudo apt update
sudo apt install golang-go libgpgme-dev pkg-config
go version                         # Upstream currently requires Go 1.26+
# If your distro Go is older, install a current official Go toolchain first.
GOPATH="$HOME/.go" go install github.com/maximbaz/yubikey-touch-detector@latest
test -x "$HOME/.go/bin/yubikey-touch-detector"
~~~

**Effects:** Installs APT packages; downloads/builds a Go binary at `~/.go/bin/yubikey-touch-detector`. No YubiKey credentials or GnuPG secrets are changed. An older apt-provided Go may automatically download a new toolchain, or may require a separate Go upgrade; verify `go version` and the build result. Alternatively use an upstream signed release appropriate to the host.

## 2. Install the helper into libexec

First inspect any existing `/usr/local/libexec/yubikey-touch-daemon` and `/etc/systemd/user/yubikey-touch.service`. **Back up any customized installation before replacing it.** Run the block only when setting up a fresh host or deliberately replacing the prior version.

~~~bash
sudo install -d -m 0755 /usr/local/libexec /etc/systemd/user

sudo tee /usr/local/libexec/yubikey-touch-daemon >/dev/null <<'SH'
#!/usr/bin/env bash
set -euo pipefail

detector="$HOME/.go/bin/yubikey-touch-detector"
if [[ ! -x "$detector" ]]; then
    printf 'Missing detector: %s\n' "$detector" >&2
    exit 127
fi

# Single process tree: systemd stops this wrapper and its detector.
"$detector" --notify --stdout |
while IFS= read -r event; do
    case "$event" in
        GPG_1|U2F_1|MAC_1)
            if command -v tty-notify >/dev/null 2>&1; then
                tty-notify -l info "Touch requested: $event" || :
            else
                # Portable fallback: notify terminals owned by this user.
                for tty in /dev/pts/[0-9]*; do
                    [[ -O "$tty" && -w "$tty" ]] || continue
                    printf '\n[Touch requested: %s]\n' "$event" >"$tty" || :
                done
            fi
            ;;
        GPG_0|U2F_0|MAC_0) ;;
        *) printf 'Unrecognized event: %s\n' "$event" >&2 ;;
    esac
done
SH

sudo chmod 0755 /usr/local/libexec/yubikey-touch-daemon
~~~

**Changes:** Root-owned executable `/usr/local/libexec/yubikey-touch-daemon`. The optional locally installed `tty-notify` produces the enhanced terminal presentation already used by the owner. **No `tty-notify` dependency is required** for the generic implementation: it otherwise writes a basic notice to writable PTYs owned by the current user. Desktop notification requires a working user notification service/session.

**Important previous bug:** If your detector binary lives in `~/.go/bin`, do **not** check for bare `yubikey-touch-detector` using an unrelated `PATH`. This helper checks the executable's **actual absolute location**. Change `detector=` if you install it elsewhere.

## 3. Install one systemd user unit

~~~bash
sudo tee /etc/systemd/user/yubikey-touch.service >/dev/null <<'UNIT'
[Unit]
Description=YubiKey touch notifications

[Service]
Type=simple
ExecStart=/usr/local/libexec/yubikey-touch-daemon
Restart=on-failure
RestartSec=2

[Install]
WantedBy=default.target
UNIT

systemctl --user daemon-reload
systemctl --user enable --now yubikey-touch.service
systemctl --user status yubikey-touch.service --no-pager
~~~

**Changes:** Persistent systemd user unit and enabled-state symlink. Starts the notifier for the current logged-in user; no hardware/PIN/key changes. **Do not run a second detector** simultaneously (including the upstream `yubikey-touch-detector.service` or a manual `-v --notify` process), because multiple instances can contend for the GPG agent proxy socket.

A systemd **user** service generally starts during user login; for headless hosts that must run without login, evaluate `loginctl enable-linger USER` separately (changes persistent user-manager lifecycle policy). The service uses `--notify` for desktop notifications; on headless-only systems, remove that flag and keep `--stdout` for terminal messages.

## 4. Verify the actual touch request

~~~bash
unset GNUPGHOME
systemctl --user restart yubikey-touch.service
systemctl --user is-active yubikey-touch.service
gpgconf --list-dirs agent-socket
journalctl --user -u yubikey-touch.service -n 30 --no-pager
~~~

Use a pre-created disposable GPG ciphertext encrypted to the card's existing public encryption key (see [GPG cheatsheet](GPG-YUBIKEY.md)). With an enrolled card inserted and **at least 20 seconds since the last touch**, run:

~~~bash
gpg --decrypt /tmp/yubikey-test.gpg
~~~

**Do not touch immediately.** Expect a desktop notice (when available) and a terminal notice (`Touch requested: GPG_1`); touch the YubiKey; confirm plaintext. Repeat after the touch cache expires and with the spare. Also test a FIDO PAM touch request separately if enabled.

**No notification despite working GPG decryption:** the touch might still be cached; a temporary `GNUPGHOME` may use a different `gpg-agent` socket from the detector; the user service may have stale GnuPG environment. Use normal `GNUPGHOME` and restart **only this user notification service** before deeper diagnosis. The detector proxy is socket-specific; notification absence is **not** proof of software-key decryption.

## 5. Stop / roll back

~~~bash
systemctl --user disable --now yubikey-touch.service
~~~

This immediately and persistently disables the notification unit without affecting GPG/PAM/card credentials. Remove **only** the helper and unit created in this procedure if retiring the installation, or restore prior saved versions. User-service disable does not uninstall the detector binary.

## Links

- [Upstream detector README: installation, user services, event protocol, GPG proxy](https://github.com/max-baz/yubikey-touch-detector)
- [Owner's previous touch-event investigation](security/yubikey/touch-notifications.md)
- [Passwordless sudo cheatsheet](SUDO-FIDO.md)
- [Passphraseless GPG/gopass cheatsheet](GPG-YUBIKEY.md)
