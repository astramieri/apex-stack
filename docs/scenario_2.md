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

24.3.2.r3121009
```

Check ORDS package version.

```
sudo dnf --showduplicates list ords
```

Install ORDS from the repositories.

```
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