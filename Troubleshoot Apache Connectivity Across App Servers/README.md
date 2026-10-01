# Troubleshoot Apache Connectivity Across App Servers

## Problem

The monitoring system reported that Apache was not reachable on the required ports in the Stratos Datacenter.

The goal was to identify the root cause on the affected application servers and restore connectivity without modifying the existing `index.html` or weakening security settings.

## How to Think

The troubleshooting approach was:

```text
Connectivity Test
       ↓
Check Apache Service
       ↓
Check Listening Ports
       ↓
Check Apache Configuration
       ↓
Check Firewall Rules
       ↓
Identify Root Cause
       ↓
Apply Minimal Fix
       ↓
Verify from Jump Host
```

## Diagnostic Commands

### 1. Test Connectivity from the Jump Host

```bash
curl http://stapp01:5000
curl http://stapp03:6200
```

These tests confirmed that the Apache services were not reachable from the Jump Host.

### 2. Check Listening Ports

On the affected server:

```bash
sudo ss -lntp
```

For a specific port:

```bash
sudo ss -lntp | grep 5000
sudo ss -lntp | grep 6200
```

### 3. Check Apache Service Status

```bash
sudo systemctl status httpd --no-pager
```

For detailed service output:

```bash
sudo systemctl status httpd -l --no-pager
```

### 4. Validate Apache Configuration

```bash
sudo apachectl configtest
```

Expected:

```text
Syntax OK
```

The `AH00558` `ServerName` message was only a warning and was not the cause of the failure.

### 5. Find Apache Listen Configuration

```bash
sudo grep -R "^[[:space:]]*Listen" /etc/httpd/conf /etc/httpd/conf.d
```

### 6. Check Firewall Rules

`firewall-cmd` was not available on App Server 1, so `iptables` was used:

```bash
sudo iptables -L -n -v
```

For numbered rules:

```bash
sudo iptables -L INPUT -n -v --line-numbers
```

## Investigation

### App Server 1

Apache was configured to listen on port `5000`:

```text
Listen 5000
```

However, port `5000` was already being used by Sendmail:

```bash
sudo ss -lntp | grep 5000
```

Output showed:

```text
127.0.0.1:5000    → sendmail
```

The Apache service logs confirmed the conflict:

```text
Address already in use
could not bind to address [::]:5000
could not bind to address 0.0.0.0:5000
```

Sendmail's configuration confirmed that port `5000` was intentionally configured for localhost:

```bash
sudo grep -R "5000" /etc/mail /etc/sendmail* 2>/dev/null
```

Relevant configuration:

```text
Port=5000,Addr=127.0.0.1
```

The server IP was identified with:

```bash
ip addr
```

The network IP was:

```text
10.244.49.64
```

### Root Cause on App Server 1

Apache was trying to bind to all interfaces on port `5000`, while Sendmail was already using:

```text
127.0.0.1:5000
```

This caused the Apache startup failure.

In addition, the firewall had a final `REJECT` rule for new connections:

```text
REJECT all ... reject-with icmp-host-prohibited
```

### App Server 3

The listening-port check showed:

```bash
sudo ss -lntp | grep 5003
```

Apache was listening on:

```text
*:5003
```

Apache configuration confirmed:

```bash
sudo grep -R "Listen" /etc/httpd/conf /etc/httpd/conf.d
```

Result:

```text
Listen 5003
```

The required port was `6200`, so Apache was configured to use the wrong port.

## Solution

### App Server 1

Configure Apache to listen only on the server's network IP while keeping port `5000`:

```apache
Listen 10.244.49.64:5000
```

Validate:

```bash
sudo apachectl configtest
```

Restart Apache:

```bash
sudo systemctl restart httpd
```

Verify:

```bash
sudo ss -lntp | grep 5000
```

Expected:

```text
127.0.0.1:5000       → sendmail
10.244.49.64:5000    → httpd
```

The firewall was updated without disabling it:

```bash
sudo iptables -I INPUT -p tcp --dport 5000 -j ACCEPT
```

Verify the rule:

```bash
sudo iptables -L INPUT -n -v --line-numbers
```

### App Server 3

Change the Apache listener from:

```apache
Listen 5003
```

to:

```apache
Listen 6200
```

Validate:

```bash
sudo apachectl configtest
```

Restart:

```bash
sudo systemctl restart httpd
```

Verify:

```bash
sudo ss -lntp | grep 6200
```

## Verification

From the Jump Host:

```bash
curl http://stapp01:5000
```

```bash
curl http://stapp03:6200
```

Both applications returned the expected webpage successfully.

## Key Takeaway

When an application is unreachable, do not assume the service is simply down.

Use a structured troubleshooting process:

```text
Connectivity
    ↓
Service Status
    ↓
Listening Ports
    ↓
Configuration
    ↓
Firewall
    ↓
Root Cause
    ↓
Minimal Fix
    ↓
Remote Verification
```

The main lesson from this task was that the same symptom — an unreachable web service — can have completely different root causes on different servers.
