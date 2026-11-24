---
title: Auth CLI engineering prompt
description: "Complete engineering handoff for a minimal YubiKey authenticator wrapper, live TOTP display and read-only diagnostics"
type: note
tags: [security, yubikey, oath, fido2, openpgp, python, cli, proposed]
created: 2026-11-24
updated: 2026-11-24
---

# `auth` — master engineering prompt for Claude

**Status: DECIDED specification; NOT IMPLEMENTED.** This is a reusable handoff to a coding assistant. The source code belongs in a separate software repository, **not** in Chatbook. Public examples are fictional; never add actual device serial numbers, credentials, secrets or account inventories to this file.

## Role and objective

You are a senior Linux CLI engineer experienced with Python, secure subprocesses, YubiKey applications, terminal interfaces, and testing. **Implement, test and document** `auth`: a small, polished CLI that wraps `ykman` for everyday use. Do not reimplement OATH algorithms, FIDO, OpenPGP or cryptography. The supported surface is deliberately small; this is **not** a replacement for all of `ykman`.

Target Debian 13, compatible Ubuntu systems and normal Unix terminal usage. Prefer a single Python 3 executable using the standard library, minimal dependencies (runtime `ykman`), no root, no daemon, no storage/database and no persistent code/secret cache. No unnecessary UI framework. Implement software in its own repository, never in this knowledge-base repository.

**Before coding:** inspect the installed `ykman --version` and relevant `--help` output. The attached `ykman-reference.txt` from the originating session is the source for that user's OATH syntax; use official Yubico documentation for commands not included there. Do not guess command flags, prompts, output fields or transport behavior.

## User-facing commands

| Command | Function | Hardware effect |
| --- | --- | --- |
| `auth` | **Watch** current OATH TOTP codes, countdown and refresh at expiry | Read-only for TOTP |
| `auth <query>` | Show/watch one matching OATH account | Read-only for TOTP |
| `auth list` / `auth ls` | List OATH accounts without requesting a code | Read-only |
| `auth add [name]` | Prompt securely, enroll OATH credential | Persistent write |
| `auth rename <query> <new-name>` | Rename one existing OATH credential | Persistent write |
| `auth rm <query>` / `auth remove <query>` | Confirm and delete exactly one OATH credential | Persistent deletion |
| `auth fido` | View discoverable FIDO2 credential inventory | **Strictly read-only** |
| `auth pgp` | View OpenPGP key-slot metadata | **Strictly read-only** |
| `auth doctor` | Diagnose host/`ykman`/YubiKey state against approved baseline | **Strictly read-only** |
| `auth help` / `auth --help` | Concise help and examples | None |
| `auth --version` | Wrapper version | None |

`-h`/`--help` must work conventionally. Support a simple explicit device selector such as `--device SERIAL` if actually supported by the installed backend. Resolve name fragments only when **unique**, never by arbitrary first match. Good error messages should distinguish no match, ambiguous match and backend failure.

## 1. Live OATH view — the primary experience

**`auth` with no arguments enters a terminal watch mode. It must NOT clear the screen.** The watch displays current TOTP codes and seconds until expiration, then automatically refreshes codes at their actual boundaries.

Illustrative terminal appearance:

~~~text
$ auth

ACCOUNT                  CODE          EXPIRES
Example Service          012 345       18s
Other Service            573 891       18s

Next refresh: 18s  ███████████░░░░░░░░
2 TOTP accounts · YubiKey connected
~~~

Display is illustrative, not a claim about user accounts, codes or precise expiry.

**Non-negotiable terminal behavior:**

- **Never** invoke `clear`, `reset`, an alternate-screen buffer, terminal-wide erase (`CSI 2J`/`3J`), or any mechanism that destroys existing terminal output/scrollback. Leave existing shell output visible.
- For an interactive TTY, render once and then update **only `auth`'s own bounded lines in place**, using controlled cursor positioning / line replacement as necessary; do not append a full new table each second. Restore cursor visibility and terminal state on Ctrl+C, SIGTERM and normal exit. Output remains on screen after exit.
- Prefer a simple 1 Hz countdown. Obtain the current wall time from the host; use monotonic timing where appropriate to avoid busy-waiting and to tolerate clock corrections. Standard 30-second TOTP changes at Unix-time multiples of 30 (normally at :00 and :30), **not** 30 seconds after program startup.
- Fetch fresh codes **at the start and when a period actually changes**, not on every countdown tick. Handle time jumps, slow `ykman` calls and disconnection without presenting expired codes as valid.
- TOTP can use nonstandard periods: show per-account expiry/grouping rather than falsely giving a universal countdown. Only display expiry when the period is known/verified. Never hard-code 30 seconds for all credentials without reliable metadata.
- HOTP must **never** be generated automatically in a bulk view: doing so advances its counter. Show it as “HOTP — on demand” or exclude it with a clear notice. Touch-required credentials should be handled individually when needed, without repeatedly generating prompts.
- `auth <query>` should use similarly convenient interactive behavior for one TOTP account. Preserve Unix composability: **non-TTY stdout prints a single snapshot and exits**, with no animation, ANSI escapes, prompts or runaway process. Document its output contract.
- Ctrl+C exits promptly and cleanly (code 130 where conventional); avoid orphan subprocesses. Handle narrow terminals, long names, Unicode, no credentials, and removed/inserted YubiKeys.
- Preserve leading zeroes. Codes are secrets: do not log them, save to disk, send to telemetry or leave background refresh processes.
- The countdown animation must remain responsive but minimal, without flicker or excessive CPU usage. Respect `NO_COLOR`, dumb terminals and reduced terminal capabilities.

Backend: `ykman oath accounts code` and `ykman oath accounts list`. For one code use a single-match backend option where appropriate and supported.

## 2. OATH CRUD

**Add:** `auth add` or `auth add <name>` interactively requests issuer (optional), account name and Base32 secret, securely, without terminal echo or shell-history exposure. Defaults: TOTP, six digits, SHA1, 30 seconds. Permit concise options for nonstandard digits, algorithm, period or HOTP counter when actually supported. Detect duplicates and verify backend success. Identify the selected YubiKey.

**Absolute decision:** NEVER pass `--touch` or `-t` during new OATH enrollment. Newly added OATH credentials must not require physical touch for code generation. Existing touch-required accounts should still be viewable on demand; do not alter their policy.

The documented command is `ykman oath accounts add [OPTIONS] NAME [SECRET]`. Since exposing `SECRET` in a process argument is unsafe, investigate the installed version's behavior when the secret argument is omitted and `ykman` securely prompts for it. Prefer backend-controlled secure entry. Do not put secrets in argv, environment variables, temporary files, debug logs or command history. If this cannot safely be accomplished via the chosen wrapper architecture, **state the limitation** and use only a verified safe interactive path; do not pretend it is solved.

**Rename:** `auth rename <query> <new-name>` uses `ykman oath accounts rename`. Resolve exactly one account, preserve the current issuer unless intentionally changed, support `issuer:name` and detect firmware/version limitations (rename requires firmware 5.3+). Never simulate rename by deleting/recreating an unbacked-up secret.

**Remove:** `auth rm <query>` uses `ykman oath accounts delete`. Resolve exactly one account; display full identity and a clear irreversible-loss warning; ask for explicit confirmation (default No); confirm success only after a successful backend result. Noninteractive force mode, if added, must require an explicit flag and precise target.

**List:** `auth list`/`ls` enumerates registered OATH accounts without generating codes, triggering touch or advancing HOTP.

## 3. Read-only FIDO2 view

`auth fido` uses `ykman fido credentials list`; inspect `--csv` support for dependable parsing. Display relying-party/service identifier and account where available, with legible compact formatting.

**Critical limitation:** This enumerates **discoverable (resident) credentials only**, not every website/app where the key is registered. An empty list does **not** prove absence of FIDO use. Always label this distinction accurately.

Credential management may require FIDO2 PIN verification. Let `ykman` prompt securely if possible; never pass a PIN as an argument or repeatedly retry incorrect PINs, since attempts can decrement the device retry counter. Never invoke FIDO delete/reset/configuration commands. Clearly report unsupported management, PIN problems and device selection issues.

## 4. Read-only OpenPGP view

`auth pgp` uses `ykman openpgp info` and `ykman openpgp keys info` for signing (`sig`), decryption (`dec`/`enc` as supported by installed version), and authentication (`aut`) slots. Display presence, key type, fingerprint if available, generation/import origin and touch policy when the backend actually provides those fields.

Do not invent public-key identities or claim that an OpenPGP slot is a list of registered services. The host GnuPG keyring is not evidence that the private key is on this card. Distinguish an empty slot, inaccessible application, and unknown data.

Never invoke key import, generation, `set-touch`, PIN changes, resets or any other mutation.

## 5. `auth doctor` — read-only environment and device diagnosis

### Purpose and baseline

`auth doctor` checks whether the **local environment and currently connected YubiKey(s)** are ready for this workflow. It compares observed `ykman info` application states to the owner's **approved configuration**, reports actionable mismatches, and **never fixes anything automatically**.

This is the desired baseline for a compatible YubiKey 5 NFC-class device. The device's exact serial number is **not** a policy requirement and must not be hard-coded in this public document.

~~~text
Enabled USB interfaces: FIDO, CCID
NFC transport is enabled

Applications    USB        NFC
Yubico OTP      Disabled   Disabled
FIDO U2F        Disabled   Disabled
FIDO2           Enabled    Enabled
OATH            Enabled    Enabled
PIV             Disabled   Disabled
OpenPGP         Enabled    Enabled
YubiHSM Auth    Disabled   Disabled
~~~

Treat `ykman info` as the main source for these states; verify the installed format because some versions/models display different column layouts or report a capability as unavailable. Check both **USB and NFC** status when exposed. An available application reported as “Disabled” differs from an unsupported/not-available application. Do not infer NFC access from USB-only app status.

**Device type, serial, firmware and form factor are informative metadata, NOT fixed pass/fail values.** Show a device identifier locally when helpful for multi-key selection, but do not log or publish its exact serial. Do not require the sample model or specific firmware number. Report unsupported devices/capabilities clearly.

### Inspect the environment

Use read-only checks and verified backend commands as applicable:

- Is the `ykman` executable installed and callable? Show installed version and any minimum feature/version limitations.
- Are zero, one, or multiple YubiKeys detected? Use `ykman list --serials` or a documented equivalent as appropriate; select or **inspect each** detected device explicitly rather than silently choosing the first.
- Can `ykman info` read the device? Is there a USB access/permissions problem, missing FIDO/CCID interface, or disabled application?
- Report the enabled USB interfaces and NFC transport state, and compare each of the seven applications on USB/NFC against the baseline above.
- Is the host's clock synchronization status available via a read-only system mechanism (e.g. `timedatectl`)? Report “unknown” when unavailable; being marked synchronized is **not** proof of exact clock accuracy. Clock correctness matters for TOTP.
- If an OATH/OpenPGP communication issue arises, check relevant smartcard/CCID dependencies and permissions *only when actually needed*. Do not assume `pcscd` must always be running: `ykman` can use different connection paths.
- Clearly distinguish backend-not-installed, device-not-present, permissions error, unsupported hardware, user-selected intentional deviations, and actual policy mismatches.

**Do not** request FIDO PIN, OATH password, OpenPGP PIN or generate authentication codes merely to perform this diagnosis. `doctor` is a **non-interactive, read-only health report** by default. If a separate authentication-based check is later considered, it requires a separate explicit user action, not a hidden `doctor` side effect.

### Output

Use a compact, Fastfetch-style diagnostic report. Every failing item needs observed/expected values and a short, useful reason; no false green checkmarks when a field could not be read.

~~~text
$ auth doctor

ENVIRONMENT
  ykman             OK       installed and callable
  Device access     OK       YubiKey connected
  Clock sync        UNKNOWN  no reliable status available

DEVICE
  USB interfaces    OK       FIDO, CCID
  NFC               OK       enabled

APPLICATION         USB         NFC
  Yubico OTP        OK          OK
  FIDO U2F          OK          OK
  FIDO2             OK          OK
  OATH              OK          OK
  PIV               OK          OK
  OpenPGP           OK          OK
  YubiHSM Auth      OK          OK

Result: configuration matches the approved application policy.
~~~

This is fictional output, not evidence of a real diagnosis. On mismatch, explicitly include `expected` and `actual`; supply an informative **suggested manual investigation**, not an automatic fix. Use status categories such as OK / MISMATCH / UNKNOWN / UNSUPPORTED / ERROR, with deterministic exit status: success when checks pass, nonzero for genuine errors/mismatches, and documented policy for unknown checks.

When multiple devices are connected, inspect each one and show its own comparison, not a merged result that hides a difference. Never automatically choose a device for a write operation.

**Absolute read-only boundary:** `auth doctor` may execute read-only information/listing commands only. Never call `ykman config usb`, `config nfc`, `config mode`, `reset`, `keys set-touch`, any PIN-management operation, or any device/app setting change. No privilege escalation or installation by the doctor.

## 6. Shared quality and safety requirements

- Prefer Python 3 standard library (`argparse`, `subprocess`, `csv`, `shutil`, `time`, `signal`, etc.) and minimal, named functions. Avoid complex frameworks.
- Centralize all `ykman` invocations behind a mockable, well-tested adapter. Use argument arrays, never shell interpolation/`shell=True`. Enforce an allowlist of appropriate backend commands, particularly for read-only routes.
- Prevent sensitive PINs, Base32 secrets, OATH codes and device-specific registration data from entering logs, test snapshots or public documentation.
- No stored credentials, background agents or automatic synchronization. Enrollment on Main does **not** automatically enroll Spare; each key must be provisioned and tested independently.
- Make persistent writes to YubiKey hardware explicit before execution and explain whether reversible. Touch or PIN prompts are not evidence that a mutation happened.
- Handle missing backend, disconnection, multiple keys, disabled applications, unsupported firmware, cancellation, failed commands, empty credentials, ambiguous queries and unexpected output without traceback spam.
- Minimize unnecessary ykman subprocess invocations in the live view; ensure robust handling of terminal signals and partial output. Respect users who pipe `auth` into other commands.
- Color only where useful; support `NO_COLOR` and non-TTY output. Do not require a specific theme.
- Never add general `ykman` passthrough commands or expand the command surface beyond proven frequent needs.

## 7. Verification and tests

Provide automated tests (prefer `unittest` with fixtures grounded in real/documented `ykman` output) that do **not** require a connected YubiKey:

1. CLI routing, help/alias semantics and exit codes.
2. Live view: **no screen clear, no alternate screen, no full-table spam**, bounded in-place updates, countdown and code refresh at precise TOTP boundaries, differing periods, system clock jumps, SIGINT and non-TTY one-shot.
3. No repeated code-fetch per second; no HOTP counter advancement during watch or list; correct handling of touch-required accounts.
4. Secure input, never-enroll-with-`--touch`, add/rename/delete matching, confirmations, ambiguous queries and error feedback.
5. FIDO discoverable credential parsing and non-discoverable caveat; PIN handling without argument leaks.
6. OpenPGP empty/present slot parsing, metadata unavailable, version-specific `dec`/`enc` support.
7. `doctor` baseline comparison for **all seven applications and both transports**; missing/changed interface, disabled NFC, absent device, unexpected app state, unsupported app, permissions failure, inaccurate/incomplete backend output, multiple connected devices, unknown clock state, and clear expected/actual diagnostics.
8. Assert `auth doctor`, `auth fido` and `auth pgp` can never call a mutating `ykman` command. Assert unit tests never modify hardware.

Also provide an **optional, clearly labeled manual hardware checklist** for both Main and Spare. Before a test that writes to hardware, explicitly disclose the change, persistence, reversal and expected verification. Never claim tests passed without actually executing them.

## 8. Deliverables and done criteria

1. Working `auth` executable in a software repository (not Chatbook).
2. Focused tests and executed test results, with any untested hardware paths identified.
3. Concise README: installation, uninstallation, usage, exit-code behavior, supported OS and backend requirements.
4. A reproducible manual check on the real device(s), including the read-only doctor, only when available and authorized.

**Done:** `auth` immediately shows **live codes without clearing the screen**, `auth doctor` reliably and **read-only** checks the approved USB/NFC application matrix and host prerequisites, CRUD remains safe, `auth fido` and `auth pgp` are read-only, and the tool is demonstrably simpler than using raw `ykman` for daily tasks.

### Official command references

- [General `ykman` / `ykman info`](https://docs.yubico.com/software/yubikey/tools/ykman/Base_Commands.html)
- [OATH](https://docs.yubico.com/software/yubikey/tools/ykman/OATH_Commands.html)
- [FIDO2 credentials](https://docs.yubico.com/software/yubikey/tools/ykman/FIDO_Commands.html)
- [OpenPGP](https://docs.yubico.com/software/yubikey/tools/ykman/OpenPGP_Commands.html)
