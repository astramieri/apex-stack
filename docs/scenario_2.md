# Scenario 2

The architecture includes:
- **Oracle Autonomous Database (ADB)** with Oracle APEX enabled
- **Oracle REST Data Services (ORDS)** for exposing APEX applications and APIs
- **NGINX** as a reverse proxy for ORDS and APEX
- **Custom Domain** & SSL/TLS configuration for secure access

## Notes

Please note that the following points are not intended as a step-by-step guide but rather a collection of helpful reminders to refer to as needed.

## Step 0 - Basic Setup

See notes [here](./basic_setup.md).

## Step 1 - Install ORDS

Install ORDS from the repositories.

```
sudo dnf install -y ords
```

## Step 2 - Upload wallet

Copy the wallet to the instance.

```
scp -i <path>\<key> <path>\<wallet> opc@<ip>:/tmp
```

Move the wallet and set right permissions.

```
sudo mv /tmp/<wallet> /opt/oracle

sudo chown oracle:oinstall /opt/oracle/<wallet>
```

## Step 3 - Configure firewall

Configure and reload firewall.

```
sudo firewall-cmd --permanent --add-port=8443/tcp
sudo firewall-cmd --permanent --add-forward-port=port=443:proto=tcp:toport=8443
sudo firewall-cmd --reload
```

## Step 4 - Configure NGINX for APEX

Add a reverse proxy entry for the domain ```mydomain.com```.

```
sudo nano /etc/nginx/conf.d/mydomain.com.conf
```

NOTE: replace ```<my_apex_url>```.

```
server {
    server_name mydomain.com;

    location / {
        rewrite ^/$ /ords permanent;
    }

    location /ords/ {
        proxy_pass <my_apex_url>/ords/;
        proxy_set_header Origin "" ;
        proxy_set_header X-Forwarded-Host $host:$server_port;        
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;        
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;   
    }

    location /i/ {
        proxy_pass <my_apex_url>/i/;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Test NGINX configuration.

```
sudo nginx -t 
```

## Step 5 - Install *Let´s Encrypt*

Enable the EPEL repository.

```
sudo dnf install -y oracle-epel-release-el8
sudo dnf config-manager --set-enabled ol8_developer_EPEL
sudo dnf update -y
```

Install ```CertBot```.

```
sudo dnf install -y certbot python3-certbot-nginx
```

Run ```CertBot```.

```
sudo certbot --nginx
```

Enter parameters.

```
sudo mkdir /etc/pki/ords

sudo chown -R oracle:oinstall /etc/pki/ords/   

sudo cp /etc/letsencrypt/live/<mydomain.com>/fullchain.pem /etc/pki/ords
sudo cp /etc/letsencrypt/live/<mydomain.com>/privkey.pem /etc/pki/ords

sudo chmod 644 /etc/pki/ords/fullchain.pem
sudo chmod 644 /etc/pki/ords/privkey.pem
```

## Step N - Configure ORDS

ToFix

```
sudo su - oracle
```

ToFix

```
nano .bash_profile
```

ToFix

```
export ORDS_CONFIG=/etc/ords/config
export PATH=$PATH:/usr/local/bin
```

ToFix

```
source .bash_profile
```

ToFix

```
ords config set jdbc.InitialLimit 1
ords config set jdbc.MaxLimit 3
ords config set jdbc.MinLimit 0
```

ToFix

```
ords install adb --interactive --prompt-password
```

Enter the following parameters.

```
ORDS: Release 24.4 Production on Sun Feb 02 14:48:33 2025

Copyright (c) 2010, 2025, Oracle.

Configuration:
  /etc/ords/config

The configuration folder /etc/ords/config does not contain any configuration files.

Oracle REST Data Services - Interactive Customer Managed ORDS for Autonomous Database

Enter the Autonomous Database Wallet path: /opt/oracle/<wallet>
Enter a number to select the TNS Network alias to use
    [1] <DBNAME>_LOW 
    [2] <DBNAME>_MEDIUM
    [3] <DBNAME>_HIGH
    [4] <DBNAME>_TP 
    [5] <DBNAME>_TPURGENT
  Choose [1]: 3
Enter the administrator username [ADMIN]: <user>
Enter the database password for <user>: <password>
Enter the ORDS runtime database username [ORDS_PUBLIC_USER2]: ords_public_user2
Enter the database password for ords_public_user2: <password>
Enter the PL/SQL Gateway database username: ords_plsql_gateway2
Enter the database password for ords_plsql_gateway2: <password>
Enter a number to select additional feature(s) to enable:
    [1] Database Actions  (Enables all features)
    [2] REST Enabled SQL and Database API
    [3] REST Enabled SQL
    [4] Database API
    [5] None
  Choose [1]: 1
Enter a number to configure and start ORDS in standalone mode
    [1] Configure and start ORDS in standalone mode
    [2] Skip
  Choose [1]: 1
Enter a number to select the protocol
    [1] HTTP
    [2] HTTPS
  Choose [1]: 2 
Enter the HTTPS port [8443]: 8443
```