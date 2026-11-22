# Current status — Host Doctor and authentication

**Reviewed:** 2026-11-22. This is a snapshot, not a live monitor.

## Resume here: YubiKey-backed sudo

**Decided:** Use two physical YubiKeys (Main and Spare), with **discoverable FIDO2 credentials** and **touch-only authentication** for sudo. The user wants no Linux password and no mandatory FIDO PIN during routine sudo authentication. Registration may still require a FIDO PIN.

**Completed / reported:**
- Debian 13 test host has the pam-u2f tooling installed.
- A root-controlled central mapping file location was selected: **/etc/u2f_mappings**.
- An initial command, **pamu2fcfg -v**, generated a **non-discoverable** credential. Its printed mapping was not saved.
- No sudo PAM changes or central mapping file creation were reported in this latest implementation attempt.
- Decision made to use **-r / --resident** for subsequent registrations.

**Immediate next step (not executed in the available record):**
1. Identify/insert **Main** only.
2. Explain the precise persistent hardware effects, expected touch/PIN interaction, and recovery path.
3. Run **pamu2fcfg -r -v** once; this creates a *new discoverable credential* and prints mapping data, but does not configure sudo.
4. Inspect with **ykman fido credentials list**, preserve the resulting mapping securely, and only then continue with Spare.

Do not assume this command ran just because it appeared in a conversation.

See [sudo and PAM](security/sudo-pam.md) for the stepwise checklist and safety requirements.

## Other workstreams

| Area | Status | Next useful action |
|---|---|---|
| Host Doctor: YubiKey presence | **Verified repo state:** [PR #1](https://github.com/axjab/host-doctor/pull/1) was open and unmerged on 2026-11-22 | Test against actual connected/disconnected hardware; review/merge if accepted |
| Host Doctor: health checks | **Reported existing implementation** | Inspect current source before extending |
| YubiKey registration inventory | **Partial** | Verify Main and Spare per service; record test and recovery status |
| Bitwarden | **Conflicting reports:** both registered was reported, but repeated Android tests passed only with Main | Resolve Spare's actual enrollment through independent test |
| Tailscale | **Reported enrolled**; a discoverable credential was observed for its login domain | Verify each key independently |
| GitHub | **Reported registered** | Verify each key independently |
| Pocket ID | **Localhost passkey experiment succeeded** | Check real deployment and OIDC consumers |
| SSH via YubiKey | **Pending design** | Choose between OpenSSH FIDO security-key auth and GPG-agent route |
| GPG / gopass | **Pending design** | Inventory and back up existing key material before any migration |
| Arbitrary-script authentication | **Pending experiment** | Select genuine cryptographic challenge-response, not USB presence |

## Scope boundary

**Host Doctor** is a local host inspection/reporting utility. **Hosts of Embers** is a distinct trusted-host administration framework. Do not conflate a health check with trusted-host enrollment or remote governance.

## Before any configuration change

Inspect the live host first. Previous descriptions, examples, and unmerged PRs are not evidence of a deployed configuration.
