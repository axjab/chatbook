# Passwordless sudo with YubiKey and PAM

**Status:** In progress; no production sudo PAM change confirmed.

## Desired behavior and decisions

- Test environment: **Debian GNU/Linux 13 (Trixie)**.
- Normal sudo authentication: **touch a recognized YubiKey**, without Linux password or mandatory FIDO PIN.
- Both **Main** and **Spare** must authenticate independently.
- Use the Debian **pam-u2f** integration; **not** sudoers NOPASSWD.
- Use an authorization mapping controlled by root at **/etc/u2f_mappings**.
- **Decided:** create **discoverable / resident** FIDO credentials for this purpose.
- Preserve a working root/admin recovery route throughout testing.
- Do not change shared PAM rules unnecessarily: a sudo-only pilot should not affect login or SSH.

## What has actually happened

**Reported completed:** Install pam-u2f tooling; select mapping path and authentication policy; execute **pamu2fcfg -v** once.

That command performed an authenticator registration and printed a **non-discoverable** credential mapping. The mapping was not saved. It did **not** create **/etc/u2f_mappings**, alter PAM, or set up sudo.

**Pending:** A new discoverable credential for Main, then Spare.

## Resume — exactly one operation at a time

The documented command for the next Main registration is:

~~~bash
pamu2fcfg -r -v
~~~

Before executing, identify that **Main** is the inserted key and disclose:
- **Hardware:** creates one **new persistent discoverable FIDO2 credential**; does not transform the earlier non-discoverable credential.
- **Filesystem:** prints mapping data; it does not save the central mapping.
- **PAM/sudo:** unchanged.
- **PIN/touch:** FIDO PIN may be requested for resident registration; this is separate from sudo's desired authentication-time policy.
- **Recovery:** the new discoverable credential can be removed through FIDO credential management; any separately saved authorization mapping must be managed independently.

After registration, inspect the key using:

~~~bash
ykman fido credentials list
~~~

Do **not** proceed to Spare or edit PAM until the result has been reviewed.

## Checklist

- [x] Install pam-u2f registration and PAM tooling (reported).
- [x] Select root-controlled mapping location.
- [x] Decide on touch-only sudo authentication.
- [x] Decide to use discoverable credentials.
- [x] Understand the earlier non-discoverable registration.
- [ ] Register a discoverable credential on **Main**.
- [ ] Inspect and verify Main's new record.
- [ ] Register a discoverable credential on **Spare**.
- [ ] Inspect and verify Spare's new record.
- [ ] Preserve both mappings and create the central authorization file with secure ownership.
- [ ] Inspect effective **/etc/pam.d/sudo**, **sudo-i**, and applicable shared PAM files on the actual host.
- [ ] Test FIDO/PAM authentication in isolation.
- [ ] Apply a narrowly scoped sudo configuration.
- [ ] Prove successful passwordless sudo separately with Main and Spare.
- [ ] Negative tests: absent/wrong key, failed touch, and recovery.
- [ ] Document and verify a manual repeatable setup for other hosts.

## Safety notes

Registration origin and relying-party identity must match between credential creation and PAM authentication. An observed default used a **pam://** hostname-based origin. Do not silently change the scope to share credentials across hosts: that is a separate security decision.

Examples that change **common-auth** can impact many services. Do not paste an upstream PAM line into production without inspecting the real Debian stack, module order, and recovery paths first. Avoid bypass options such as **alwaysok** or **nouserok**.

## Official references

- [Yubico pam-u2f](https://developers.yubico.com/pam-u2f/)
- [pamu2fcfg manual](https://developers.yubico.com/pam-u2f/Manuals/pamu2fcfg.1.html)
- [Debian smartcard/YubiKey wiki](https://wiki.debian.org/Smartcards/YubiKey4) — may include legacy instructions; confirm Debian 13 applicability.
