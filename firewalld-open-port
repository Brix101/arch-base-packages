# Open a Port in Firewalld

## Log into SSH

Log into the system using SSH (Secure Shell). This is a common method for remotely accessing and managing a Linux server.

### Check if the application port is defined as a service

Use the following command to check if the desired application port is already defined as a service. Services are predefined sets of rules for common applications.

```bash
firewall-cmd --get-services
```

If the service is listed, proceed to whitelist it permanently in the current zone.

### Whitelist a service

If the service is listed in the output of the previous command, use the following command to permanently allow traffic for that service in the current zone. Replace "SERVICE" with the actual service name.

```bash
sudo firewall-cmd --permanent --add-service=SERVICE
```

### If the service is not listed

If the service for your application is not listed, open the port permanently by specifying the port number and protocol (TCP or UDP). Replace "1234" with the actual port number and "tcp" with the protocol used by your application.

```bash
sudo firewall-cmd --permanent --add-port=1234/tcp
```

### Reload Firewalld

After making changes, reload Firewalld to apply the new configurations.

```bash
firewall-cmd --reload
```

### Check Open Ports in Firewalld

After you whitelist ports and services, you should ensure your changes are accurate.

List whitelisted services in Firewalld:

```bash
sudo firewall-cmd --list-services
```

List currently open ports in Firewalld:

```bash
sudo firewall-cmd --list-ports
```
