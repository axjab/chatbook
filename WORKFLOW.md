# Working agreements

These rules were established through repeated implementation and troubleshooting discussions.

## Technical changes

Use **Inspect → Explain → Implement → Test → Verify → Record**, one coherent step at a time.

Before suggesting a command, disclose:
- What it does and why.
- Whether it is read-only, performs cryptographic work, writes a file, changes hardware credential storage, changes system policy, or is destructive.
- Whether its effects persist and how to reverse them, when applicable.
- What output or independent test would prove success.

Answer direct yes/no questions **YES**, **NO**, or **IT DEPENDS** first. Keep operational replies concise. When learning the underlying technology is the goal, explain it accurately rather than hiding distinctions.

Prefer official Yubico, Debian, OpenSSH, and GnuPG documentation. Verify options and side effects. Do not invent procedures or introduce unsupported steps without approval. Do not replace a working recovery path before testing the new one.

Procedures should be manually reproducible on a fresh Debian host. No unsolicited automation or multi-step scripts.

## Repository boundaries

**Chatbook exception:** The user expressly authorized direct updates to **axjab/chatbook/master** without prior approval, provided **every push is reported** with a concise description and commit reference.

**Other projects:** This permission does **not** apply to other repositories. In particular, earlier Host Doctor instructions require feature branches and explicit approval to merge into master.

Honor human edits; inspect before replacing. Prefer updating a topic page over making a new duplicate.

## Public publishing policy — balanced

Allowed: general architecture, accepted choices, useful insights, reproducible non-secret commands, project names, public domains, and coarse progress state.

Do not publish: passwords, tokens, private keys, PINs, recovery codes, exact device identifiers, private-network addressing, or sensitive host-specific security configurations. Avoid detailed live account mappings. This public notebook is not the full operational inventory.

## Evidence labels

- **Verified:** directly observed output or inspected current repository state.
- **Reported:** user described an outcome; not independently reproduced in this session.
- **Decided:** a deliberate choice, whether or not implemented.
- **Proposed:** an option not yet approved or deployed.
- **Pending:** planned but not completed.
- **Unknown:** evidence is insufficient, contradictory, or possibly stale.

Do not silently promote Proposed or Decided to Verified.
