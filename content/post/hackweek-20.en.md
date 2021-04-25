+++
title = "Hackweek 20"
date = "2021-03-26"
+++

Rancher maintains 2 Kubernetes distribution, both production grade:

 * k3s: Single binary containing containerd backend. Can be run as
   server or agent on each node of cluster. SQLite is used by default
   instead of ETCD. MySQL and Postgres also supported.
   No prerequisity. Intended for edge computing.
 * RKE: Deployed remotely over SSH on top of Docker which is the only
   required component of the underlying OS. Standard ETCD database
   in container. Each node and its roles are defined in YAML.
   Resource hungry, datacenter grade.

<!--more-->

## Each scenario needs:
 1) Support Server: As Internet gateway, DHCP and DNS server.
 2) Master node for the control node and it's database
 3) ‎4. Worker nodes joining the cluster.

## TO-DO:
 * Run more tests on top of the Kubernetes cluster.
 * Implement the same for rke2 (not in production yet).

## Verification runs:
 * [rke1](http://pdostal-server.suse.cz/tests/11596)
 * [k3s](http://pdostal-server.suse.cz/tests/11592)

## Resources:
 * [k3s webpage](https://k3s.io/)
 * [k3s github repository](https://github.com/k3s-io/k3s)
 * [rke1 github repository](https://github.com/rancher/rke)
 * [rke1 cluster.yaml](https://rancher.com/docs/rke/latest/en/example-yamls/#minimal-cluster-yml-example)
 * [rke1 deploy guide](https://rancher.com/docs/rke/latest/en/installation/#prepare-the-nodes-for-the-kubernetes-cluster)

This project was inspired by the awesome [SUSE @Home](https://github.com/SUSE/suse-at-home) workshop.
This was done as [HackWeek 20](https://hackweek.suse.com/20/projects/create-openqa-multimachine-tests-for-deploying-kubernetes-on-tumbleweed-using-both-k3s-and-rke1) project.
The [pull request](https://github.com/os-autoinst/os-autoinst-distri-opensuse/pull/12200) has been opened in os-autoinst-distri-opensuse repository.
