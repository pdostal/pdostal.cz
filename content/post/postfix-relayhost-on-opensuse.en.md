---
title: Postfix relayhost on openSUSE
date: 2022-06-25
tags:
  - linux
---

This will describe how to make your openSUSE machine send e-mail via reliable
relayhost of your choice using Postfix. I will use gmail as an example.

<!--more-->

 1) Installing required packages

    ```
    zypper in postfix cyrus-sasl libsasl2-3 cyrus-sasl-plain cyrus-sasl-gssapi
    ```

 2) Configure Postfix:

    ```
    cp /etc/postfix/main.cf{,_bak}
    cat <<EOT >> /etc/postfix/main.cf
    compatibility_level = 3.6
    queue_directory = /var/spool/postfix
    command_directory = /usr/sbin
    daemon_directory = /usr/lib/postfix/bin/
    data_directory = /var/lib/postfix
    mail_owner = postfix
    inet_interfaces = 127.0.0.1,::1
    setgid_group = maildrop
  
    relayhost = [smtp.gmail.com]:587
    smtp_use_tls = yes
    smtp_sasl_auth_enable = yes
    smtp_sasl_security_options =
    smtp_sasl_password_maps = lmdb:/etc/postfix/sasl_passwd
    smtp_tls_CAfile = /etc/ssl/ca-bundle.pem
    EOT
    ```

 3) Create `sasl_passwd` file and 

    ```
    echo "[smtp.gmail.com]:587 $USER@gmail.com:$PASSWORD" >> /etc/postfix/sasl_passwd
    postmap /etc/postfix/sasl_passwd
    ```

 4) Restart the systemd service

    ```
    systemctl enable --now postfix.service
    ```

 5) Test it out

    ```
    echo "This is just a test." | mail -r "quiiicq@gmail.com" -s "A test of the system mail system" -Ssendwait pdostal@pdostal.cz
    ```

    The `-Ssendwait` makes the `mail` program wait until the e-mail is really submited to the queue instead of doing it in background.

