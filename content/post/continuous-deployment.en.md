---
title: Continuous deployment
date: 2021-04-25
tags:
  - linux
---

After trying to to deploy to GitLab Pages and GitHub Pages I decided to deploy
to my own server as neither of those providers fit my needs, because:

 * Neither of those providers support IPv6.
 * Second repository is necessary in GitHub.
 * I already have my own webserver.

<!--more-->

First of all, I'd like to remind you that this is not necessary and that for most
people those providers mentioned above will probably be good enough.
They even support custom domain names.

Second of all, my goal is to keep my setup as simple as possible and focus as much
as possible on writing. That means, that I of course need CI / CD setup and as I
see no point of hosting my own Git server, I use GitHub and their GitHub Actions:

## .github/workflows/main.yml

```
name: CI
on: push
jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - name: Git checkout
        uses: actions/checkout@v2

      - name: Update theme
        run: git submodule update --init --recursive

      - name: Setup hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: "0.64.0"

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: appleboy/scp-action@master
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.KEY }}
          port: ${{ secrets.PORT }}
          source: "public/"
          target: "/var/www/pdostal.cz/"
```

This file is pretty self explaining, it runs each time I push to the repository
where I have my Hugo source code and it does those steps:

 1) Check out my repository
 2) Initiate Git submodules (the theme)
 3) Install Hugo
 4) Build the site
 5) SCP the site to my server

## Server

Of course we need to also prepare the server, I created separate user
with `scponly` shell and no password but **keep in mind that this user
will be reachable using RSA key storred in GitHub Secrets** so it's up
to you whether you trust GitHub Secrets. Here is my setup:

```
# zypper ar https://download.opensuse.org/repositories/security/openSUSE_Leap_15.2/security.repo
# zypper in scponly
# useradd -s /usr/bin/scponly -m -d /var/www/pdostal.cz/ deploy_pdostal.cz
# sudo -u deploy_pdostal.cz ssh-keygen -t rsa
# sudo -u deploy_pdostal.cz cp /var/www/pdostal.cz/.ssh/{id_rsa.pub,authorized_keys}
```

Now we know all of our secrets which we need to add to GitHub Settings Secrets section:

 * `HOSTNAME`: domain name of the web server
 * `PORT`: by default 22
 * `USERNAME`: the username of the user we created
 * `KEY`: The content of `/var/www/pdostal.cz/.ssh/id_rsa.pub`

## Result

This setup automates the building and publishing process of my site. After each push I can
expect a directory in `/var/www/pdostal.cz/public/` containg up-to-date version of my site.

I wrote a [blog post]({{< ref "nginx-certbot" >}}) about my Nginx and Certbot setup which
I use to host this site.
