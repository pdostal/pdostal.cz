---
title: DNSSEC Monitoring
date: 2024-06-01
tags:
  - linux
---

Do you have your own public domain name? Do you manage your own DNS server(s)? And do you have DNSSEC enabled?
If you answered yes to all of those questions, then you probably want to monitor your DNSSEC setup. This post will show you how I do it.

<!--more-->

Everyone who has experience with DNSSEC deployment probably knows [DNSViz](https://dnsviz.net) and [DNSSEC-Debugger](https://dnssec-debugger.verisignlabs.com)
but I wanted to have a more automated solution that I can run on my own server and after thinking about it for a while I realized that it's just another YAML
snippet to my belowed [URLWatch](https://urlwatch.readthedocs.io/).

## URLWatch

URLWatch is a tool originally for monitoring web pages for changes, but as it grew it become way more flexible. It can handle CSS, Json and even JavaScript if you attach a browser to it.
But it can also monitor the output of shell commands and with it's built-in Filters and Reporters it can be highly customized and it's output can be sent pretty much anywhere (e-mail, Slack, Matrix, etc.).

## Installing URLWatch

As it's Python open source tool, it can be installed via pretty much every package manager, f.e. `sudo zypper in urlwatch` will do the job on openSUSE.

After installing the package you need to create `~/.config/urlwatch/urlwatch.yaml` configuration file. Here is a simple example:

```yaml
display:
  error: true
  new: true
  unchanged: false
report:
  email:
    enabled: true
    from: 'pdostal@pdostal.cz'
    html: true
    method: smtp
    smtp:
      host: 'smtp.gmail.com'
      port: 587
      starttls: true
      auth: true
      user: 'user@gmail.com'
      insecure_password: 'your_password_here'
    subject: '{count} changes: {jobs}'
    to: 'pdostal@pdostal.cz'
```

## Monitoring DNSSEC

For monitoring DNSSEC I decided to use `delv` which is pretty much the standard tool for DNSSEC validation.
You can install it via your package manager, f.e. `sudo zypper in bind-utils` on openSUSE.

To validate your DNSSEC setup you can use the following command:

```bash
$ delv chalupakadov.cz @1.1.1.1
; fully validated
chalupakadov.cz.        81278   IN      A       62.171.184.19
chalupakadov.cz.        81278   IN      RRSIG   A 13 2 86400 20250608144012 20250525131012 29363 chalupakadov.cz. 5AlgjePgGQaBYYyJ3YKDjzlANKSY/BiPOEaSh+rSbbpk5Ykf0vCreU8A VTay2pYyOretC+j0rqm67fd5582KlQ==
```

**Warning:** Do not use `delv` against your own DNS server as it may give you false positive results, always use public DNS server for validation.

## Letting URLWatch to do the job for us

All the jobs URLWatch runs are defined in `~/.config/urlwatch/urls.yaml` file, so let's create a job for our DNSSEC validation:

```yaml
---
name: "delv chalupakadov.cz"
command: "/usr/bin/delv chalupakadov.cz @1.1.1.1 +rtrace +vtrace +short"
filter:
  - re.sub:
      pattern: "keyid=[0-9]*"
      repl: " keyid=XXX"
```

Now every time we run `urlwatch` it will run the `delv` command for us and send us a `diff` if the output changed.

Note: The `+rtrace` and `+vtrace` options are used to get the full validation trace, which is useful for debugging.
The `+short` option is to not print the final `RRSIG` record which changes regularly. And the `re.sub` filter is used to hide the `keyid` as it changes often as well.

Let's try it out:

```bash
$ urlwatch
===========================================================================
01. CHANGED: delv chalupakadov.cz
02. NEW: delv chalupa-vlasta.cz
===========================================================================
```

## Automating the execution

To automate the execution of URLWatch you can use `cron` or `systemd` timer. I prefer the latter, so let's create a systemd timer for it.

First of all we need the service that will run the URLWatch. Let's create a file `~/.config/systemd/user/urlwatch.service` with the following content:

```ini
[Unit]
Description=Watch selected URLs for changes

[Service]
ExecStart=/usr/bin/urlwatch
```

Now we need to create the timer that will run the service periodically. Create a file `~/.config/systemd/user/urlwatch.timer` with the following content:

```ini
[Unit]
Description=Periodically watch selected URLs for changes

[Timer]
OnCalendar=*:0/30
Persistent=true

[Install]
WantedBy=timers.target
```

Now we reload systemd, enable and start the timer and see if it's configured properly:

```bash
$ systemctl --user daemon-reload
$ systemctl --user --now enable urlwatch.timer
$ systemctl --user list-timers
NEXT                             LEFT LAST                               PASSED UNIT                                               ACTIVATES
Sun 2025-06-01 23:30:00 CEST    12min Sun 2025-06-01 23:00:00 CEST    17min ago urlwatch.timer                                     urlwatch.service
```

# Conclusion

With this setup I can easily monitor my DNSSEC setup and as I use URLWatch for other purposes as well, I didn't have to reinvent the wheel.

