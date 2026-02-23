---
title: Podman static networking with Netavark and Quadlet
date: 2026-02-23
tags:
  - linux
---

TL;DR: This article describes how to configure custom static networking with Podman Quadlet and Netavark as the network backend.

<!--more-->

## Preparation phase

First of all, let's set proper subnet and disablet Podman - NFTables interaction:

```bash
# cat /etc/containers/containers.conf
[network]
default_subnet = "100.64.0.0/24"

# cat /etc/containers/netavark/config.json
{
  "firewall_driver": "none"
}
```

The `100.64.0.0/10` prefix is designed for CGNAT and won't colide with other private networks. See [rfc6598].

## Deployment

Let's deploy each service (NextCloud, Immich, Gitea, ...) to separate isolated pod and therefore let's provide separate isolated network for each pod.

```bash
# cat /etc/containers/system/example.network
[Unit]
Description=Example Network

[Network]
Subnet=100.64.1.0/24
Gateway=100.64.1.1
IPAMDriver=host-local
Label=app=Example
InterfaceName=podman-example
NetworkName=example

[Install]
WantedBy=default.target

# cat /etc/containers/system/example.pod
[Unit]
Description=Example Pod

[Pod]
PodName=example
Network=example
IP=100.64.1.2
Label=app=example

[Install]
WantedBy=multi-user.target

# /etc/containers/system/example-app.container
[Unit]
Description=Example App Container

[Container]
ContainerName=example-app
Image=registry.opensuse.org/opensuse/nginx:latest
Pod=example.pod
Label=app=example

[Service]
Restart=always

[Install]
WantedBy=multi-user.target

# /etc/containers/system/example-db.container
[Unit]
Description=Example DB Container

[Container]
ContainerName=example-db
Image=registry.opensuse.org/opensuse/nginx:latest
Pod=example.pod
Label=app=example

[Service]
Restart=always

[Install]
WantedBy=multi-user.target
```

## Conclusion

Our Example application is reachable via `100.64.1.2` address.

There are of course two important steps missing:

1) Firewall of your choise
2) HTTP(S) Proxy of your choice

[rfc6598]: https://datatracker.ietf.org/doc/html/rfc6598

