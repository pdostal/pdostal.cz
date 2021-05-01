---
title: Matomo docker-compose deployment
date: 2021-04-25
tags:
  - linux
---

As I still didn't deploy Kubernetes on my personal server, I introduce you my Docker(-Compose) setup:

<!--more-->

First of all we need to install both the `docker` and the `docker-compose` utility:
```
zypper in docker python3-docker-compose
```

## Container deployment

We must create a directory for our `docker-compose.yml` and some Matomo specific configuration files:
```
# mkdir -p /var/www/matomo/; cd /var/www/matomo/
# wget https://raw.githubusercontent.com/matomo-org/docker/master/.examples/nginx/docker-compose.yml 
# wget https://raw.githubusercontent.com/matomo-org/docker/master/.examples/nginx/matomo.conf 
# wget https://raw.githubusercontent.com/matomo-org/docker/master/.examples/nginx/db.env
# edit db.env # Put random value for MYSQL_PASSWORD and the same for MATOMO_DATABASE_PASSWORD
# edit docker-compose.yml # Put random value for MYSQL_ROOT_PASSWORD
```

After we have the configuration ready we can start the whole setup:
```
# docker-compose up -d
Creating matomo_web_1 ... done
Creating matomo_db_1  ... done
Creating matomo_app_1 ... done
```

When all containers are running we can connect to our machine on port 8080 and setup Matomo.

## Proxy
It is recommended to put the instance behind a proxy server such as HAProxy or NGINX.
I wrote [blog post]({{< ref "nginx-certbot" >}}) exmplaing how to setup NGINX for this.

## Hugo integration - Harbor theme

With working Matomo instance, I just had to [Add Matomo support](https://github.com/matsuyoshi30/harbor/pull/102)
for Harbor theme and add Motomo domain name and site ID to my `config.toml`:

```
[params.matomo]
  domain = "matomo.pdostal.cz"
  id = "1"
```

If your template does not support Matomo natively, you may add the tracking code
manually as well.
