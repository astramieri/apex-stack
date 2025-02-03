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