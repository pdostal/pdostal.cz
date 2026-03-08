---
title: IPv6 on Linux servers
date: 2023-03-08
tags:
  - linux
  - network
---

TL;DR: This article speaks about IPv6 Stateless Address Autoconfiguration (SLAAC) EUI64 mechanism

<!--more-->

There are basically three ways to configure IPv6 on a Linux server:

1) DHCPv6: This has been unreliable in my experience. By default, you cannot use the MAC address to create a static lease, and the UUID is difficult to find and is not static either.
2) SLAAC: The Stateless Address Autoconfiguration method is essentially the default. On a server, though, I want a single, static, predictable address. Let's cover that below.
3) Manual: We can always configure the address manually. This is of course a valid option and there is nothing more to add about it.

Note: There is always link local address from the `fe80::/10` subnet and that is fine.

## 2) SLAAC:

### Systemd-Networkd

1) Let's first list our connections:
```bash
# networkctl
IDX LINK    TYPE     OPERATIONAL SETUP
  1 lo      loopback carrier     unmanaged
  2 ens18   ether    routable    configured
  3 podman0 bridge   routable    unmanaged
```

2) The configuration part:

```
# /etc/systemd/network/20-ens18.network
[Match]
Name=ens18

[Network]
DHCP=ipv4
Token=eui64
IPv6AcceptRA=yes
IPv6PrivacyExtensions=no
```

The `Token` option sets which SLAAC mechanism will be used to generate the primary IPv6 address.

The `IPv6PrivacyExtensions` option controls whether additional dynamic IPv6 addresses will be assigned.

3) Now we just need to restart the network. Make sure you have another way to access the machine!
```bash
# networkctl reload
# networkctl reconfigure ens18
```

### NetworkManager

1) Let's first list our connections:
```bash
# nmcli con show
NAME     UUID                                  TYPE      DEVICE
ens18    711aee22-6717-3d36-871e-4b14478ca5f3  ethernet  ens18
lo       08efa8c5-780b-4a47-878d-1ef091fe9b32  loopback  lo
podman0  e6cf2685-25c7-464f-93d9-078f283cc37c  bridge    podman0
```

2) The configuration part:
```bash
# nmcli con modify ens18 ipv6.method auto
# nmcli con modify ens18 ipv6.ip6-privacy disabled
# nmcli con modify ens18 ipv6.addr-gen-mode eui64
```

The `ipv6.addr-gen-mode` option sets which SLAAC mechanism will be used to generate the primary IPv6 address.

The `ipv6.ip6-privacy` option controls whether additional dynamic IPv6 addresses will be assigned.

3) Now we just need to restart the network. Make sure you have another way to access the machine!
```bash
# nmcli con down ens18; nmcli con up ens18
```

## Final thoughts

I like EUI64 because the IPv6 address is predictable. I don't have to configure both DNS and the system manually — I just keep track of the NIC's hardware address, and the only place I need to make changes is DNS. I'd like to use DHCPv6 static leases, but until that becomes reliable I'll probably stick with this method.
