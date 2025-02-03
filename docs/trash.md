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
