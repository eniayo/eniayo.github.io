# swaveycode.site Deployment

This project sets up and configures the deployment of `swaveycode.site` using Nginx on a DigitalOcean droplet. The deployment includes obtaining an SSL certificate using Certbot for HTTPS support.

## Prerequisites

- A DigitalOcean droplet running Ubuntu 22.04 or later
- A domain name (`swaveycode.site`) managed via a DNS provider
- A valid SSH connection to the server

## Steps to Set Up Deployment

### 1. Update System Packages
```sh
sudo apt update && sudo apt upgrade -y
```

### 2. Install Nginx
```sh
sudo apt install nginx -y
```

### 3. Configure Firewall (Optional)
```sh
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

### 4. Verify Nginx is Running
```sh
systemctl status nginx
```

### 5. Configure Domain DNS Records
Ensure the following A record exists:

| Type | Name | Value |
|------|------|--------|
| A    | @    | 64.227.102.251 |
| CNAME | www  | swaveycode.site |

### 6. Configure Nginx for the Domain
```sh
sudo nano /etc/nginx/sites-available/swaveycode
```
Add the following configuration:

```nginx
server {
    listen 80;
    server_name swaveycode.site www.swaveycode.site;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
Save and exit, then enable the configuration:
```sh
sudo ln -s /etc/nginx/sites-available/swaveycode /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### 7. Obtain an SSL Certificate using Certbot
```sh
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d swaveycode.site -d www.swaveycode.site
```
Follow the prompts to enter an email and agree to the terms.

### 8. Verify SSL Configuration
```sh
curl -I https://swaveycode.site
curl -I https://www.swaveycode.site
```
Both should return `HTTP/1.1 200 OK` with SSL enabled.

### 9. Automate SSL Renewal
```sh
sudo systemctl list-timers | grep certbot
```
Certbot is automatically configured to renew SSL certificates.

## Troubleshooting
### Multiple IP Addresses in DNS Resolution
To ensure that only the correct IP is associated with your domain, update the DNS settings and remove any incorrect A records.

### Checking DNS Resolution
```sh
dig A swaveycode.site +short
```
Ensure it returns only `64.227.102.251`.

### Checking Web Server Logs
```sh
sudo journalctl -u nginx --since "1 hour ago"
```

### Restart Nginx if Needed
```sh
sudo systemctl restart nginx
```

## Conclusion
The website `swaveycode.site` is now fully deployed with HTTPS support via Certbot and Nginx.

