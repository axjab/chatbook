---
title: SFTPGo Pocket ID authentication and virtual folders
description: "Selected SFTPGo features, verified documentation, decisions and the next-session checkpoint"
type: note
tags: [ai, sftpgo, pocket-id, oidc, virtual-folders]
created: 2026-11-26
updated: 2026-11-26
---

# SFTPGo — Pocket ID and virtual folders

**Decision (2026-11-26):** Of the SFTPGo feature exploration, retain **only Pocket ID authentication and virtual folders** as active topics. Other features are not part of the present work. The earlier [Getting Started notes](sftpgo-getting-started.md) remain as source-reference documentation, not as an implementation backlog.

## Context and status

- SFTPGo 2.7 is the selected file service for the AI mastery project; **FTX — File Transfer Exchange** and `ftx.alj.cx` are provisional names.
- Deployment uses Docker Compose on Majula and a Caddy reverse proxy; WebAdmin/WebClient/API share container HTTP port `8080`. SFTP uses SSH on container port `2022`, restricted at the host network boundary. Exact private network details are intentionally omitted from this public notebook.
- The latest owner-supplied Compose had no active OIDC configuration. Do **not** assume Pocket ID is already deployed, integrated, or reachable from the SFTPGo container.
- **Current milestone: integrate Pocket ID with SFTPGo WebClient using passkey login.** Virtual folders follow after the authentication experiment.

## 1. Pocket ID passkey authentication via OpenID Connect

### How it works

Pocket ID is a self-hosted OpenID Connect (OIDC) provider that supports **WebAuthn/passkey login**. SFTPGo 2.7 can act as an OIDC client for its **WebClient and WebAdmin**. On browser login, SFTPGo redirects the user to Pocket ID; Pocket ID authenticates the user (for example with a YubiKey passkey), then sends an OIDC response back to SFTPGo.

**Important boundary:** This enables passkey-based **browser** login. It does not turn the SFTP/SSH protocol or WebDAV into native passkey authentication.

SFTPGo's built-in web second factor is TOTP, not native passkey enrollment; the passkey is handled by Pocket ID.

### Exact relevant SFTPGo 2.7 settings

OIDC is configured per `httpd.bindings` entry, including when using Docker environment variables:

```dotenv
SFTPGO_HTTPD__BINDINGS__0__OIDC__CONFIG_URL=https://id.example.com
SFTPGO_HTTPD__BINDINGS__0__OIDC__CLIENT_ID=example-client-id
SFTPGO_HTTPD__BINDINGS__0__OIDC__CLIENT_SECRET_FILE=/run/secrets/oidc_client_secret
SFTPGO_HTTPD__BINDINGS__0__OIDC__REDIRECT_BASE_URL=https://files.example.com
SFTPGO_HTTPD__BINDINGS__0__OIDC__USERNAME_FIELD=preferred_username
```

**These are examples, not a ready-to-run configuration.** A secret file requires a matching read-only Docker secret/bind mount, correct permissions and a real OIDC client. Avoid publishing any actual client secret.

- `config_url` is the provider issuer URL; SFTPGo appends `/.well-known/openid-configuration` and fetches discovery data **at startup**. If unreachable, SFTPGo may refuse to start.
- `redirect_base_url` is the public SFTPGo URL; SFTPGo appends `/web/oidc/redirect` automatically. Register the complete callback URL in Pocket ID.
- `client_id` and `client_secret_file` identify the confidential OIDC client; the file takes precedence over `client_secret`.
- `username_field` selects a claim from the ID token to map **exactly** onto an existing SFTPGo account. Prefer an immutable, unique claim; `sub` has issuer-scoped uniqueness, while `preferred_username` is acceptable only if the provider guarantees it cannot be reused or reassigned. Verify Pocket ID's actual claims before deciding.
- SFTPGo does **not** automatically provision every OIDC user; accounts must already exist unless an explicit pre-login provisioning mechanism is configured.
- `role_field` allows administrator-role determination from a trusted ID-token claim; `implicit_roles=true` derives admin/user context from the selected login endpoint and matching SFTPGo account. Both require care to prevent unintended privilege.
- **Do not** enable `insecure_skip_signature_check` or debug token logging as shortcuts.

### Planned first experiment

1. **Read-only inspection:** Verify live SFTPGo version and Compose, working WebAdmin recovery login, Pocket ID existence/version/URL, and provider reachability from the container.
2. If Pocket ID is absent, deploy it separately with trusted HTTPS and register at least one passkey; preserve its administrator recovery process.
3. Register an OIDC **confidential** client in Pocket ID, using the exact SFTPGo callback `https://files.example.com/web/oidc/redirect` (adapt to actual domain). Restrict which identities may use the client.
4. Inspect the provider's real ID-token claims and choose a safe claim-to-account mapping. Create or confirm a **test SFTPGo WebClient user** with the exact mapped name.
5. Add only the necessary SFTPGo OIDC settings and secret mount. Back up the original Compose/configuration before changing them; preserve existing local WebAdmin access.
6. Recreate/restart SFTPGo deliberately, inspect startup logs, and test passkey login and logout in a separate browser session. Verify the login maps to the expected **ordinary user** and cannot access administrator functions.
7. Test fallback/recovery, failed login with an unauthorized identity, and the ability to revert the added environment variables and secret mount.

**Change disclosure:** Pocket ID enrollment writes passkey credentials to the authenticator/provider; creating an OIDC client persists identity-provider configuration and a secret; SFTPGo env/volume edits persist on the host and container restart changes service state. All must be discussed explicitly before execution.

### Open questions before writing live configuration

- Is Pocket ID deployed and reachable, and does it already contain the desired passkey identity?
- Which exact identity claim is uniquely and immutably assigned by the running Pocket ID version?
- Does the SFTPGo version and current Compose match the documented 2.7 configuration?
- What path will hold the client secret outside source control and public documentation?

**Official references:** [SFTPGo OIDC](https://docs.sftpgo.com/2.7/oidc/), [SFTPGo config — oidc](https://docs.sftpgo.com/2.7/config-file/#oidc), [Pocket ID — connect an app](https://pocket-id.org/docs/setup/connect-an-app), [Pocket ID introduction](https://pocket-id.org/docs/introduction).

## 2. Virtual folders

### Concept

A **virtual folder** maps a named storage location to a virtual path in a user's SFTPGo namespace. The backing store can be a local directory outside the user's home or a supported external storage provider. Create it using **WebAdmin → Folders**, then attach it to users with a chosen absolute `virtual_path`.

Example **proposed** layout:

```text
/
├── inbox/             # normal home subdirectory, not necessarily a virtual folder
└── shared/            # explicitly mounted shared virtual folder
```

For the planned agents: each user receives its own private home, and selected users mount the **same** dedicated shared folder at `/shared`; grant read-only to readers and appropriately limited write access to contributors.

### Permissions, quotas and correctness

- **SFTPGo virtual permissions** are enforced by the application and can differ per path; host/container filesystem permissions must also permit the actual operation.
- A virtual folder references a folder definition by **name** and maps it at `virtual_path`. Its `quota_size` counts bytes and `quota_files` counts files.
- Quota `0` means unlimited; `-1` means included in the user's quota.
- **Do not include a genuinely shared virtual folder in individual per-user quota accounting**: SFTPGo documents incorrect quota calculation in that case.
- Avoid backing paths that overlap each other, are nested, or sit beneath a mounting user's home. These can produce quota-accounting errors and unexpected alternate paths that bypass intended path-specific access rules.
- Keep the mapped path and its parent directories under trusted administrative control. Do not use host paths that the SFTPGo container was not intentionally permitted to access.
- For local folders, the same folder may be mounted by multiple users, but quota treatment and permissions need deliberate testing.

### Next experiment after Pocket ID

Create a separate, dedicated shared folder inside the SFTPGo storage mount; map it to two test users with different access rights. Verify each allowed and denied operation and inspect quota behavior. No production agent identities or folders are assumed to exist yet.

**Official references:** [SFTPGo virtual folders](https://docs.sftpgo.com/2.7/virtual-folders/), [Getting Started — virtual folders](https://docs.sftpgo.com/2.7/getting-started/), [SFTPGo config](https://docs.sftpgo.com/2.7/config-file/).

## Resumption point

**Next session:** Begin with **Pocket ID → SFTPGo OIDC** integration, starting from safe inspection of actual deployed components, then proceeding one validated change at a time. Do not start the virtual-folders experiment before finishing or explicitly postponing Pocket ID authentication. Do not broaden the scope to SSH hardware-key login, Event Manager, other file-transfer features or general service hardening unless necessary for this integration.
