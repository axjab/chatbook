---
title: SFTPGo 2.7 Getting Started feature overview
description: "Complete topical notes from the supplied SFTPGo 2.7 Getting Started Feature overview, with examples and deployment distinctions"
type: note
tags: [ai, infrastructure, sftpgo, file-transfer, documentation]
created: 2026-11-26
updated: 2026-11-26
---

# SFTPGo 2.7 — Getting Started: Feature overview

**Source:** [SFTPGo 2.7 documentation](https://docs.sftpgo.com/2.7/), section **Getting Started → Feature overview**. The full chapter was supplied for review on 2026-11-26.

**Scope:** This page documents **every section of the supplied Feature overview chapter** in reusable form. It does not claim to cover other pages linked by the documentation's navigation, such as the separate Docker, SSH, configuration-reference, or security-hardening chapters.

**Evidence:** The behavior and examples below are **documented by SFTPGo**, not tests of a particular server. The owner's SFTPGo deployment is **in progress**; nothing here proves account setup, storage permissions, or remote access is operational.

## 1. Initial configuration

SFTPGo must already be installed and running. Open the **WebAdmin** at `http://127.0.0.1:8080/web/admin` on an unproxied local installation, or use the appropriately secured reverse-proxy URL.

1. Create the first **administrator** account.
2. SFTPGo logs in the administrator and offers **two-factor authentication (2FA)** setup. The guide calls this optional.
3. Open **Status** to inspect active services.

The guide's default setup enables **SFTP on port 2022** and uses an embedded data provider (SQLite or Bolt, depending on OS/architecture). These are defaults, **not guarantees** for a deployment with overridden configuration.

Security warning from the guide: default settings balance convenience and safety, and are **not a production hardening checklist**. Understand all enabled services and settings; use TLS certificates for HTTP-based services. Terminate HTTPS at a trusted reverse proxy where appropriate.

## 2. Create a local user

In **WebAdmin → Users → +**:

1. Enter a **Username** and either a **Password** or **Public key** (these are the only always-required fields).
2. If no `users_base_dir` is configured, or the platform requires it, set an **absolute Home Dir** (Linux example: `/srv/sftpgo/data/example-user`; Windows example: `C:\sftpgo\data\example-user`).
3. Click **Save**.

SFTPGo attempts to create a missing home directory when that user logs in. The user's visible filesystem is confined to their home directory and any explicitly mapped virtual folders, subject to permissions.

**Filesystem permissions are independent of virtual permissions.** On Linux, the SFTPGo service runs as its own OS identity. For paths outside its normal data directory, the service identity must have suitable permissions on the target and every parent directory. In a container, verify permissions using the **effective container UID/GID and bind mounts**, not an assumed host account name. Do not automatically change ownership of unrelated host paths.

A standard SFTP client can test an account:

```sh
sftp -P 2022 example-user@127.0.0.1
```

Inside the SFTP prompt:

```text
sftp> ls
sftp> put file.txt
sftp> mkdir adir
sftp> cd adir
sftp> put file.txt
sftp> get file.txt
sftp> quit
```

These commands are **not read-only**: `put` uploads persistent files and `mkdir` creates a directory; `get` downloads a file to the local machine. The guide demonstrates successful listing, upload, download, and directory creation.

The **WebClient**, normally `http://127.0.0.1:8080/web/client`, lets end users browse/download files, share files externally, change credentials, and configure 2FA.

## 3. Other user storage backends

Creating a user with a different backend follows the same general user procedure; select the storage provider and supply its settings and credentials.

| Backend in guide | Core concept |
| --- | --- |
| AWS S3 / S3-compatible | Map a user to a bucket and optionally a restricted key prefix |
| Azure Blob Storage | Supply Azure storage configuration and credentials |
| Google Cloud Storage | Supply Google Cloud storage configuration and credentials |
| Another SFTP server | Use a remote SFTP service as the storage backend |

A **key prefix** limits a cloud-backed user or folder to a particular portion of a bucket. Credentials are sensitive and must never be committed to this public notebook.

### Local encrypted storage

SFTPGo also offers a local encrypted backend, called **Data At Rest Encryption** (the guide illustrates a `cryptfs` user). The creation procedure resembles a local user but additionally requires an **encryption passphrase**. The chapter does not describe passphrase backup/recovery or cryptographic internals; consult the dedicated encryption documentation before enabling it.

## 4. Virtual permissions

Permissions are assigned **per SFTPGo user** and can be overridden by **virtual directory path**:

- Set default permissions for `*` (all paths).
- Override a path such as `/read-only` with **list/download only**.
- Override another path such as `/subdir` with **no permissions**.

The guide demonstrates full access at the root, successful listing and downloading from `/read-only`, denied upload there, and denied listing under `/subdir`.

**Important distinction:** A directory being visible in a listing does **not** imply read or write permission inside it. SFTPGo's virtual permissions also cannot grant filesystem access that the container's underlying OS identity lacks.

## 5. Virtual folders and quotas

A **virtual folder** maps a SFTPGo virtual path (visible to the user) to a local filesystem directory **outside the home directory** or to another storage backend. There is no need for a virtual folder for a user's own home or ordinary subdirectories under it.

**WebAdmin → Folders → +** creates a folder definition:

- Local folder: specify a unique **Name** and filesystem **Absolute path**.
- Nonlocal folder: choose a backend and configure its credentials and settings.

To make one accessible, open **Users → Edit user**, associate it by **unique folder name**, and supply the virtual mount path (absolute Unix-style path even on Windows).

Example from the guide:

| Virtual path | Backend | Quota treatment |
| --- | --- | --- |
| `/vdirlocal` | Local folder | Usage counted in this user's quota; intended to be private |
| `/vdirminio` | MinIO / S3 compatible | Separate folder quota: **2 files and 100 MB** |

An upload to the MinIO folder fails once the number-of-files limit is reached, even if free bytes remain. The guide says either quota limit can stop new writes.

**Critical quota rule:** Do **not** share a folder between users while treating its usage as part of individual user quotas; the guide warns that doing so breaks accurate quota calculation. Use appropriate separate quota treatment for genuinely shared folders. A shared folder can be mounted for several users, potentially with different per-user quota limits.

## 6. Groups

SFTPGo groups simplify applying settings to multiple users and support three roles:

- **Primary group:** A user's primary inherited configuration.
- **Secondary groups:** Additional settings, including path-specific access restrictions.
- **Membership groups:** Group membership with distinct inheritance behavior.

A user may have one primary and multiple secondary/membership groups. **Detailed inheritance and precedence are covered in the separate Groups reference**, not fully specified by this chapter; do not assume they are interchangeable.

**SFTPGo groups are application constructs, not Linux/Windows OS groups.** You do not need to create or modify host system groups to use them.

### Worked group example from the guide

Requirements: per-user home directories, a **1 GB per-file upload limit**, a private S3 location, a common shared S3 location, and some users with read-only shared access while others may write.

1. Create folder **S3private** on an S3-compatible backend, with key prefix `users/%username%/`. The placeholder expands to the actual associated username, isolating each user's namespace.
2. Create **S3shared** pointing to a common key prefix `shared/` (no username placeholder).
3. Create a primary group **Primary**, set Home Dir to `/srv/sftpgo/data/%username%`, associate both folders, and set the maximum single-file upload to **1 GB**.
4. Create a group **SharedReadOnly** with read/list access only to the virtual path `/shared`.
5. Give all applicable users the **Primary** primary group; add **SharedReadOnly** as a secondary group for users who may only download/list the shared folder.
6. Test with a regular SFTP client, verifying permitted actions **and denied writes**.

The guide demonstrates the design; it does **not** establish the exact permissions of any user's existing deployment.

## 7. Simplify the administrator's Add user page

An administrator can simplify the extensive user-creation form, **automatically assigning groups** to new users and **hiding selected settings**. The guide creates an example admin named `simply`:

1. Open **Admins → +**.
2. Under **Groups for users**, assign **Primary** as primary and **SharedReadOnly** as secondary.
3. Under **User page preferences**, hide the form sections.
4. Sign in as this administrator: new users can then be provisioned mainly by setting their username and credentials, while the hidden settings come from the groups.

This is an administrative convenience with real access-control consequences: verify group inheritance before using a simplified user form.

## 8. Global configuration and overrides

The chapter explains two configuration methods:

1. Edit a configuration file (default **`sftpgo.json`**; TOML and YAML are also supported).
2. Set **environment variables**, then **restart SFTPGo**.

SFTPGo recommends environment variables for customizations because merging a modified config file with upstream defaults can be cumbersome during upgrades. Environment variables are particularly convenient in Docker.

Examples for packaged installations:
- Linux config directory: `/etc/sftpgo`
- Optional Linux environment file: `/etc/sftpgo/sftpgo.env`
- Additional files: `/etc/sftpgo/env.d/`
- Windows config directory: commonly `C:\ProgramData\SFTPGo`

These are package/platform examples, **not proof of paths inside an arbitrary Docker image**. Consult the version's Docker and configuration docs before overriding the container's startup paths.

### PostgreSQL data provider

The chapter demonstrates creating a `sftpgo` PostgreSQL database and a restricted database user, granting database privileges, then selecting the PostgreSQL driver using settings equivalent to:

```dotenv
SFTPGO_DATA_PROVIDER__DRIVER=postgresql
SFTPGO_DATA_PROVIDER__NAME=sftpgo
SFTPGO_DATA_PROVIDER__HOST=127.0.0.1
SFTPGO_DATA_PROVIDER__PORT=5432
SFTPGO_DATA_PROVIDER__USERNAME=sftpgo
SFTPGO_DATA_PROVIDER__PASSWORD=<password-from-secret-store>
```

The guide invokes `sftpgo initprovider -c /etc/sftpgo` under the service identity to **initialize or upgrade the schema**, arranges for SFTPGo to start after PostgreSQL, and restarts the service. Initialization **changes database state**; do not run against a live database without suitable backup and migration planning. The guide's `127.0.0.1` example assumes same-host networking, which is **not normally another container's address**.

### MySQL / MariaDB data provider

The guide shows creating a database using `utf8mb4`, creating/granting a dedicated database account, and changing the data provider:

```dotenv
SFTPGO_DATA_PROVIDER__DRIVER=mysql
SFTPGO_DATA_PROVIDER__NAME=sftpgo
SFTPGO_DATA_PROVIDER__HOST=127.0.0.1
SFTPGO_DATA_PROVIDER__PORT=3306
SFTPGO_DATA_PROVIDER__USERNAME=sftpgo
SFTPGO_DATA_PROVIDER__PASSWORD=<password-from-secret-store>
```

As with PostgreSQL, initialize the data provider, establish correct service ordering (MariaDB), and restart SFTPGo. Adapt host addresses and credentials to the real network and secret management.

### CockroachDB data provider

The chapter supplies a much longer **illustrative single-node installation**, including a versioned binary archive, required GEOS libraries, a service directory, CA/node/client TLS certificates, a systemd unit, and a `sftpgo` database. It then configures:

```dotenv
SFTPGO_DATA_PROVIDER__DRIVER=cockroachdb
SFTPGO_DATA_PROVIDER__NAME=sftpgo
SFTPGO_DATA_PROVIDER__HOST=localhost
SFTPGO_DATA_PROVIDER__PORT=26257
SFTPGO_DATA_PROVIDER__USERNAME=root
SFTPGO_DATA_PROVIDER__SSLMODE=3
SFTPGO_DATA_PROVIDER__ROOT_CERT=/etc/cockroach/certs/ca.crt
SFTPGO_DATA_PROVIDER__CLIENT_CERT=/etc/cockroach/certs/client.root.crt
SFTPGO_DATA_PROVIDER__CLIENT_KEY=/etc/cockroach/certs/client.root.key
```

It illustrates initializing/upgrading the SFTPGo schema, ordering its service after CockroachDB, and restarting. **Do not blindly execute the guide's old sample CockroachDB version or certificate-generation procedure** on a current installation; consult up-to-date CockroachDB and SFTPGo documentation. The chapter's example is not a requirement to deploy CockroachDB.

**For the initial single-instance deployment:** the supplied chapter's embedded SQLite/Bolt defaults avoid the work of maintaining a separate database. External providers are optional alternatives.

## 9. Optional FTP / FTPS service

SFTPGo does **not** require FTP for SFTP transfers. The guide enables the separate FTP listener with:

```dotenv
SFTPGO_FTPD__BINDINGS__0__PORT=2121
```

Then restart SFTPGo. Its config example includes a **passive port range 50000–50100**, passive IP overrides, NAT-related `force_passive_ip`, TLS settings, and security options. FTP clients often require passive ports to be reachable; behind NAT/firewalls the public IP and port rules must match.

The guide **recommends SFTP over FTP**, even if FTP is configured with TLS. Do not enable a legacy FTP listener merely to transfer files through SFTP.

## 10. Optional WebDAV service

WebDAV is an **HTTP-based remote file-management protocol**, independent of the SFTP/SSH listener. The guide enables a separate server using:

```dotenv
SFTPGO_WEBDAVD__BINDINGS__0__PORT=10080
```

Then restart SFTPGo. The sample `webdavd.bindings` includes listener address, `enable_https`, certificate and TLS settings, client authentication, URL prefix, and reverse-proxy/client-IP options.

The guide recommends **HTTPS** for WebDAV; HTTPS can be terminated by a carefully configured reverse proxy or enabled directly in SFTPGo. Publishing/exposing a Docker port does **not by itself start** a WebDAV listener.

## 11. Applying the guide to the AI mastery file service

**Decided/reported (2026-11-26):**
- **SFTPGo** is the selected transfer service for private per-agent files and explicitly shared storage, without interactive shell access; **Forgejo** remains for Git/versioned artifacts.
- The working service identity **FTX — File Transfer Exchange** and hostname `ftx.alj.cx` are **provisional**.
- The owner supplied a Docker Compose service using the Docker `caddy` network, reverse-proxied WebAdmin/WebClient/API on container port **8080**, and an SFTP port **2022** bound to a private trusted-network interface.
- The WebDAV port **10080** is listed in `expose`, but the WebDAV enablement settings are commented out; **its operation has not been confirmed**.

**Not yet verified:** Deployment completion, admin registration, user provisioning, home-directory write access, groups/folder shares, SFTP client login, or WebDAV activation.

**Practical application order:**
1. Finish and confirm the SFTPGo/Caddy deployment; establish the WebAdmin account and preferably 2FA.
2. Create a single test user with a dedicated home and password or SSH public key; verify upload, download, and directory creation.
3. Define a dedicated shared folder; mount it for selected users and explicitly test both allowed and denied actions.
4. Add primary/secondary group templates only when multiple agent accounts make repetition worthwhile.
5. Enable WebDAV only when a real client integration requires it; keep FTP disabled unless separately justified.

### Boundaries and open questions

- SFTPGo **user permissions** and the service process's **host/container filesystem permissions** are separate checks.
- Sharing and quota accounting require planning: do not mix a genuinely shared folder with per-user-only quota accounting.
- The source chapter teaches concepts; it is **not a complete security hardening or backup/recovery guide**.
- Test remote access without publishing private IPs or credential material in this public notebook.

## Reference

- [SFTPGo version 2.7 official documentation](https://docs.sftpgo.com/2.7/) — the supplied *Getting Started → Feature overview* chapter and its linked technical references.
- [AI mastery overview](overview.md) and [tasks](ai-mastery.todo.md).
