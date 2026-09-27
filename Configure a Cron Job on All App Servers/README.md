Useful website will help u instead of manual calculating or remembaring what does each field stands for:

https://crontab.guru/


1. Problem

The admins want to test scheduled automation before deploying their real scripts.

We need to do two things on every Nautilus app server:

Install the cronie package and make sure the crond service is running.
Add this cron job for the root user:
*/5 * * * * echo hello > /tmp/cron_text

This means: every 5 minutes, execute the command and write hello to /tmp/cron_text.

2. How to Think

Break the task into:

Package
  ↓
Service
  ↓
Cron schedule
  ↓
Verification
Step 1 — Package

Cron functionality requires the cron daemon/package.

The task explicitly says cronie, so install:

sudo yum install -y cronie
Step 2 — Service

Installing a package doesn't necessarily mean the daemon is running.

Start crond:

sudo systemctl start crond

You can also enable it to start automatically after reboot:

sudo systemctl enable crond

Since this is a server, enabling it is a sensible practice, although the task specifically requires starting it.

Step 3 — Cron job

The five cron fields are:

Minute  Hour  Day  Month  Weekday
  ↓      ↓     ↓     ↓       ↓
 */5     *     *     *       *

So:

*/5

means every 5 minutes.

Because the task specifically says for root, install the entry into root's crontab:

sudo crontab -e

Add:

*/5 * * * * echo hello > /tmp/cron_text
3. Solution

On each Nautilus app server:

sudo yum install -y cronie

Then:

sudo systemctl start crond
sudo systemctl enable crond

Check the service:

systemctl status crond

Then edit root's crontab:

sudo crontab -e

Add:

*/5 * * * * echo hello > /tmp/cron_text
4. Verification

First verify that crond is running:

systemctl is-active crond

Output:

active

Verify the root cron entry:

sudo crontab -l

You should see:

*/5 * * * * echo hello > /tmp/cron_text

Then after up to 5 minutes:

cat /tmp/cron_text

Output:

hello