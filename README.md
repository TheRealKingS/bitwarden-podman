# bitwarden-podman
Podman rootless Quadlet files for Bitwarden.

---

## Setup
### Requirements
On RHEL 9-based systems like Rocky Linux 9, RHEL 9, Oracle Linux 9, and AlmaLinux, install `podman` and `systemd-container` as follows:

```bash
dnf install podman systemd-container -y
```

### Create a User and Enable Linger
Create a new user, e.g., `bitwarden`, and enable `linger` to allow the user to run systemd services:

```bash
useradd bitwarden
loginctl enable-linger bitwarden
```

### Clone the Repository
Switch to the `bitwarden` user and clone this repository:

```bash
machinectl shell bitwarden@
git clone git@github.com:TheRealKingS/bitwarden-podman.git
```

### Move Files
Move all files to the directory `~/.config/containers/systemd`:

```bash
mkdir -p ~/.config/containers/systemd
cp bitwarden-podman/* ~/.config/containers/systemd
```

Last step: relaod systemd daemon and start all containers

### Adjust Rootless Networking
**IMPORTANT:**  
The default rootless network driver is `pasta`, which prevents host accessibility. To fix this, create the file `~/.config/containers/containers.conf` with the following content:

```ini
[network]
default_rootless_network_cmd = "slirp4netns"
```

### Start the Services
Reload the systemd daemon and start all required containers:

```bash
systemctl --user daemon-reload
systemctl --user start bitwarden-mssql bitwarden-web bitwarden-attachments bitwarden-api bitwarden-identity bitwarden-sso bitwarden-admin bitwarden-icons bitwarden-notifications bitwarden-events
systemctl --user start bitwarden-nginx
```

Your Bitwarden vault should now be accessible at `http://<your-domain>:8099`.

## Reverse Proxy Configuration
If you'd like to set up a reverse proxy using Apache, use the following example configuration:

```apache
<VirtualHost *:80>
   ServerName bitwarden.example.com
   DocumentRoot /var/www/html
</VirtualHost>

<VirtualHost *:443>
    ServerAdmin bitwarden@example.com
    ServerName bitwarden.example.com
    DirectoryIndex index.php index.htm index.html
    DocumentRoot /var/www/html
    Header add Strict-Transport-Security "max-age=15768000"
    SSLEngine on
    SSLCertificateKeyFile  /etc/letsencrypt/live/bitwarden.example.com/privkey.pem
    SSLCertificateFile  /etc/letsencrypt/live/bitwarden.example.com/cert.pem
    SSLCertificateChainFile /etc/letsencrypt/live/bitwarden.example.com/chain.pem

    ProxyVia On
    ProxyRequests Off
    ProxyPass / http://127.0.0.1:8099/
    ProxyPassReverse / http://127.0.0.1:8099/
    ProxyPreserveHost on

    <Proxy *>
        Options FollowSymLinks MultiViews
        AllowOverride All
        Require all granted
    </Proxy>

    ServerSignature Off
</VirtualHost>
```

## Further Links
Official Bitwarden documentation: [Install On-Premise (Manual)](https://bitwarden.com/help/install-on-premise-manual/)
