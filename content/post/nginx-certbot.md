---
title: Nginx & Certbot configuration
date: 2021-05-01
tags:
  - linux
---

This is a description of my basic web server setup:

<!--more-->

I use `nginx` web server and `certbot` for the Let's Encrypt certificate generation. Let's install those:
```
# zypper in nginx certbot
```

The `nginx` web server has its configuration in `/etc/nginx` directory. The main
configuration file `nginx.conf` is in most cases fine as it.

The `certbot` will ask you for your e-mail address while being executed for the first time
which I find usefull in case something goes wrong and the certificate is about to expire.

## Nginx configuration snippets

I have two configuration snippets which I usually include into all of my virtual hosts.

1) ssl.conf
```
# mkdir /etc/nginx/ssl
# openssl dhparam -out /etc/nginx/ssl/dhparams.pem 4096
# cat /etc/nginx/ssl.conf
ssl_dhparam ssl/dhparams.pem;
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
ssl_prefer_server_ciphers off;
```

2) letsencrypt.conf
```
# mkdir /var/www/letsencrypt
# cat /etc/nginx/letsencrypt.conf
location ^~ /.well-known/acme-challenge/ {
  allow all;

  default_type "text/plain";
  root /var/www/letsencrypt;

  access_log /var/log/nginx/letsencrypt.access.log;
  error_log /var/log/nginx/letsencrypt.error.log error;
}

location = /.well-known/acme-challenge/ {
  return 404;
}
```

## Nginx virtual hosts

1) For being able to get the certificate, we must have HTTP virtual host:
```
# cat /etc/nginx/vhosts.d/example.com.conf
server {
  listen 80;
  listen [::]:80;

  include letsencrypt.conf;
  server_name example.com www.example.com

  access_log /var/log/nginx/example.com.access.log;
  error_log /var/log/nginx/example.com.error.log error;
}
# systemctl reload nginx.service
```

2) Now we can obtain the certificate from Let's Encrypt:
```
# certbot certonly --webroot --noninteractive -w /var/www/letsencrypt/ -d example.com -d www.example.com
```

3) We may now create the HTTPS virtual host which I usually do in the same file:
```
server {
  listen 443 ssl http2;
  listen [::]:443 ssl http2;

  server_name example.com www.example.com

  include ssl.conf;
  include letsencrypt.conf;
  ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

  access_log /var/log/nginx/example.com.access.log;
  error_log /var/log/nginx/example.com.error.log error;
}
```

4a) If HTTP to HTTPS redirection is needed we may add this:
```
return 301 https://$host$request_uri;
```

4b) If we wanna server static files (e.g. Hugo generated site):
```
index index.html;
root /var/www/pdostal.cz/public/;
try_files $uri $uri/ =404;
```

4c) For proxying traffic to the external software, another server or container:
```
location / {
  proxy_pass http://127.0.0.1:3567/;
  proxy_http_version 1.1;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header Host $http_host;
  proxy_set_header X-Forwarded-Proto https;
  proxy_redirect off;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection "upgrade";
}
```

## Let's Encrypt auto renewal systemd timer

1) For the `systemd-timer` auto renewal we need `.timer` file and `.service` file:
```
# cat /etc/systemd/system/certbot-renewal.service
[Unit]
Description=Certbot Renewal

[Service]
ExecStart=/usr/bin/certbot renew --post-hook "systemctl reload nginx"

# cat /etc/systemd/system/certbot-renewal.timer
[Unit]
Description=Timer for Certbot Renewal

[Timer]
OnBootSec=300
OnUnitActiveSec=1w

[Install]
WantedBy=multi-user.target
```
2) Now we need to enable and start this timer:
```
# systemctl enable --now certbot-renewal.timer
```

3) We may check the status of the timer by those commands:
```
# systemctl list-timers
NEXT                          LEFT                LAST                                PASSED       UNIT                         ACTIVATES
Sat 2021-05-08 14:42:49 CEST  6 days left         Sat 2021-05-01 14:42:49 CEST        36min ago    certbot-renewal.timer        certbot-renewal.service
# journalctl -u certbot-renewal.service
May 01 14:42:49 witbier systemd[1]: Started Certbot Renewal.
May 01 14:50:26 witbier certbot[20717]: Processing /etc/letsencrypt/renewal/example.com.conf
May 01 14:50:25 witbier certbot[20717]: Cert not yet due for renewal
```

## Conclusion

This setup is in my opinion very clean and easy to maintain. Of course I use special options for special cases but the base looks always like this.

