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

- [ ] **Deploy SFTPGo on Majula** for confined private and shared agent file storage, with no shell access.

## NEXT

- [ ] Enable agent access to self-hosted infrastructure (Forgejo and SFTPGo) through supported integrations and restricted credentials.
- [ ] Select an isolated agent runtime and execution host.
- [ ] Run a supervised agent workflow against a test repository.
- [ ] Begin structured AI/LLM/agent lessons and experiments.

## DONE

- [x] Deploy Forgejo on Majula (`git.alj.cx`).

## Notes

SFTPGo selected for SFTP, WebDAV and HTTP-based file access, per-agent private storage and explicitly shared directories, with quotas and permissions. Forgejo remains the version-controlled source of truth for code and documentation. Keep tasks at milestone level, not installation/verification microsteps. Agent hosting remains undecided. This public notebook must not contain credentials or private network details.
