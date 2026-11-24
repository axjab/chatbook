---
title: YubiKey touch notifications and event hooks
description: "Verified YubiKey touch-event detection, terminal notifications, and systemd setup"
type: note
tags: [security, yubikey, openpgp, notifications, linux]
created: 2026-11-24
updated: 2026-11-24
---

# YubiKey touch notifications and command hooks

## Why this exists

OpenPGP decryption can pause silently while waiting for a YubiKey touch. GnuPG does not reliably display its own touch cue. The same need exists for FIDO/U2F and HMAC operations; wrapping individual `gpg` or `sudo` commands does not cover all callers.

**Verified on 2026-11-24:** [yubikey-touch-detector](https://github.com/maximbaz/yubikey-touch-detector) detected a real OpenPGP decryption touch request and emitted `GPG_1` when the request began, then `GPG_0` when it ended. With `-v --notify`, it also produced a desktop notification. A separate `socat`-based listener triggered an existing terminal-broadcast notification, visible in multiple terminal sessions during the same decryption. After touching the card, GPG decrypted successfully. This establishes a working **event-driven** proof of concept; the actual contents of the locally named listener script were not supplied or audited.

**Not yet verified:** A boot/login-persistent systemd deployment of the detector or custom listener. No claims are made about other device types, other hosts, or full PAM/FIDO coverage.

## Architecture

```text
GnuPG / FIDO / HMAC / supported SSH requests
                    |
            yubikey-touch-detector
                    |
     $XDG_RUNTIME_DIR/yubikey-touch-detector.socket
                    |
         separate subscriber/listener
                    |
       tty-notify / logger / NATS / other command
```

The detector watches supported operations and publishes events through a Unix-domain socket. For GPG it proxies the local `gpg-agent` socket and infers a pending hardware operation. Detection is **heuristic**, not a hardware-provided touch-status API. It does not need to perform extra private-key operations. The listener should be independent of both the detector and each application invoking the key.

## Event stream

The project's documented socket protocol emits **fixed-length five-byte messages** without newline separators:

| Event | Meaning |
| --- | --- |
| `GPG_1` | GPG operation began waiting for touch |
| `GPG_0` | GPG operation stopped waiting |
| `U2F_1` / `U2F_0` | FIDO/U2F touch state changed |
| `MAC_1` / `MAC_0` | HMAC touch state changed |

`_1` means touch needed; `_0` means the wait ended. Notifications should normally fire on `_1` only. The event identifies an operation class, **not** the credential, requesting application or successful authentication. The listener must never treat an event as proof of success.

## Reproduce the observed workflow

In one terminal, run the detector interactively:

```bash
yubikey-touch-detector -v --notify
```

In another, start a subscriber. This is a **reproducible example**, not a verbatim copy of the user's local script:

```bash
socat -u UNIX-CONNECT:"$XDG_RUNTIME_DIR/yubikey-touch-detector.socket" - |
while IFS= read -r -N 5 event; do
    case "$event" in
        GPG_1|U2F_1|MAC_1)
            logger -t yubikey-touch "Touch requested: $event"
            # Substitute a known local notification command here.
            ;;
    esac
done
```

Then initiate a GPG operation that requires touch. Observed: a terminal notification appeared while decryption was paused, and the operation completed when the YubiKey was touched. The example uses `logger` rather than assuming the local notification script's command-line interface.

**Effects:** The detector may temporarily proxy the user GPG-agent socket while running. The subscriber reads a per-user socket and runs the explicitly configured command; it does not change key material or PIN/touch policies. This is session-scoped until a persistent user service is enabled.

## Daemonizing — documented procedure, deployment pending

The upstream README documents systemd **user** service and socket modes. Use the service for an always-on detector; choose the socket mode only when on-demand startup is preferred. Do not enable both simply because both exist.

If the installed package provides the unit, first inspect it:

```bash
systemctl --user cat yubikey-touch-detector.service
```

The upstream README documents `$XDG_CONFIG_HOME/yubikey-touch-detector/service.conf` (typically `~/.config/yubikey-touch-detector/service.conf`) for optional environment configuration:

```text
YUBIKEY_TOUCH_DETECTOR_NOTIFY=true
```

To use the packaged service after stopping an interactive detector:

```bash
systemctl --user daemon-reload
systemctl --user enable --now yubikey-touch-detector.service
systemctl --user status yubikey-touch-detector.service
```

If the installation lacks a unit, create one only after inspecting the installed executable path. A custom event listener can have its **own** user service with `Restart=always` and `After=yubikey-touch-detector.service`; service creation and successful persistence still need confirmation.

**Security and reliability notes:** Use exact event-pattern matches, not `eval` or arbitrary data as shell code. Use a single detector instance when proxying GPG sockets; stop manual testing before daemonizing. Use a per-user service because the GPG agent and notification environment are user-scoped; a user service is not necessarily running before login. Review socket access and desktop-notification behavior if multiple users or sessions are involved.

## Related decisions and next steps

- **Verified:** Imported OpenPGP decryption subkey works on-card, and a cached touch policy permits another decryption within the 15-second window without an additional touch. See [GPG and gopass](gpg-gopass.md).
- **Decided:** Prefer one detector feeding multiple subscribers/notification transports, rather than modifying GPG or PAM directly.
- **Pending:** Inspect service unit, enable persistent detector, daemonize the listener, test after logout/login and card reinsertion, and decide whether to relay events over NATS or only local terminal/desktop notifications.
- **Pending:** Verify U2F, SSH and HMAC events separately; observed `GPG_1` alone does not prove these paths work.

## References

- [maximbaz/yubikey-touch-detector README](https://github.com/maximbaz/yubikey-touch-detector) — protocol, event semantics, GPG proxy implementation, configuration, systemd service and socket
- [GnuPG smartcard documentation](https://www.gnupg.org/documentation/manuals/gnupg/gpg_002dcard.html)
