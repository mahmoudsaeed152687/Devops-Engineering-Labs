
## Problem

The Nautilus DevOps team decided to use **Ansible** for automation and configuration management because of its simple setup and minimal prerequisites.

The **Jump Host** will be used as the **Ansible Controller** for managing and testing tasks on the other servers.

The requirement is to:

* Install **Ansible version 4.9.0**
* Use **pip3 only**
* Make the Ansible binary available **globally**, so all users on the Jump Host can run Ansible commands

---

## How to Think

The important part of this task is understanding the difference between a **user-level installation** and a **system-wide installation**.

If Ansible is installed only for the current user, other users may not be able to access the `ansible` command.

Therefore, we need to install it system-wide using `sudo`.

```text
Jump Host
    │
    └── Ansible 4.9.0
            │
            ├── User 1 → ansible ✓
            ├── User 2 → ansible ✓
            └── User 3 → ansible ✓
```

---

## Solution

### 1. Verify pip3

```bash
pip3 --version
```

Make sure `pip3` is available before installing Ansible.

### 2. Install Ansible 4.9.0 system-wide

```bash
sudo pip3 install ansible==4.9.0
```

### Why `sudo`?

Using `sudo` allows the package and its executable to be installed in a system-wide location instead of only within the current user's environment.

This allows other users on the Jump Host to access the Ansible command.

---

## Verification

### Check the installed Ansible version

```bash
ansible --version
```

Ansible 4.9.0 uses a specific `ansible-core` version underneath, so the output may display both the Ansible package and the corresponding core version.

### Check the Ansible binary location

```bash
which ansible
```

A typical system-wide installation may place the binary under:

```text
/usr/local/bin/ansible
```

### Verify global availability

Switch to another user and run:

```bash
ansible --version
```

The command should work without requiring a user-specific installation.

---

## Key Takeaway

For a system-wide Ansible installation using `pip3`, install the required version with elevated privileges:

```bash
sudo pip3 install ansible==4.9.0
```

The key requirement is not only installing the correct version, but also ensuring that the **Ansible executable is available through the system PATH for all users**.

> **Ansible Controller:** The Jump Host is acting as the control machine from which Ansible commands will be executed against the managed servers.
