---
title: AI mastery and agent infrastructure
description: "Objectives, concepts, decisions, and handoff for learning AI while building a provider-independent agent environment"
type: note
tags: [ai, agents, infrastructure, learning]
created: 2026-11-26
updated: 2026-11-26
---

# AI mastery and agent infrastructure

## Status and naming

- **Decided:** Current working project name is **AI mastery**. **Kindred** is also provisional, not an approved permanent name.
- **Decided:** Bring a demonstrably better project name to the owner's attention whenever one arises; do not silently rename the project.
- **Decided:** Prioritize agent tooling and integration before the full curriculum.
- **Pending:** The location of agent execution (VPS, local workstation, hybrid) is undecided; first understand effects on memory, access, and security.
- **Reported:** The owner has primarily used browser chat with GitHub connectivity and wants access to self-hosted Git and file servers.

## Ambition and success criteria

Build broad practical mastery of LLMs, AI models, inference, training, agents, tools, harnesses, MCP, memory, RAG, evaluation, safety, and related technologies. Learn to inspect, install, run, evaluate, and modify unfamiliar trending AI repositories (Trendshift is a discovery source; repository examples are not a fixed syllabus).

Literal knowledge of *all* AI is not a finite milestone. Practical mastery means independent comprehension, reproducible experiments, debugging, and responsible deployment of previously unfamiliar systems.

## Core distinctions

- **Model:** Supplies token predictions/reasoning capabilities; does not by itself operate SSH or a filesystem.
- **Agent:** Software process or workflow that uses a model, tools, and an execution loop to work toward goals. An agent may be interactive, a background job, a scheduled task, or a persistent service.
- **Harness/runtime:** Software supplying model calls, tool implementations, context handling, permission gates, execution and lifecycle management.
- **MCP:** An interoperability protocol for exposing tools/resources to compatible clients; running a server does not automatically give every chat client access to it.
- **Memory:** Distinguish model parameters (training-derived knowledge), session context (supplied messages), and external persistent storage (files/databases). Agents can run on one machine and use persistent memory hosted elsewhere.
- **Security boundary:** Runtime isolation, tool scope, credentials, network reachability, and approval rules all matter. Do not equate access to a repository with unrestricted access to its host.

## Proposed learning roadmap

0. Agent infrastructure: isolated workspace, SSH/Git/filesystem tools, scoped credentials, self-hosted integrations.
1. AI basics: machine learning, neural networks, tokens, embeddings, transformers, LLM limits.
2. Models and inference: architectures, local vs hosted, quantization, context, compute, evaluation.
3. Agents: tool calls, loops, MCP, skills, context and memory, approvals.
4. Agent engineering: orchestration, sandboxes, multi-agent patterns, logging, recovery and testing.
5. Advanced topics: RAG, vector stores, fine-tuning, multimodal, reinforcement learning, evaluations.
6. Repository literacy: independently evaluate and operate unfamiliar AI projects from Trendshift and other sources.

Every topic follows **concept → architecture → experiment → verification → documentation**. Phases can overlap.

## Initial infrastructure hypothesis — NOT a deployment decision

- Existing interest: Debian, native SSH, self-hosted Forgejo, filesystem services, and controlled remote access.
- Prefer a provider-independent tool surface to avoid locking all workflows to one hosted model or agent vendor.
- Candidate execution: isolated containers with least-privilege, per-agent credentials and a test repository.
- First proof: an agent can clone a dedicated non-sensitive repository, edit a file, run tests, push a branch, and open a pull request, all with traceable permissions and observable actions.
- **Unknown / to verify:** Which ChatGPT integration paths can reach a self-hosted MCP endpoint or self-hosted Git server in the user's actual product environment. Never promise that simply hosting MCP creates ChatGPT connectivity.
- Do not put secrets, exact private network details, credentials, or live security configurations in this public notebook.

## Session handoff

As of 2026-11-26, no agent host, integration, deployment, credential, or sandbox has been provisioned for this project. The next task is **agent fundamentals and an access-path comparison**, before any installation. See [next steps](ai-mastery.todo.md).
