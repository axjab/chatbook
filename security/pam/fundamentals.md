---
title: Linux PAM fundamentals and experiments
description: "How PAM services, modules, control flow, prompts and isolated tests work"
type: note
tags: [security, linux, pam, learning]
created: 2026-11-23
updated: 2026-11-23
---

# Linux PAM: what was learned

PAM (Pluggable Authentication Modules) is a framework for applications to *request* authentication, account checks, credential changes and session setup. It is configurable, but the application remains in control of when PAM runs and how it handles the results and messages.

## A PAM file is a service policy, not an executable

An executable does **not** activate /etc/pam.d/<name> merely because its filename matches. A PAM-aware program explicitly calls PAM and chooses a service name:

~~~text
application
  -> pam_start("pam-goblin", user, conversation_handler, ...)
  -> /etc/pam.d/pam-goblin
  -> pam_authenticate(...) or another PAM operation
  -> application receives status and optionally messages
  -> pam_end(...)
~~~

The service is normally named for the application (sudo, login, sshd), but this is a convention, not a requirement. Debian's common-auth, common-account and common-session files are shared rule sets included by other services, not executables. For example, sudo and sudo-i can have separate PAM files.

A PAM module such as pam_u2f.so or pam_echo.so is a compiled shared library loaded by PAM. The host application need not know the module's internal implementation.

## The four management groups

| Rule type | When applications invoke it | Example |
| --- | --- | --- |
| auth | Prove identity | Password, YubiKey touch |
| account | Decide if the account may use the service | Account expiry, time-of-day restriction |
| password | Change authentication credentials | Change a password |
| session | Set up or tear down access | Log opening/closing, limits, mount resources |

A PAM *session* is an application-managed lifecycle, not necessarily a terminal or SSH shell. Not every application invokes every group. A caller can use pam_authenticate(), pam_acct_mgmt(), pam_chauthtok(), pam_open_session() and pam_close_session() as appropriate.

## How rules control outcomes

Typical line:

~~~text
auth sufficient pam_u2f.so authfile=/path/to/mappings cue
~~~

- auth: management group.
- sufficient: success may finish this group's stack immediately if no earlier required module failed; failure is ignored and subsequent rules can run.
- pam_u2f.so: module to load.
- Remaining tokens: module-specific options.

Key controls:
- required: failure is remembered, but processing continues; final result fails.
- requisite: failure ends processing immediately.
- sufficient: success can short-circuit remaining rules; failure normally falls through.
- optional: mostly informational when other rules determine the result.

The order of lines matters. A test that simulates success (pam_debug.so auth=success) **does not verify identity** and must never be placed in production access paths.

## Output is customizable, but not guaranteed

PAM has a conversation interface: modules request prompts or send PAM_TEXT_INFO / PAM_ERROR_MSG to the calling application. The *application's* conversation handler decides what appears on the terminal or UI. This is why PAM can produce rich prompts and messages, but cannot force arbitrary output in every program.

**Observed:** pam_u2f's configured touch cue appeared under sudo, but an attempted pam_echo greeting did not. The owner's investigation implicated sudo calling pam_authenticate() with PAM_SILENT. The pam_echo manual explicitly says it can ignore messages when PAM_SILENT is set. The observation does not prove that every module's output is suppressed in every sudo version.

There is no PAM daemon to reload for ordinary /etc/pam.d edits. A new PAM transaction normally reads the updated service policy.

For module arguments, square brackets have TWO distinct uses:
1. In the *control* field, e.g. [success=ok default=bad].
2. Around a *module argument containing spaces*, e.g. [cue_prompt=Touch your key now] or [prompt=Insert your key].
Ordinary single-token arguments like file=/path/message do not need brackets. Earlier advice that brackets are always invalid around module options was incorrect.

ANSI colors or countdown animations are possible through a caller willing to display them, but PAM does not guarantee terminal handling; stdout from pam_exec is also controlled by the host application.

## Isolated practice lab

Use a disposable service such as /etc/pam.d/pam-goblin and invoke it with pamtester instead of editing sudo or common-auth. The pamtester utility triggers a named PAM service, without granting privileges merely by returning success.

~~~bash
pamtester -v pam-goblin "$USER" authenticate
pamtester -v pam-goblin "$USER" acct_mgmt
pamtester -v pam-goblin "$USER" open_session close_session
~~~

Read-only *with respect to the PAM configuration*: invoking a configured module may still write logs, run commands or alter other state. Keep experiment files separate from production authentication rules.

| Module | Lab idea | Lesson |
| --- | --- | --- |
| pam_debug.so | Compare auth=success and auth=auth_err | Return values and control flow |
| pam_echo.so | Show a file containing %u, %h, %s | PAM conversation, user/host/service interpolation |
| pam_succeed_if.so | Check a chosen username or group | Conditional checks are not identity proof |
| pam_listfile.so | Permit/deny names from a file | Whitelist and blacklist rules |
| pam_warn.so | Inspect system journal | Audit requests without granting access |
| pam_exec.so | Execute /usr/bin/hostname | External scripts and application-controlled stdout |
| pam_time.so | Test the account phase | Time-based restrictions, separate from auth |
| pam_echo.so in session | Call open_session and close_session | Lifecycle hooks |

Illustrative *isolated lab* configurations (use ONE at a time):

~~~text
auth required pam_debug.so auth=auth_err
~~~

~~~text
auth required pam_echo.so file=/etc/security/pam-goblin.txt
~~~

~~~text
auth required pam_exec.so stdout /usr/bin/hostname
~~~

For pam_echo, a message such as "Hello %u on %h via %s" expands the local username, host and service. For pam_exec, external programs may have side effects, receive untrusted environment variables, and execute under the caller's process identity unless configured otherwise. Do not treat arbitrary scripts or decorative prompts as security controls.

## Application development

- C: link libpam, supply a pam_conv handler; call pam_start, desired operations, pam_end.
- Bash: cannot call C PAM APIs directly; use a PAM-aware client such as pamtester or a separate native wrapper.
- Python: use PAM bindings or ctypes; high-level password bindings may not expose all conversation messages.
- Go: use a libpam binding (often cgo) and a conversation handler.
- Rust: use a PAM crate/binding and a conversation handler.

The crucial point is the service name and conversation callback, not the language.

## Safety rules for experiments

- Do not put simulated-success modules, alwaysok, or nouserok into sudo/login/sshd.
- Keep a working administrative recovery route before PAM edits.
- Test edits on an isolated PAM service; changes to common-auth affect many applications.
- No program becomes privileged just because a PAM test returns success.
- Prefer read-only module inspection and official manuals before altering authentication behavior.

## References

- [Linux-PAM configuration format](https://man7.org/linux/man-pages/man5/pam.conf.5.html)
- [pam_echo manual](https://man7.org/linux/man-pages/man8/pam_echo.8.html)
- [pam_exec manual](https://man7.org/linux/man-pages/man8/pam_exec.8.html)
- [Yubico pam_u2f manual](https://developers.yubico.com/pam-u2f/Manuals/pam_u2f.8.html)
- [YubiKey-backed sudo status and next steps](sudo-pam.md)
