# Pi-Hole-DoH
# Pi-Hole and DNS over HTTPS
https://docs.pi-hole.net/main/basic-install/

# First install Pi-Hole:
sudo curl -sSL https://install.pi-hole.net | bash

# To change the port for the lighttpd web server add port 8080 to the line 'server.port = 8080' in the file /etc/lighthttpd/lighttpd.conf
sudo nano /etc/lighttpd/lighttpd.conf

# Intsll Cloudflare Damon 
# Ubuntu 22.04 (Jammy Jellyfish)
# Add cloudflare gpg key
sudo mkdir -p --mode=0755 /usr/share/keyrings
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-main.gpg >/dev/null

# Add this repo to your apt repositories
echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared jammy main' | sudo tee /etc/apt/sources.list.d/cloudflared.list

# install cloudflared 
sudo apt-get update && sudo apt-get install cloudflared

# Configuring cloudflared to run on startup 
https://docs.pi-hole.net/guides/dns/cloudflared/

# Create a cloudflared user to run the daemon:
sudo useradd -s /usr/sbin/nologin -r -M cloudflared

# Proceed to create a configuration file for cloudflared:
sudo nano /etc/default/cloudflared
# Edit configuration file by copying the following in to /etc/default/cloudflared. This file contains the command-line options that get passed to cloudflared on startup:
---------
# Commandline args for cloudflared, using Cloudflare DNS
CLOUDFLARED_OPTS=--port 5053 --upstream https://1.1.1.1/dns-query --upstream https://1.0.0.1/dns-query
---------

# Update the permissions for the configuration file and cloudflared binary to allow access for the cloudflared user:
sudo chown cloudflared:cloudflared /etc/default/cloudflared
sudo chown cloudflared:cloudflared /usr/local/bin/cloudflared

# Then create the systemd script by copying the following into /etc/systemd/system/cloudflared.service. This will control the running of the service and allow it to run on startup:

sudo nano /etc/systemd/system/cloudflared.service

---------

[Unit]
Description=cloudflared DNS over HTTPS proxy
After=syslog.target network-online.target

[Service]
Type=simple
User=cloudflared
EnvironmentFile=/etc/default/cloudflared
ExecStart=/usr/local/bin/cloudflared proxy-dns $CLOUDFLARED_OPTS
Restart=on-failure
RestartSec=10
KillMode=process

[Install]
WantedBy=multi-user.target


# Enable the systemd service to run on startup, then start the service and check its status:
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
sudo systemctl status cloudflared

# Now test that it is working! Run the following dig command, a response should be returned similar to the one below:
dig @127.0.0.1 -p 5053 google.com


# Configuring Pi-hole¶
# Finally, configure Pi-hole to use the local cloudflared service as the upstream DNS server by specifying 
127.0.0.1#5053 
# as the Custom DNS (IPv4):

![image](https://github.com/Spadinww/Pi-Hole-DoH/assets/51522675/5df13b45-0e8c-40d6-b2c0-1f812f7f1ab3)
