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

## Step 3 - Configure instance firewall

Configure and reload firewall.

```
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```

Check firewall.

```
sudo firewall-cmd --list-all
```

## Step 4 - Configure OCI ingress rules

Add ingress rules for VCN.

```
source-type=CIDR source-cidr=0.0.0.0/0 destination-port-range=80
source-type=CIDR source-cidr=0.0.0.0/0 destination-port-range=443
```
