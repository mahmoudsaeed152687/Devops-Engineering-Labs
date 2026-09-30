
## Problem

The production support team needed to automate the backup of website media files from **App Server 2** and store a copy on the **Nautilus Storage Server**.

## How to Think

The script should:

```text
/var/www/html/media
        ↓
Create ZIP archive
        ↓
/archives/xfusioncorp_media.zip
        ↓
Copy via SCP
        ↓
Storage Server:/archives/
```

The copy must work **without asking for a password**, so passwordless SSH authentication is required.

## Solution

### 1. Install `zip`

Install the package manually outside the script:

```bash
sudo yum install -y zip
```

### 2. Configure Passwordless SSH

Generate an SSH key on App Server 2 and copy the public key to the Storage Server:

```bash
ssh-keygen -t rsa -b 4096
ssh-copy-id <storage-user>@<storage-server-ip>
```

Verify passwordless access:

```bash
ssh <storage-user>@<storage-server-ip>
```

### 3. Create the Script

Create:

```text
/scripts/media_archive.sh
```

Script:

```bash
#!/bin/bash

zip -r /archives/xfusioncorp_media.zip /var/www/html/media

scp /archives/xfusioncorp_media.zip <storage-user>@<storage-server-ip>:/archives/
```

Make it executable:

```bash
chmod +x /scripts/media_archive.sh
```

## Verification

Run the script without `sudo`:

```bash
/scripts/media_archive.sh
```

Verify the archive exists locally:

```bash
ls -l /archives/xfusioncorp_media.zip
```

Verify the archive exists on the Storage Server:

```bash
ls -l /archives/xfusioncorp_media.zip
```

## Key Takeaway

This task combines **Bash scripting, file archiving, Linux permissions, SSH key authentication, and SCP** to automate website content backups without requiring interactive passwords or `sudo` inside the script.
