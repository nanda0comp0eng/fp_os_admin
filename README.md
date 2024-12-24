# Rocket.Chat Installation Guide

<img src="/api/placeholder/800/400" alt="rocket-chat" />

A comprehensive guide for installing and configuring Rocket.Chat with HTTPS support.

## Prerequisites

- Linux-based system (Ubuntu, Debian, Fedora, etc.)
- CPU with AVX/AVX2 support
- Snap package manager

## Installation Steps

### Rocket.Chat

<img src="/api/placeholder/800/400" alt="snap-install" />

1. Install Rocket.Chat using Snap:
```bash
sudo snap install rocketchat-server --channel=x.x/stable
```
Note: Replace x.x with your desired version (e.g., 6.x/stable)

For the latest version:
```bash
sudo snap install rocketchat-server
```

2. Access your workspace at `http://localhost:3000`
   - First user to complete setup wizard becomes administrator

### HTTPS Setup

<img src="/api/placeholder/800/400" alt="caddy-ssl" />

#### Using Caddy (Recommended for 4.x AMD64 or 3.x ARM64)

1. Configure domain resolution
2. Set site URL:
```bash
sudo snap set rocketchat-server siteurl=https://<your-domain>
```
3. Enable services:
```bash
sudo systemctl enable --now snap.rocketchat-server.rocketchat-caddy
sudo snap restart rocketchat-server
```

### Nginx Reverse Proxy

<img src="/api/placeholder/800/400" alt="nginx" />

#### SSL Certificate Generation
1. Generate private key:
```bash
openssl genrsa -out localhost.key 2048
```

2. Create SSL certificate:
```bash
openssl req -x509 -new -nodes -key localhost.key -sha256 -days 365 -out localhost.crt
```

#### Nginx Configuration

1. Create SSL directory and copy certificates:
```bash
sudo mkdir -p /etc/nginx/ssl
sudo cp localhost.key localhost.crt /etc/nginx/ssl/
sudo chmod 600 /etc/nginx/ssl/localhost.*
```

2. Create and configure Nginx site:
```nginx
server {
    listen 443 ssl;
    server_name your.domain.com;

    ssl_certificate /etc/nginx/ssl/localhost.crt;
    ssl_certificate_key /etc/nginx/ssl/localhost.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        proxy_set_header X-Nginx-Proxy true;
        proxy_redirect off;
    }
}

server {
    listen 80;
    server_name your.domain.com;
    return 301 https://$server_name$request_uri;
}
```

3. Enable and start Nginx:
```bash
sudo ln -s /etc/nginx/sites-available/reverse-proxy /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### Fail2ban Configuration

<img src="/api/placeholder/800/400" alt="fail2ban" />

1. Install Fail2ban:
```bash
sudo apt-get install fail2ban
```

2. Configure jail settings in `/etc/fail2ban/jail.local`:
```ini
[nginx-limit-req]
enabled = true
port = http,https
filter = nginx-limit-req
action = iptables-multiport[name=http, port="http,https", protocol=tcp, http-status=429]
logpath = /var/log/nginx/access.log
findtime = 3600
maxretry = 100
bantime = 3600
```

### SSH Port Configuration

<img src="/api/placeholder/800/400" alt="ssh" />

1. Edit SSH configuration:
```bash
sudo nano /etc/ssh/sshd_config
```
Change port to 2222:
```plaintext
Port 2222
```

2. Update firewall rules:
```bash
sudo ufw allow 2222/tcp
sudo ufw deny 22/tcp
```

3. Restart SSH service:
```bash
sudo systemctl restart ssh
```

## Troubleshooting

### Nginx
- Check error logs: `sudo tail -f /var/log/nginx/error.log`
- Verify SSL certificate: `openssl x509 -in /etc/nginx/ssl/localhost.crt -text -noout`
- Check service status: `sudo systemctl status nginx`

## Additional Notes
- Always maintain backup SSH access when changing SSH port
- Update hosts file: Add `<ip> <domain>` to `/etc/hosts`
- Use strong passwords and SSH keys for enhanced security
