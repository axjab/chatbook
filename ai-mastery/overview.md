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

## Infrastructure decisions and current status

### Git hosting — Forgejo

**Reported DONE (2026-11-26):** One Forgejo instance is deployed on **Majula**, with the public-facing name `git.alj.cx`. It provides Git repository management and agent-facing access while retaining native Git over SSH for the owner.

Use **one Forgejo instance** for multiple repositories. Avoid externally modifying Forgejo's managed bare-repository storage. Forgejo is the source of truth for versioned code and documentation.

### File exchange — SFTPGo

**CURRENT / IN PROGRESS (2026-11-26):** The owner selected **SFTPGo** for private per-agent storage, explicitly shared folders, file permissions and quotas, without granting agents unrestricted shell access.

- Access mechanisms include **SFTP/SSH**, a **WebAdmin**, **WebClient**, **REST API**, and optional **WebDAV**.
- The proposed Docker deployment shares the existing Caddy network; Caddy provides HTTPS for the WebAdmin/WebClient/API; SFTP listens on a trusted private-network interface.
- WebDAV is not confirmed enabled. A Docker `expose` declaration alone does not start it.
- The temporary service identity is **FTX — File Transfer Exchange**, with provisional hostname `ftx.alj.cx`. Rename later if a better identity emerges.
- The supplied Compose file and Caddy configuration are designs supplied/discussed in chat, **not verified proof of a running service**.

Read [SFTPGo 2.7 Getting Started notes](sftpgo-getting-started.md) for the complete supplied Feature overview chapter: initial setup, users, storage backends, ACLs, virtual folders, quota behavior, groups, config, databases, FTP and WebDAV.

### Host and execution context

Majula is the initial home-server host for Git and file storage. Reported resource snapshot (2026-11-26): Intel Core i5-4590T, four logical CPUs; about 7.7 GiB RAM and 3.4 GiB available; 87 GB free on root and 2.6 TB free on separate SSD.

Other hosts: Old Firelink Shrine is a deprecated VPS; replacement Firelink Shrine is planned; a dedicated agent VPS is a possible later project. No new VPS is required for initial Git/file hosting. Agent execution host remains **undecided** and should be isolated from trusted infrastructure.

## Principles and distinctions

- An **agent** is software that uses a model, a harness/loop and tools. It can run interactively or in the background.
- A **workspace** holds working files; **persistent memory** is separately implemented storage, not automatically inherent to an agent.
- **MCP** allows compatible clients to interact with exposed tools; merely running an MCP server does not automatically connect ChatGPT.
- Aim for provider-independent, least-privilege access to Git, files and other services; isolate future agent execution from trusted infrastructure.
- **Version-controlled material** belongs in Forgejo, while larger or unversioned artifacts can live in SFTPGo; backup and synchronization are separate questions.
- Keep credentials, sensitive host specifics and private network details out of this public Chatbook.

## Roadmap

1. **Current:** Complete the SFTPGo deployment and verify basic WebClient/SFTP access with a single user.
2. Configure dedicated private and shared virtual folders, then test permissions using distinct user identities.
3. Verify supported ChatGPT/other-agent integration paths to Forgejo and SFTPGo, with restricted credentials.
4. Choose an isolated agent runtime and execution host; run a supervised test workflow against a test repository.
5. Begin structured AI learning and increasingly sophisticated experiments.

Resume from [current tasks](ai-mastery.todo.md).
