1. Problem

The script:

/tmp/xfusioncorp.sh

already exists on App Server 2, but it does not have executable permission.

The requirement is:

The script must be executable.
All users must be able to execute it.

2. How to Think

First identify the Linux permission requirement.

Linux permissions are divided into:

Owner | Group | Others

"All users" means owner + group + others.

We need to add the execute (x) permission for all three.

The simplest command is:

chmod +x /tmp/xfusioncorp.sh

+x adds execute permission for owner, group, and others without removing the permissions that already exist.

3. Solution

On App Server 2:

sudo chmod +x /tmp/xfusioncorp.sh

4. Verification

Check the permissions:

ls -l /tmp/xfusioncorp.sh

You should see x in all three permission groups, for example:

-rwxr-xr-x 1 root root ... /tmp/xfusioncorp.sh

The important part is:

rwx r-x r-x
 ^   ^   ^
 |   |   |
 |   |   └── Others can execute
 |   └────── Group can execute
 └────────── Owner can execute

 5. Key Takeaway

Remember:

chmod +x file

means:

Add execute permission for everyone without changing the existing read/write permissions.

For more precise control, you can use:

chmod a+x /tmp/xfusioncorp.sh

Here a means all users.

🧠 Interview Tip

If someone asks:

"How do you make a script executable for everyone?"

A good answer is:

chmod a+x script.sh

Then verify with:

ls -l script.sh