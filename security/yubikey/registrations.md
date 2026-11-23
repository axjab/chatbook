---
title: "YubiKey registrations"
description: "Public summary of enrollment reports, tests and remaining verification"
type: note
tags: [security, yubikey, inventory]
created: 2026-11-22
updated: 2026-11-23
---

# YubiKey registrations and inventory

There are **two physical keys**, labeled **Main** and **Spare**. Each has a separate manually maintained record of device identity and usage in a **non-public remote inventory**. Do not copy serials, live OTPs, credential IDs, PINs, public authorization mappings or sensitive account details into this public page.

## Known status (reviewed 2026-11-23)

| Integration | Main | Spare | Evidence |
|---|---|---|---|
| Bitwarden | Successful Android authentication reported | Repeated Android authentication failures reported | Earlier "both enrolled" claim conflicts with practical test |
| Tailscale | Enrollment reported | Enrollment reported | A discoverable Tailscale passkey observed on an inserted key; both not independently verified |
| GitHub | Registration reported | Not verified | Do not assume both registered |
| Pocket ID | Localhost passkey test worked | Not verified | Production access not established |
| sudo / pam-u2f | **Discoverable credential listed; touch-based sudo reported working** | **Not yet registered for sudo** | Main output seen; no exhaustive negative-test record |
| SSH | Pending | Pending | Design not selected |
| OpenPGP / gopass | Pending | Pending | Existing software key needs inspection |
| Custom script challenge | Pending | Pending | Experimental only |

**Important unresolved conflict:** The user described both keys as configured for Bitwarden, but practical tests succeeded only with Main. Treat Spare as unverified/not working for that flow until independently retested.

## What the key can show

ykman fido credentials list lists **discoverable FIDO2 credentials** accessible through credential management; a PAM entry for Main was observed. Absence from this listing does **not** prove absence of a non-discoverable enrollment.

## Suggested private record fields

- Service / host and account.
- Physical key label.
- Protocol/application (FIDO2, OTP, OpenPGP, PIV, etc.).
- Relying-party ID where relevant.
- Discoverable / non-discoverable.
- Enrolled / tested / failed / revoked.
- Date of last successful authentication.
- Recovery method and relevant configuration location.

The complete inventory stays private and is maintained manually. Building a general CRUD application is **not** part of this authentication workstream.

## Next action

Finish recording Main sudo test outcomes and recovery, then enroll and test Spare independently. See [sudo handoff](sudo-pam.todo.md). For all other services, verify Main and Spare **separately**, distinguishing enrollment from actual successful authentication.
