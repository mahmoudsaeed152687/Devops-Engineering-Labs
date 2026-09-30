
## Problem

Deploy a Java web application on App Server 2 using Tomcat and make it accessible through port `6000`.

## How to Think

```text
ROOT.war
   ↓
App Server 2
   ↓
Tomcat
   ↓
Port 6000
   ↓
http://stapp02:6000
```

## Solution

### 1. Install Tomcat

```bash
sudo yum install -y tomcat
```

### 2. Configure Port 6000

Edit:

```bash
sudo vi /etc/tomcat/server.xml
```

Change the Connector port from `8080` to `6000`.

### 3. Copy the Application

Copy `ROOT.war` from the Jump Host to App Server 2:

```bash
scp /tmp/ROOT.war <user>@stapp02:/tmp/
```

Then deploy it:

```bash
sudo cp /tmp/ROOT.war /var/lib/tomcat/webapps/
```

`ROOT.war` is deployed at the base URL `/`.

### 4. Start Tomcat

```bash
sudo systemctl enable --now tomcat
```

## Verification

Check the service:

```bash
sudo systemctl status tomcat
```

Test the application:

```bash
curl http://stapp02:6000
```

## Key Takeaway

Tomcat is a Java application server used to run Java web applications. A WAR file contains the web application, and naming it `ROOT.war` makes it available directly at the server's base URL.
