+++
title = "Matomo docker-compose deployment"
date = "2021-04-25"
+++

As I still didn't deploy Kubernetes on my personal server, I introduce you my Docker(-Compose) setup:
<!--more-->

```
# mkdir -p /var/www/matomo/; cd /var/www/matomo/
# wget https://raw.githubusercontent.com/matomo-org/docker/master/.examples/nginx/docker-compose.yml 
# wget https://raw.githubusercontent.com/matomo-org/docker/master/.examples/nginx/matomo.conf 
# wget https://raw.githubusercontent.com/matomo-org/docker/master/.examples/nginx/db.env
# edit db.env # Put random value for MYSQL_PASSWORD and the same for MATOMO_DATABASE_PASSWORD
# edit docker-compose.yml # Put random value for MYSQL_ROOT_PASSWORD
# docker-compose up -d
Creating matomo_web_1 ... done
Creating matomo_db_1  ... done
Creating matomo_app_1 ... done
```

When all containers are running we can connect to our machine on port 8080 and setup Matomo.

## Proxy
It is recommended to put the instance behind a proxy server such as HAProxy or NGINX.
I plan to write an article about my NGINX setup later and I will link it here as well.

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
