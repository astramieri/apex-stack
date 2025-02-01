# Scenario 1

The architecture includes:
- **Oracle Autonomous Database (ADB)** with Oracle APEX enabled
- **NGINX** as a reverse proxy for APEX
- **Custom Domain** & SSL/TLS configuration for secure access

## Notes

Please note that the following points are not intended as a step-by-step guide but rather a collection of helpful reminders to refer to as needed.

## Step 0 - Basic Setup

See notes [here](./basic_setup.md).

## Step 1 - Configure NGINX for APEX

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

## Step 2 - Install *Let´s Encrypt*

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

## Step 3 - Automate certificate renewal

Create a new cronjob.

```
sudo crontab -e
```

Add the following entry.

```
0 0 * * * /bin/certbot renew --quiet --post-hook "systemctl restart nginx"
```

Check cronjob.

```
sudo crontab -l
```

## Step 4 - Fix Bad Gateway 

Run this commands.

```
sudo cat /var/log/audit/audit.log | grep nginx | grep denied | audit2allow -M apex_proxy

sudo semodule -i apex_proxy.pp
```

## Step 5 - Point to a specific APEX app

Edit the NGINX configuration file.

```
sudo nano /etc/nginx/conf.d/mydomain.com.conf
```

NOTE: replace ```<my_apex_workspace>``` and ```<my_apex_app>```.

```
location / {
    rewrite ^/$ /ords/r/<my_apex_workspace>/<my_apex_app> permanent;
}
```

Test NGINX configuration.

```
sudo nginx -t 
```

Restart NGINX.

```
sudo nginx -s reload
```