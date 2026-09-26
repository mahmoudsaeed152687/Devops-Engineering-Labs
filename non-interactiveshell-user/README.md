Problem:
The system admin team needs a user named anita on App Server 2. The user must have a non-interactive shell because it's intended for a service/backup agent rather than normal human login.

How to Think:

We need to create a Linux user → useradd.
The requirement says non-interactive shell.
Linux provides /sbin/nologin for accounts that should not be used for interactive login.
Therefore, create the user and explicitly assign /sbin/nologin.
Verify the user's entry in /etc/passwd.

Solution:

sudo useradd -s /sbin/nologin anita

Verification:

grep '^anita:' /etc/passwd

Key Takeaway:
/sbin/nologin is commonly used for service accounts that need to exist but should not allow interactive shell access.