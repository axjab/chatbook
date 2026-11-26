---
title: AI mastery tasks
description: "Current infrastructure work, next actions, and deferred learning"
type: todo
tags: [ai, infrastructure, tasks]
created: 2026-11-26
updated: 2026-11-26
---

# AI mastery tasks

## CURRENT — IN PROGRESS

- [ ] **Verify Forgejo on Majula** (owner reports web service live at `https://git.alj.cx`; independent checks pending).
  - [x] Bring up Forgejo behind Caddy at `https://git.alj.cx` (owner-reported).
  - [ ] Verify HTTPS API response and SSH connectivity, then native Git clone/push.
  - [ ] Verify native `git clone`, branch push, and Forgejo pull-request workflow.
  - [ ] Configure restricted credentials/permissions and backups; test restoration.

**Host choice:** Majula selected after resource inspection (4 logical CPUs, 3.4 GiB available RAM, 87 GB free root, 2.6 TB free SSD). Plan for one instance hosting many repositories; no CI runners at first. Agent hosting remains undecided.

## NEXT

- [ ] Decide whether OpenSSH SFTP is sufficient for shared files; justify SFTPGo only if additional management features are needed.
- [ ] Verify real integration options for ChatGPT and external agents with self-hosted Git/files and MCP.
- [ ] Design one isolated test workspace with minimal credentials and observable access.
- [ ] Compare local, VPS, and hybrid agent execution after infrastructure access requirements are known.

## LATER

- [ ] Build/test a supervised coding-agent workflow against a non-sensitive test repository.
- [ ] Develop the AI/LLM/agent curriculum; use Hugging Face and other resources.
- [ ] Revisit the temporary project name when a better one emerges.

## Handoff

Forgejo web deployment at `https://git.alj.cx` is owner-reported live; API, SSH, Git operations, persistence and backup remain unverified. No agent runtime or remote integration is deployed. Do not publish credentials or private host details. No secrets or private host access details belong in this public notebook.
