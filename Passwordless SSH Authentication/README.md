
## 1. Problem

The system administration team needs to automate scripts from the **Jump Host** that perform operations across all application servers in the **Stratos Datacenter**.

For this automation to work without human interaction, the `thor` user on the Jump Host must be able to connect to all application servers through their respective administrative users **without entering a password**.

For example:

```text
Jump Host
thor
  │
  ├── SSH → App Server 1 → tony
  ├── SSH → App Server 2 → <server-user>
  └── SSH → App Server 3 → <server-user>
```

---

## 2. How to Think

Password-based SSH authentication is not ideal for automated tasks because every connection would require a password.

Instead, use **SSH public-key authentication**.

The basic model is:

```text
Jump Host
thor
 │
 ├── Private Key 🔒
 └── Public Key 🔑
          │
          ▼
     App Server
          │
          ▼
~/.ssh/authorized_keys
```

The private key remains on the Jump Host.

The public key is copied to the `authorized_keys` file of the appropriate user on each App Server.

When `thor` connects, SSH uses the private key to prove ownership of the corresponding public key.

---

## 3. Implementation

### Step 1 — Verify the current user

```bash
whoami
```

Expected:

```text
thor
```

---

### Step 2 — Generate an SSH Key Pair

Check whether the user already has an SSH key:

```bash
ls -la ~/.ssh
```

If no key exists, generate one:

```bash
ssh-keygen -t rsa -b 4096
```

The command creates:

```text
~/.ssh/id_rsa
~/.ssh/id_rsa.pub
```

Where:

* `id_rsa` → Private key
* `id_rsa.pub` → Public key

The private key must remain on the Jump Host and should never be shared.

---

### Step 3 — Copy the Public Key to the App Servers

For each App Server, copy the public key to the corresponding administrative user.

Example for App Server 1:

```bash
ssh-copy-id tony@<APP_SERVER_1_IP>
```

Repeat the same process for App Server 2 and App Server 3 using their respective users.

`ssh-copy-id` adds the public key to:

```text
~/.ssh/authorized_keys
```

on the target server.

---

## 4. Verification

Test the SSH connection to each App Server:

```bash
ssh tony@<APP_SERVER_1_IP>
```

The connection should succeed without asking for the user's password.

For an automation-style test:

```bash
ssh tony@<APP_SERVER_1_IP> 'hostname'
```

Repeat the verification for all App Servers.

A successful setup should allow:

```text
thor @ Jump Host
        │
        │ SSH
        ▼
App Server
        │
        └── Login without password
```

---

## 5. Important Security Concept

The private key and public key have different roles:

```text
Private Key 🔒
    │
    └── Stays with thor on the Jump Host

Public Key 🔑
    │
    └── Stored in authorized_keys on target servers
```

The private key should **never be copied to the remote servers**.

---

## 6. Key Takeaway

Passwordless SSH does not mean that authentication is disabled.

It means that SSH uses **cryptographic key-based authentication instead of asking for a password**.

The general workflow is:

```text
Generate Key Pair
       ↓
Keep Private Key Secure
       ↓
Copy Public Key to authorized_keys
       ↓
SSH Authentication
       ↓
Passwordless Login
```

This approach is widely used in **DevOps automation, Ansible, CI/CD pipelines, bastion/jump hosts, and remote server administration**.
