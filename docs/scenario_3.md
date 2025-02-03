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

Check ADB ORDS version.

```
SELECT ords.installed_version FROM dual;
```

Install ORDS from the repositories.

```
sudo dnf --showduplicates list ords

sudo dnf install -y ords-24.3.2-1.el8
```

**NOTE**. The package creates the ```oracle``` user and puts the installation and configuration under the oracle ownership.

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

## Step 3 - Configure ORDS

Switch the current user.

```
sudo su - oracle
```

Start the ORDS configuration process. 

```
ords install adb --interactive --prompt-password
```

Enter the parameters.

```
Enter the Autonomous Database Wallet path: <wallet_path>
Enter a number to select the TNS Network alias to use: 1
Enter the administrator username [ADMIN]: ADMIN
Enter the database password for ADMIN: <password>
Enter the ORDS runtime database username [ORDS_PUBLIC_USER2]: ORDS_PUBLIC_USER2
Enter the database password for ORDS_PUBLIC_USER2: <password>
Enter the PL/SQL Gateway database username: ORDS_PLSQL_GATEWAY2
Enter the database password for ORDS_PLSQL_GATEWAY2: <password>
Enter a number to select additional feature(s) to enable: 1
Enter a number to configure and start ORDS in standalone mode: 1
Enter a number to select the protocol: 1
Enter the HTTP port [8080]: 8080
```

**NOTE**. Use this script if the first configurations fails.

```
BEGIN
    ORDS_ADMIN.CONFIG_PLSQL_GATEWAY(
        p_runtime_user       => 'ORDS_PUBLIC_USER2', 
        p_plsql_gateway_user => 'ORDS_PLSQL_GATEWAY2'
    );
END;
```

## Step 4 - Setup ORDS to start automatically

Setup ORDS to start automatically on boot.

```
sudo systemctl start ords
sudo systemctl enable ords
```

## Step 5 - Configure NGINX for APEX

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

## Step 6 - Install *Let´s Encrypt*

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

Copy files.

```
sudo mkdir /etc/pki/ords

sudo chown -R oracle:oinstall /etc/pki/ords/   

sudo cp /etc/letsencrypt/live/<mydomain.com>/fullchain.pem /etc/pki/ords
sudo cp /etc/letsencrypt/live/<mydomain.com>/privkey.pem /etc/pki/ords

sudo chmod 644 /etc/pki/ords/fullchain.pem
sudo chmod 644 /etc/pki/ords/privkey.pem
```

## Step 7 - Re-Configure ORDS

Switch the current user.

```
sudo su - oracle
```

Start the ORDS configuration process. 

```
ords install adb --interactive --prompt-password
```

Enter the parameters.

```
Enter the Autonomous Database Wallet path: <wallet_path>
Enter a number to select the TNS Network alias to use: 1
Enter the administrator username [ADMIN]: ADMIN
Enter the database password for ADMIN: <password>
Enter the ORDS runtime database username [ORDS_PUBLIC_USER2]: ORDS_PUBLIC_USER2
Enter the database password for ORDS_PUBLIC_USER2: <password>
Enter the PL/SQL Gateway database username: ORDS_PLSQL_GATEWAY2
Enter the database password for ORDS_PLSQL_GATEWAY2: <password>
Enter a number to select additional feature(s) to enable: 1
Enter a number to configure and start ORDS in standalone mode: 1
Enter a number to select the protocol: 1
Enter the HTTP port [8080]: 8080
```

**NOTE**. Use this script if the first configurations fails.

```
BEGIN
    ORDS_ADMIN.CONFIG_PLSQL_GATEWAY(
        p_runtime_user       => 'ORDS_PUBLIC_USER2', 
        p_plsql_gateway_user => 'ORDS_PLSQL_GATEWAY2'
    );
END;
```

## Step 8 - Fix Bad Gateway 

Run this commands.

```
sudo cat /var/log/audit/audit.log | grep nginx | grep denied | audit2allow -M apex_proxy

sudo semodule -i apex_proxy.pp
```
