1. Problem

The security team requires direct SSH login as root to be disabled on all App Servers in the Stratos Datacenter.

This is a server-hardening task. We need to change the SSH daemon configuration on every app server, then reload/restart SSH so the change takes effect.

2. How to Think

The key question is:

Where does SSH decide whether root is allowed to log in?

The SSH server configuration is normally located at:

/etc/ssh/sshd_config

The setting we care about is:

PermitRootLogin

We want:

PermitRootLogin no

Because the task says all app servers, don't stop after configuring one server. Connect to each App Server and apply the same change.

3. Solution

On each App Server:

First check the existing configuration:

sudo grep -i '^PermitRootLogin' /etc/ssh/sshd_config

Edit the SSH configuration:

sudo vi /etc/ssh/sshd_config

Find:

PermitRootLogin yes

or a commented/default setting such as:

#PermitRootLogin prohibit-password

Set it to:

PermitRootLogin no

Then validate the SSH configuration before restarting SSH:

sudo sshd -t

If there is no output, the configuration syntax is valid.

Restart/reload SSH:

sudo systemctl restart sshd

4. Verification

Check the configuration:

sudo grep -i '^PermitRootLogin' /etc/ssh/sshd_config

Expected:

PermitRootLogin no

5. Key Takeaway

For SSH hardening:

/etc/ssh/sshd_config
        ↓
PermitRootLogin no
        ↓
sshd -t
        ↓
restart/reload sshd
        ↓
verify with sshd -T
⚠️ Important

Don't restart SSH before running:

sudo sshd -t