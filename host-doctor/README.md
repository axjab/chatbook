---
title: "Host Doctor"
description: "Host health inspections and YubiKey USB presence checking"
type: note
tags: [host-doctor, linux, diagnostics]
created: 2026-11-22
updated: 2026-11-22
---

# Host Doctor

**Repository:** [axjab/host-doctor](https://github.com/axjab/host-doctor)  
**Status:** Earlier implementation discussed; current deployed version not inspected in this notebook.

## Purpose and boundaries

Host Doctor is a Unix-friendly local host diagnosis tool, intended to report host health and configuration in a format useful to humans and scripts. It is **not** the trusted-host control plane.

The earlier **hostdr** implementation used Bash and jq for readable and JSON output. Discussed checks included operating-system identity, networking/DNS/HTTPS, VPN and external connectivity, firewall/SSH posture, maintenance, systemd services and timers, and storage/mounts. Some checks contained host-specific assumptions: those should be inspected rather than generalized without evidence.

Persistent history, background collection, remote reporting and self-healing were discussed as future possibilities, **not confirmed features**.

## YubiKey USB presence check

A separate [PR #1](https://github.com/axjab/host-doctor/pull/1), branch **feat/yubikey-presence**, was opened for **check-yubikey.sh**.

The check attempts USB identification through **lsusb**, with a sysfs fallback, and reports either:

~~~text
YubiKey is inserted.
YubiKey is not inserted.
~~~

A prior no-match exit-status bug was identified and corrected during development. **As of 2026-11-22, PR #1 is open and unmerged** (GitHub checked). Testing on an actual connected/disconnected YubiKey is not confirmed.

**Security distinction:** This check only reports apparent device presence. USB identifiers are not cryptographic proof of a trusted authenticator. Do not use it to grant privileges.

## Relationship with Hosts of Embers

**Hosts of Embers** is a separate initiative for kindling and governing trusted, remotely manageable hosts. It may consume health information from Host Doctor later, but no such integration is established here.

## Next steps

- Inspect current repository source and actual diagnostic output before making architectural claims.
- Validate PR #1 on real hardware, including both presence and absence.
- Keep future Host Doctor work on branches until merge approval is given.
