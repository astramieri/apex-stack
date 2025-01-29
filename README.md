# Full APEX Stack on Oracle Cloud Free Tier

This repository contains notes and configurations for setting up a full Oracle APEX stack on **Oracle Cloud Infrastructure (OCI)** using the Free Tier. 

The goal is to create a production-ready architecture that leverages OCI's free resources while maintaining scalability, security, and best practices.

## Overview

The architecture includes:
- **Oracle Autonomous Database (ADB)** with Oracle APEX enabled
- **Oracle REST Data Services (ORDS)** for exposing APEX applications and APIs
- **NGINX** as a reverse proxy for ORDS and APEX
- **Custom Domain** & SSL/TLS configuration for secure access

## Notes

Please note that the following points are not intended as a step-by-step guide but rather a collection of helpful reminders to refer to as needed

### Step 1 - Initial setup

Update system packages.

```
sudo dnf update -y
```

Install Java (latest).

```
sudo dnf install -y java-21-openjdk
```

Check Java version.

```
java -version
```

### Step 2 - Install NGINX

Install NGINX.

```
sudo dnf install -y nginx
```

Enable and start NGINX.

```
sudo systemctl enable nginx
sudo systemctl start nginx
```

Check NGINX status.

```
sudo systemctl status nginx
```

Configure and reload firewall.

```
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```

Add ingress rules for OCI VCN.

```
source-type=CIDR source-cidr=0.0.0.0/0 destination-port-range=80
source-type=CIDR source-cidr=0.0.0.0/0 destination-port-range=443
```