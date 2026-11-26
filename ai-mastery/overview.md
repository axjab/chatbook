---
title: AI mastery
description: "Project objectives, infrastructure choices, and resumption context"
type: note
tags: [ai, agents, infrastructure]
created: 2026-11-26
updated: 2026-11-26
---

# AI mastery

**Name:** AI mastery (temporary). Kindred is another provisional candidate; propose a better name if one emerges.

## Goal

Learn AI, LLMs, models, agents, tools, MCP, harnesses and adjacent technologies through hands-on work. Eventually understand, deploy and modify unfamiliar AI projects, including those found on [Trendshift](https://trendshift.io).

**Priority:** Build self-hosted infrastructure that future agents can use **before** launching agents or following the broader AI curriculum.

## Current decision — Forgejo on Majula

**Selected:** Majula, the existing home server, for initial Forgejo deployment. **CURRENT / IN PROGRESS (owner installing)**. This is not yet a verified installation.

Reported capacity snapshot (2026-11-26):
- Intel Core i5-4590T, 4 logical CPUs; load average ~0.27.
- 7.7 GiB RAM; 3.4 GiB available; ~989 MiB swap used.
- 87 GB free on root filesystem; 2.6 TB free on separate SSD mount.
- Sufficient for an initial personal Forgejo service with modest activity and no CI runners.

Other hosts: Old Firelink Shrine is a deprecated VPS; replacement Firelink Shrine is planned; a dedicated agent VPS is a possible later project. No new VPS is required for initial Git hosting.

Use **one Forgejo instance** for multiple repositories, not one instance per repository. Preserve native Git over SSH for the owner and controlled access for agents; Forgejo's managed bare-repository storage should not be edited externally. Avoid unnecessarily duplicating Forgejo with SFTPGo; OpenSSH already supports SFTP. Agent execution host remains **undecided** and independent of Forgejo hosting.

## Principles and distinctions

- An **agent** is software that uses a model, a harness/loop and tools. It can run interactively or in the background.
- A **workspace** holds working files; **persistent memory** is separately implemented storage, not automatically inherent to an agent.
- **MCP** allows compatible clients to interact with exposed tools; merely running an MCP server does not automatically connect ChatGPT.
- Aim for provider-independent, least-privilege access to Git, files and other services; isolate future agent execution from trusted infrastructure.
- Keep credentials, sensitive host specifics and private network details out of this public Chatbook.

## Roadmap

1. **Now:** Forgejo on Majula; verify storage, access controls, native Git over SSH, and backup/recovery.
2. Evaluate whether basic SFTP suffices for file sharing, and what additional service is actually needed.
3. Verify available ChatGPT/other-agent connection paths to self-hosted services.
4. Choose agent runtime and execution location; implement a small isolated test workspace with narrow credentials.
5. Begin structured AI learning and increasingly sophisticated real-world experiments.

Resume from [current tasks](ai-mastery.todo.md).
