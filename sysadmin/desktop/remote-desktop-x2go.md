# X2Go Remote GUI Setup (AlmaLinux 9 / RHEL 9)

## Overview

This document describes how we configured X2Go remote GUI access on `eiclin07`.

The setup provides:
- Secure remote desktop access over SSH
- Lightweight XFCE desktop environment
- Linux and Windows client support

---

# Server Setup (`eiclin07`)

## 1. Install XFCE Desktop

Verify XFCE group exists:

```bash
dnf group list | grep -i xfce
```

Install XFCE:

```bash
dnf groupinstall "Xfce" -y
```

Verify installation:

```bash
rpm -qa | grep xfce4
```

---

## 2. Enable Required Repositories

Install EPEL:

```bash
dnf install epel-release -y
```

Enable CRB repository:

```bash
dnf config-manager --set-enabled crb
```

Verify repositories:

```bash
dnf repolist
```

Expected repositories include:
- appstream
- baseos
- epel
- crb

---

## 3. Install X2Go Server

Install packages:

```bash
dnf install x2goserver x2goserver-xsession -y
```

Verify installation:

```bash
rpm -qa | grep -i x2go
```

Expected packages include:
- x2goserver
- x2goserver-xsession
- x2goagent

---

## 4. Ensure SSH Service Is Running

X2Go uses SSH and does not have its own systemd service.

Verify SSH:

```bash
systemctl enable --now sshd
systemctl status sshd
```

---

# Linux Client Setup

## 1. Install X2Go Client

Enable EPEL if needed:

```bash
dnf install epel-release -y
```

Install client:

```bash
dnf install x2goclient -y
```

Launch client:

```bash
x2goclient
```

---

## 2. Configure Session

Create a new session with:

| Field | Value |
|---|---|
| Host | eiclin07 |
| Login | your Linux username |
| SSH Port | 22 |
| Session Type | XFCE |

Save the session and connect.

---

# Windows Client Setup

## 1. Install X2Go Client

Download and install the Windows X2Go Client from:

https://wiki.x2go.org/doku.php/doc:installation:x2goclient

Use default installation settings.

---

## 2. Configure Session

Create a new session:

| Field | Value |
|---|---|
| Host | eiclin07 |
| Login | your Linux username |
| SSH Port | 22 |
| Session Type | XFCE |

Then connect using normal SSH credentials.

---

# Notes

- X2Go uses SSH for authentication and encryption.
- No separate `x2goserver.service` exists.
- Multiple users can connect simultaneously.
- If users cannot connect with X2Go, first verify normal SSH access:

```bash
ssh username@eiclin07
```

- Users must exist in the appropriate Centrify/zone configuration before login.

---

# Troubleshooting

## Verify SSH Connectivity

```bash
ssh username@eiclin07
```

## Verify X2Go Packages

```bash
rpm -qa | grep -i x2go
```

## Verify XFCE Packages

```bash
rpm -qa | grep xfce4
```

## Check SSH Service

```bash
systemctl status sshd
```
