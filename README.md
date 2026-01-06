# COIOTE TOTP

`Coiote TOTP` is a **PAM-native, offline TOTP (Time-based One-Time Password)**
multi-factor authentication solution for Linux systems.

It provides a **local second authentication factor** for services such as
**SSH, sudo, and console login**, without relying on cloud services,
external MFA providers, or background daemons.

Designed for **security-first, auditable, and controlled environments**.

---

##  What is Coiote TOTP

- Local MFA based on **TOTP (RFC 6238)**
- Native integration with **Linux PAM**
- **100% offline**
- Encrypted secrets at rest
- Built-in rate limiting and lockout protection
- Single-use backup recovery codes
- Full audit logging via journald
- Explicit break-glass recovery (no silent resets)

Compatible with:
- Google Authenticator
- 1Password
- Authy (standard TOTP mode)

---

## Supported Systems

- Debian
- Ubuntu
- Rocky Linux
- AlmaLinux
- RHEL

Architecture:
- `amd64 / x86_64`

---

## Installation (Packages)

Official packages are distributed via **GitHub Releases**.

### Debian / Ubuntu

```bash
sudo dpkg -i coiote-totp_<version>_amd64.deb
```

### Rocky / RHEL / Alma

```bash
sudo rpm -ivh coiote-totp-<version>-1.x86_64.rpm
```

Installation does **NOT** enable MFA automatically
and does **NOT** modify PAM configuration.

---

## Recommended: Verify Package Integrity

### Import the public signing key

```bash
gpg --import coiote-totp.gpg
```

### Verify checksums

```bash
gpg --verify SHA256SUMS.asc
sha256sum -c SHA256SUMS
```

### Verify package signature

#### Debian / Ubuntu

```bash
dpkg-sig --verify coiote-totp_<version>_amd64.deb
```

Expected output:
```
GOODSIG
```

#### RPM-based systems

```bash
sudo rpm --import coiote-totp.gpg
rpm --checksig pam-totp-ng-<version>-1.x86_64.rpm
```

Expected output:
```
pgp md5 OK
or 
digests signatures OK
```

---

## Step 1 — Create the Master Key (REQUIRED)

The master key encrypts **all user secrets at rest**.

```bash
sudo mkdir -p /etc/coiote-totp
sudo chmod 700 /etc/coiote-totp

sudo dd if=/dev/urandom of=/etc/coiote-totp/master.key bs=32 count=1
sudo chmod 400 /etc/coiote-totp/master.key
```

Losing this key invalidates all configured TOTP secrets.
Store it securely (offline backup recommended).

---

## Directory Layout

```text
/etc/coiote-totp/
 └── master.key

/var/lib/coiote-totp/
 └── <user>/
     ├── secret.enc
     ├── backup.enc
     ├── meta.json
     └── .locked
```

All files are readable **only by root**.

---

## Step 2 — Enable PAM Integration

### SSH (`/etc/pam.d/sshd`)

```text
auth sufficient pam_unix.so
auth required   coiote-totp.so
```

### sudo (`/etc/pam.d/sudo`)

```text
auth required coiote-totp.so
```

### Console login (`/etc/pam.d/login`)

```text
auth required coiote-totp.so
```

---

## Step 3 — Enroll a User

```bash
sudo pam-totpctl init <username>
```

During enrollment:
- a QR code is shown in the terminal
- configure your authenticator app
- backup codes are shown once only

---

## Authentication

- User logs in normally
- System prompts for a TOTP code
- Backup codes may be used once if needed

---

## Recovery (Break-Glass)

```bash
sudo coiote-ctl reset <username>
```

This invalidates all MFA data and requires re-enrollment.

---

## Auditing

```bash
journalctl | grep coiote-totp
```

No sensitive data is logged.

---

## License

MIT License

---

## Security Contact

delcio_cabanga@hotmail.com

---

## Philosophy

> No cloud.  
> No silent changes.  
> Real system security.

---

