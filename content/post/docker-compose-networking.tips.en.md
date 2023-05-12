---
title: docker-compose networking tips
date: 2023-02-24
tags:
  - linux
---

TL;DR: This article is so far about how to disable iptables integration and choose different addressing for the 'bridge' (default) network.

<!--more-->

## docker default network ('bridge')

There are two defaults I don't like:
1) Default network has `172.17.0.0/16` address space
2) Docker is altering the iptables configuration automagically

The solution is custom `/etc/docker/daemon.json` file:
```json
{
  "bip": "100.64.236.1/24",
  "iptables": false
}
```

## docker-compose

By default `docker-compose` creates dedicated network for each project.
Addressing of those dedicated networks is again under `172.17.0.0/16` prefix.

### New solution

As mentioned on [docker/compose#4336](https://github.com/docker/compose/issues/4336#issuecomment-457326123)
we can instruct Docker which prefix should those additional networks use:

```yaml
# cat ~/.docker/daemon.json
{
  "debug" : true,
  "default-address-pools" : [
    {
      "base" : "172.31.0.0/16",
      "size" : 24
    }
  ]
}
```

### Previous solution
This problem is fixable by forsing each service in our project to joing the
default network:

```yaml
version: "3.1"

services:
  app:
    image: registry.opensuse.org/opensuse/leap:15.4
    network_mode: bridge
```
