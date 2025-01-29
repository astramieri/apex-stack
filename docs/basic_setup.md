# Basic Setup

Please note that the following points are not intended as a step-by-step guide but rather a collection of helpful reminders to refer to as needed

## Step 1 - Setup instance

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

## Step 2 - Install NGINX

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

### Step 3 - Configure firewall and rules

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

### Step 4 - Configure NGINX for custom domain (TOFIX)

Add website configuration.

```
sudo nano /etc/nginx/conf.d/mydomain.com.conf

server {
    listen         80;
    listen         [::]:80;
    server_name    mydomain.com www.mydomain.com;
    root           /usr/share/nginx/html/mydomain.com;
    index          index.html;
    try_files $uri /index.html;
}
```

Create the website directory.

```
sudo mkdir /usr/share/nginx/html/mydomain.com
```

Test NGINX configuration.

```
sudo nginx -t
```

Restart NGINX.

```
sudo nginx -s reload
```

### Step 5 - Obtain an SSL Certificate

Install Certbot.

```
sudo dnf install epel-release -y
sudo dnf install python3 python3-pip -y

sudo certbot --nginx
```