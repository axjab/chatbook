---
title: AI mastery
description: "Project objectives, infrastructure choices, and resumption context"
type: note
tags: [ai, agents, infrastructure]
created: 2026-11-26
updated: 2026-11-26
---

# AI mastery

**Name:** AI mastery (temporary); Kindred is another provisional candidate.

## Goal

Learn AI, LLMs, models, agents, tools, MCP, harnesses and related technologies through experiments. Ultimately understand and deploy unfamiliar projects such as those listed on [Trendshift](https://trendshift.io).

**First priority:** Provision provider-independent self-hosted tools for future agents.

## Current infrastructure

- **Forgejo on Majula:** Reported deployed at `git.alj.cx`; supplies version-controlled repositories with native Git over SSH.
- **SFTPGo on Majula:** Selected for file exchange without granting agents interactive shell access. The owner supplied a Docker Compose design with Caddy reverse proxy, HTTP interface on container port `8080`, and private-network SFTP on `2022`. Its actual live state and first-account setup must still be verified. **FTX — File Transfer Exchange** and `ftx.alj.cx` are provisional identifiers.
- **Agent execution environment:** Not yet selected; it should be isolated from trusted infrastructure.

## SFTPGo feature decisions (2026-11-26)

Only **Pocket ID passkey login via SFTPGo OpenID Connect** and **virtual folders** remain in the active investigation. Do not add other features to the plan merely because SFTPGo offers them.

- **Now:** Pocket ID authentication for WebClient, with existing administrator access preserved.
- **Then:** Private homes and explicitly shared virtual folders with verified permissions and quota accounting.

See the [focused SFTPGo research and checkpoint](sftpgo-pocketid-virtual-folders.md). The [complete supplied Getting Started guide notes](sftpgo-getting-started.md) are retained solely as a reference.

## Principles

- Forgejo stores version-controlled code and documents; SFTPGo handles file exchange and larger/unversioned artifacts.
- An agent requires a model, harness/loop and tools; it does not inherently have persistent memory or arbitrary filesystem access.
- MCP is a tool-interface protocol, not an automatic self-hosted connection or security boundary.
- Enforce least privilege, distinct identities and isolated agent runtimes.
- Keep credentials and private network details out of the public Chatbook.

Resume at [current tasks](ai-mastery.todo.md).
