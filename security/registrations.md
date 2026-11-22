# YubiKey registrations and inventory

There are **two physical keys**, labeled **Main** and **Spare**. Each has a separate manually maintained record of its device identity and usage in a **non-public remote inventory**. Do not copy serials, live OTPs, credential IDs, or sensitive account details into this public page.

## Known status (2026-11-22)

| Integration | Main | Spare | Evidence |
|---|---|---|---|
| Bitwarden | Successful Android authentication reported | Repeated Android authentication failures reported | Earlier “both enrolled” claim conflicts with actual test |
| Tailscale | Enrollment reported | Enrollment reported | A discoverable Tailscale passkey was observed on an inserted key; both not separately verified here |
| GitHub | Registration reported | Not verified | Do not assume both registered |
| Pocket ID | Localhost passkey test worked | Not verified | Production access not established |
| sudo / pam-u2f | New discoverable registration pending | Pending | No sudo PAM integration completed |
| SSH | Pending | Pending | Design not selected |
| OpenPGP / gopass | Pending | Pending | Existing software key needs inspection |
| Custom script challenge | Pending | Pending | Experimental only |

**Important unresolved conflict:** The user described both keys as configured for Bitwarden, but repeated practical tests succeeded only with Main. Treat Spare as **unverified/not working for that flow** until independently retested.

## What the key can show

The command **ykman fido credentials list** lists **discoverable FIDO2 credentials** accessible through credential management. A discoverable Tailscale login entry was observed. Absence from this listing does **not** prove absence of a non-discoverable enrollment.

## Suggested record fields (non-secret operational inventory)

- Service / host and account.
- Physical key label.
- Protocol/application (FIDO2, OTP, OpenPGP, PIV, etc.).
- Relying-party ID where relevant.
- Discoverable / non-discoverable.
- Enrolled / tested / failed / revoked.
- Date of last successful authentication.
- Recovery method and relevant configuration location.

The remote inventory is maintained manually. Building a general CRUD application is **not** part of this authentication workstream.

## Next action

For each service, verify Main and Spare **separately**, distinguishing “account UI says enrolled” from “authentication actually passed.” Preserve recovery access during enrollment.
