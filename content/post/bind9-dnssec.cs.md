---
title: Odstranění starých DNSSEC klíčů na serveru bind9
date: 2024-02-16
tags:
  - linux
---

Dodatek k článku [DNSSEC s BIND 9.9 snadno a rychle](https://www.root.cz/clanky/dnssec-s-bind-9-9-snadno-a-rychle/) od Ondry Caletky, který vyšel na serveru [root.cz](https://root.cz/).

<!--more-->

V Listopadu jsem vygeneroval nový ECDSA klíč a správce domény aktualizoval DS záznam u CZ NICu, ale ani v Únoru staré RSA klíče nezmizeli, viz:

```
$ dig +short nmnm.cz DNSKEY @ns01.nmnm.cz +dnssec
257 3 5 AwEAAcT5Z/OZFCmoXlQVmMp0gVLNgITykOwRqzdfrrgxtd2gz1LuRNTx /8D/2eeWmnlVQEv873IwF/gZqG9Jm7X7bZpzOXc6zBeYSMPYfdKZVoQe VdY8NWMFZGjOynq9JvWB7CWGSj+53QhF+2ttPR4+HH3SYaHL+N9J965I D0X2nU
/3bg+brzJswKTYRAUMXIrvFCYUpEMOpsuuDc1s5z7WVoaM+6fU iZ2FYzi0OTTgJgV1JjvNUe1eatVTKqpVWgcJpq4XuC3RhAta6+3a4SeJ rVRoAzKRJ32Kmzub3XnxhksVqAZ8+lZN4+nXroUlhpLetO7UOkQwIWjf OIJ48tFy6CE=
256 3 5 AwEAAbL6SEuzMofj3Nv254H+2ir/mywULM/Z6IiiaKYms/RB4TcUmrp8 QsIgn0XowmcoBXeLeAxM6ydNtl4X4oklrrwqq2Vh7kXNi4rMnHRHZWMX BnTkXoYi9RYhhUKaC6MxG0gLvGgkCiTHglOeL2Arzy4q+RQaT6z3TRn5 BJqVjHndhYcmEOG9yuVDkP28ZMjVZZfTJsM91d9Ycaq9oWPoDe817JgT onRfugwwMNpHH8g9XThVAOD/Z0EsbCiZHfPZCFMSQCkzZkv2On8QP/nD RkZKTXAmpmKK53ybyuSj1taBdcXTOdFe1DATJTcnx4XTBtzYOnzTxrtx 97meQVUNDEc=
257 3 13 l8uzAfa94MtbhKoaj7DTO3dB3xRBY4LeYP8r69shrz+nTfpMh1rLvutk atFxWuubvvHeV5r0WnJEBy06d4ZSYg==
DNSKEY 13 2 86400 20240317091356 20240216081357 13968 nmnm.cz. XD7s7wczVGXKxo+gBKiNHHqnKkZDUOFkVuEd4yU4Y3ZT6ijqzdwmLzng RR/UQLfL0Dv1cQG0/mTRySLEhGIPjg==
```

Vzhledem k tomu, že nový klíč již nějakou dobu funguje, rozhodl jsem se staré klíče odstranit manuálně:

```
# rndc signing -list nmnm.cz
Done signing with key 76/RSASHA1
Done signing with key 16272/RSASHA1
Done signing with key 13968/ECDSAP256SHA256
# rndc sign nmnm.cz
# rndc reload nmnm.cz
# rndc signing -list nmnm.cz
Done signing with key 13968/ECDSAP256SHA256
```

A konečně se dostavil kýžený výsledek:

```
$ dig +short nmnm.cz DNSKEY @ns01.nmnm.cz
257 3 13 l8uzAfa94MtbhKoaj7DTO3dB3xRBY4LeYP8r69shrz+nTfpMh1rLvutk atFxWuubvvHeV5r0WnJEBy06d4ZSYg==
```

V `journalctl -fu named` jsem žádnou chybu nenašel a jak [dnsviz.net](https://dnsviz.net/) tak [dnssec-analyzer.verisignlabs.com](https://dnssec-analyzer.verisignlabs.com/) jsou čistě zelené.

