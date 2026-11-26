---
title: AI mastery tasks
description: "Current and upcoming project milestones"
type: todo
tags: [ai, infrastructure, tasks]
created: 2026-11-26
updated: 2026-11-26
---

# AI mastery tasks

## CURRENT

- [ ] **Integrate Pocket ID passkey authentication with SFTPGo WebClient**, preserving local administrator recovery.

## NEXT

- [ ] **Configure SFTPGo virtual folders** for independent private user storage and explicitly shared folders with tested permissions.
- [ ] Enable agent access to self-hosted infrastructure (Forgejo and SFTPGo) through supported integrations and restricted credentials.
- [ ] Select an isolated agent runtime and execution host.
- [ ] Run a supervised agent workflow against a test repository.
- [ ] Begin structured AI/LLM/agent lessons and experiments.

## DONE

- [x] Deploy Forgejo on Majula (`git.alj.cx`).

## Notes

The owner selected **only two SFTPGo features for active exploration** on 2026-11-26: **Pocket ID OpenID Connect/passkey web authentication** and **virtual folders**. Other ideas from broad feature research are out of scope, not pending tasks. See [SFTPGo focused findings and handoff](sftpgo-pocketid-virtual-folders.md). SFTPGo deployment completion and first-user setup have not been independently verified; confirm them before OIDC changes. Keep tasks at milestone level. Never publish secrets or private network details.
