# Rocket.Chat Installation Guide

## Prerequisites
- Linux-based system (Ubuntu, Debian, Fedora, etc.)
- CPU with AVX/AVX2 support
- Snap package manager

## Installation Steps

### 1. Deploy Rocket.Chat
```bash
# Install specific version
sudo snap install rocketchat-server --channel=x.x/stable

# Or install latest version
sudo snap install rocketchat-server
```

Access workspace at `http://localhost:3000`. First user to complete setup wizard becomes administrator.

### 2. Enable HTTPS

#### Option 1: Using Caddy (Recommended)
1. Ensure domain resolves to server IP
2. Configure site URL:
```bash
sudo snap set rocketchat-server siteurl=https://<your-domain>
```
3. Start services:
```bash
sudo systemctl enable --now snap.rocketchat-server.rocketchat-caddy
sudo snap restart rocketchat-server
```

#### Option 2: Self-Signed SSL with Nginx

1. Generate SSL Certificate:
```bash
openssl genrsa -out localhost.key 2048
openssl req -x509 -new -nodes -key localhost.key -sha256 -days 365 -out localhost.crt
```

2. Configure Nginx:
```bash
sudo mkdir -p /etc/nginx/ssl
sudo cp localhost.key localhost.crt /etc/nginx/ssl/
sudo chmod 600 /etc/nginx/ssl/localhost.*
```

3. Create Nginx configuration in `/etc/nginx/sites-available/rocketchat`
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

4. Enable and start Nginx:
```bash
sudo ln -s /etc/nginx/sites-available/rocketchat /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### 3. Rate Limiting with Fail2ban

1. Install fail2ban:
```bash
sudo apt-get install fail2ban
```

2. Configure jail in `/etc/fail2ban/jail.local`:
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

## Troubleshooting
```bash
# Check Nginx logs
sudo tail -f /var/log/nginx/error.log

# Verify SSL certificate
openssl x509 -in /etc/nginx/ssl/localhost.crt -text -noout

# Check Nginx status
sudo systemctl status nginx
```
