---
title: AI mastery next steps
description: "Prioritized verified-next-action checklist for agent infrastructure and AI learning"
type: todo
tags: [ai, agents, infrastructure, checklist]
created: 2026-11-26
updated: 2026-11-26
---

# AI mastery — next steps

## Immediate

- [ ] **1. Explain agent lifecycle and boundaries:** interactive process vs background job vs daemon; distinguish model, agent, harness, tool, MCP client/server, context, and persistent memory. Use concrete examples and avoid vendor-specific assumptions.
- [ ] **2. Map the connectivity options:** differentiate ChatGPT's connected apps, external coding agents, SSH/Git tools, MCP, and self-hosted remote services. Verify available product capabilities and their limitations before choosing a solution.
- [ ] **3. Define one minimal experiment:** a non-sensitive test repository, allowed operations (read, edit, test, push branch, PR), observable logs, explicit approvals, and cleanup. Identify the exact agent software and how it receives credentials.
- [ ] **4. Compare execution locations** (local workstation, VPS, hybrid) for isolation, availability, memory persistence, host reachability, backups, cost, and recovery. Do not pick yet.
- [ ] **5. Select a small initial runtime/tool stack** after comparison, including authentication and access restrictions.
- [ ] **6. Execute and verify** the test-repository end-to-end workflow, one persistent/reversible change at a time.

## Subsequent

- [ ] Expand access from Git to a narrow, non-sensitive filesystem test.
- [ ] Document how memory/context is persisted, inspected, limited, and reset.
- [ ] Start structured AI foundations lessons, maintaining demonstrations and evaluation criteria.
- [ ] Periodically reassess the provisional project name; propose alternatives only when clearly better.

## Operating constraints

- Priority is the very next actionable step, not large speculative deployments.
- Before commands that modify files, services, credentials or host settings, explain effects, persistence, verification and rollback.
- Start with least privilege; no unrestricted trusted-host or production-repository access.
- Mark reported, proposed and verified outcomes distinctly.
- Chatbook is public: publish general patterns and safe examples, not private infrastructure information.
- Existing standing authorization permits routine public-note commits directly to `master`; notify the owner after each successful push.
