# bitwarden-podman
podman rootless quadlet files for bitwarden

# Setup
On RHEL 9 based systems like Rocky 9, RHEL 9, Oracle Linux 9 and Alma Linux install podman and systemd-container like so

```
dnf install podman systemd-container -y
```

After add a user like 'bitwarden' and enable linger

```
useradd bitwarden
loginctl enable-linger bitwarden
```

Next switch user to bitwarden and clone the repository

```
machinectl shell bitwarden@
git clone git@github.com:TheRealKingS/bitwarden-podman.git
```

Then move all files to ~/.config/containers/systemd

```
mkdir -p ~/.config/containers/systemd
cp bitwarden-podman/* ~/.config/containers/systemd
```

Now set the network cmd in `~/.config/containers/containers.conf`:

```
[network]
default_rootless_network_cmd = "slirp4netns"
```

Last step: relaod systemd daemon and start all containers

```
systemctl --user daemon-reload
systemctl --user start bitwarden-mssql bitwarden-web bitwarden-attachments bitwarden-api bitwarden-identity bitwarden-sso bitwarden-admin bitwarden-icons bitwarden-notifications bitwarden-events
systemctl --user start bitwarden-nginx
```

Now your bitwarden vault is reachable under http://<Your Domain>:8099. If you like to setup an reverse proxy, check the following configuration example for apache out:

```
<VirtualHost *:80>
   ServerName bitwarden.example.com

   DocumentRoot /var/www/html
</VirtualHost>

<VirtualHost *:443>
        ServerAdmin bitwarden@example.com
        ServerName bitwarden@example.com
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

# Further links

Official Bitwarden documentation: https://bitwarden.com/help/install-on-premise-manual/
