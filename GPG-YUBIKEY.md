---
title: Passphraseless GPG and gopass with YubiKey on Debian and Ubuntu
description: "Generate and back up an OpenPGP identity once, provision multiple cards, then decrypt on any host without a software private key"
type: note
tags: [security, gpg, openpgp, yubikey, gopass, cheatsheet, debian, ubuntu]
created: 2026-11-24
updated: 2026-11-24
---

# Passphraseless GPG — YubiKey OpenPGP

**Goal:** Decrypt GPG and \`gopass\` secrets using the **private encryption subkey held on a YubiKey**, not a local software private key. The software-key **passphrase** is no longer needed during decryption. A **YubiKey OpenPGP User PIN** can still be required after insertion; physical touch is required when the card's touch cache expires.

**Do the key-generation and card-provisioning section ONCE for an identity.** On every other Debian/Ubuntu host, do only **Host setup** below. Do not generate a new encryption identity independently on every host.

**Original Yubico documentation:** [Using Your YubiKey with OpenPGP](https://support.yubico.com/s/article/Using-Your-YubiKey-with-OpenPGP). The vendor's walkthrough illustrates RSA; the observed implementation used **Ed25519 primary + cv25519 encryption subkey**, not that RSA example. See also [GnuPG card documentation](https://www.gnupg.org/documentation/manuals/gnupg/gpg_002dcard.html).

## A. Once per OpenPGP identity — generate, back up, provision Main and Spare

### 1. Install tools on a trusted provisioning system

~~~bash
sudo apt update
sudo apt install gnupg scdaemon yubikey-manager pinentry-curses
gpg --version
ykman --version
~~~

**Effects:** Installs packages; no card or key mutations. For higher assurance, generate the private keys on a trusted **offline** system (e.g. a verified live Linux USB); this reduces exposure to network-connected malware. Protect the offline media and all backups.

### 2. Generate a software primary key and a dedicated encryption subkey

Run on the **trusted provisioning system**, without a card requirement:

~~~bash
gpg --quick-generate-key "Your Name <name@example.invalid>" ed25519 cert,sign 1y
gpg --list-secret-keys --fingerprint
# Copy the displayed PRIMARY fingerprint into the following variable:
PRIMARY_FPR='REPLACE_WITH_PRIMARY_FINGERPRINT'
gpg --quick-add-key "$PRIMARY_FPR" cv25519 encr 1y
gpg --list-secret-keys --keyid-format LONG
~~~

**Effects:** Generates private/software keys in this GnuPG home. You should have a primary \`[SC]\` and a \`cv25519\` \`[E]\` subkey. **If an encryption subkey is already present, do not add another blindly.** Review the list and select the correct \`[E]\` subkey in later steps.

### 3. Back up BEFORE using keytocard

~~~bash
# Replace /media/secure-backup with a mounted, secured/offline destination.
BACKUP_DIR=/media/secure-backup
gpg --armor --export-secret-keys "$PRIMARY_FPR" \
  > "$BACKUP_DIR/secret-key-backup.asc"
gpg --armor --export "$PRIMARY_FPR" \
  > "$BACKUP_DIR/public-key.asc"
~~~

**Effects:** Writes an extremely sensitive **secret-key backup** and a shareable public-key export. Encrypt/protect the backup, disconnect it from the host, keep an independent copy, and **test restoration separately**. Exporting after converting local keys to card stubs might not produce a recoverable private-key backup. Never publish either a private key or the actual recovery path in a public repository.

### 4. Provision EACH YubiKey (Main, then Spare)

Insert **only the card being configured**:

~~~bash
ykman openpgp info
ykman openpgp keys info DEC
ykman openpgp access change-pin
ykman openpgp access change-admin-pin
~~~

The PIN commands **persistently change credentials on this card**. Verify that the decryption slot is empty before transferring; **do not overwrite** a preexisting encryption key without deliberate backup/recovery planning. User/Admin OpenPGP PINs are **not** FIDO2 PINs.

While the provisioning GnuPG home still holds the **software** private key:

~~~bash
gpg --edit-key "$PRIMARY_FPR"
~~~

At the interactive \`gpg>\` prompt, select **only the encryption \`[E]\` subkey** (usually \`key 1\`; verify the displayed \`ssb*\`), then:

~~~text
key 1
keytocard
2
quit
~~~

Choose **Encryption slot 2**. When prompted to save local GnuPG changes, answer **No** and confirm quitting without saving. This keeps the software key available for provisioning the spare; **the transfer onto the YubiKey is already persistent** and is not undone by quitting.

~~~bash
ykman openpgp keys set-touch dec cached
ykman openpgp keys info DEC
~~~

**Effects:** Persistent decryption touch policy; typically **15-second cached touch**. Verify that the card's \`DEC\` fingerprint matches the intended encryption subkey. Repeat the same process for **Spare**. The two cards will then hold the same private encryption subkey, but each has independent PINs and physical identity.

If the software key was deleted before provisioning Spare, restore the **verified secret-key backup** in an **isolated trusted GnuPG home**, confirm \`ssb\` (not only \`ssb>\`), repeat \`keytocard\`, and remove/protect the temporary secret-key staging environment. Do not assume deleting ordinary filesystem files securely erases their contents.

**Do not use \`gpg --card-edit -> generate\`** for this two-card/same-key workflow: on-card-generated private keys are non-exportable and cannot be cloned to another card.

## B. On EVERY Debian/Ubuntu host — use the existing cards

### 5. Install GnuPG and import PUBLIC key only

~~~bash
sudo apt update
sudo apt install gnupg scdaemon pinentry-curses
# Optional tools (if packages are available in the distro's enabled repos):
sudo apt install yubikey-manager
gpg --import /path/to/public-key.asc
gpg --fingerprint
gpg --card-status
gpg --list-secret-keys --keyid-format LONG
~~~

**Effects:** Package installation and **local GnuPG public-key/card-stub metadata**; no new private key or hardware key is generated. The public key fingerprint must be checked against a trusted record before use. An \`ssb>\` entry indicates an OpenPGP-card-backed secret-subkey reference; \`sec#\` indicates the primary software secret is unavailable here.

Insert Main or Spare to decrypt; use \`gpg --card-status\` after switching if GnuPG retains a stale card reference. The private encryption key remains on hardware.

### 6. Prove hardware-only decryption

~~~bash
TEST_HOME="$(mktemp -d)"
chmod 700 "$TEST_HOME"
GNUPGHOME="$TEST_HOME" gpg --import /path/to/public-key.asc
GNUPGHOME="$TEST_HOME" gpg --card-status
GNUPGHOME="$TEST_HOME" gpg --list-secret-keys --keyid-format LONG
printf 'hardware-only test\n' |
  GNUPGHOME="$TEST_HOME" gpg --batch --yes --trust-model always \
    --encrypt --recipient "$PRIMARY_FPR" --output /tmp/yubikey-test.gpg
GNUPGHOME="$TEST_HOME" gpg --decrypt /tmp/yubikey-test.gpg
~~~

Expected plaintext: \`hardware-only test\`. **The isolated home imported only a public key**, so successful decryption proves card use rather than software-private-key fallback. The \`--trust-model always\` override is **only for this disposable test** after verifying the recipient fingerprint; do not use it as a global trust setting.

**Effects:** Temporary directory and disposable encrypted test file; card operation may change its transient PIN/touch verification state. No hardware credential installation. The temporary directory remains until explicitly removed; never put software private backups inside this test home.

Repeat decryption with **Main and Spare independently**.

### 7. Use an existing gopass vault

If the vault's \`.gpg-id\` / recipient configuration already targets **this same OpenPGP encryption identity**, no GnuPG software private-key import or new \`gopass init\` is necessary:

~~~bash
# Install gopass using your distro package or the official gopass release.
gopass version
gopass mounts
gopass NAME_OF_EXISTING_SECRET
~~~

**Effects:** Normally reads/decrypts the requested entry (some gopass commands copy it into clipboard). It **does not migrate recipients**. If the vault uses an **older/different GPG recipient**, do not delete its recovery keys: plan and verify an additive recipient migration first.

For a fresh, empty vault only, initialize it with the verified public key using \`gopass init "$PRIMARY_FPR"\`; **do not run init against an existing store** without understanding its effects.

**Observed 2026-11-24:** Main and Spare each decrypted a GPG test message, and an existing gopass secret was decrypted with a hardware-touch terminal notification. The user's last gopass secret test was not independently attributed to both cards.

## Recovery and troubleshooting

- **Wrong card / No secret key:** Import the right public key, insert a card containing the **matching encryption subkey**, and refresh with \`gpg --card-status\`. Do not generate a new key to fix a card mismatch.
- **PIN prompt:** The OpenPGP User PIN is distinct from the removed software-key passphrase. Do not repeatedly guess PINs; retries are limited.
- **No touch notification:** Wait beyond the card's touch cache and check [Touch-key notifications](TOUCH-NOTIFICATIONS.md); an isolated \`GNUPGHOME\` can use a different agent socket than the detector.
- **Lost both cards:** Recovery requires the previously protected **secret-key backup** and independent restore verification. A public-key-only export cannot recover private decryption.
- **No sudo service changes:** This guide does **not** enable FIDO2 sudo, SSH authentication, LUKS disk unlock, or passkeys. Those use separate applications/protocols.

## References

- [Original Yubico OpenPGP setup guide](https://support.yubico.com/s/article/Using-Your-YubiKey-with-OpenPGP)
- [Yubico OpenPGP applet technical manual](https://docs.yubico.com/hardware/yubikey/yk-tech-manual/yk5-apps-openpgp.html)
- [GnuPG smartcard guide](https://www.gnupg.org/documentation/manuals/gnupg/gpg_002dcard.html)
- [Chatbook GPG/gopass experiment and verified observations](security/yubikey/gpg-gopass.md)
