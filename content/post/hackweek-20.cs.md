---
title: Hackweek 20
date: 2021-03-26
tags:
  - linux
  - hackweek
---

Nedávno proběhl dvacátý [SUSE Hack Week](https://hackweek.suse.com)
a já si konečně našel čas na nasazení Kubernetes pomocí dvou nástrojů
vyvíjených Rancher Labs.

Celý experiment jsem napsal jako multi machine test do openQA. Kód
najdete na [GitHub]u a je plánováno jej nasadit na openqa.opensuse.org.

<!--more-->

## Resources:
 * [k3s webpage](https://k3s.io/)
 * [k3s github repository](https://github.com/k3s-io/k3s)
 * [rke1 github repository](https://github.com/rancher/rke)
 * [rke1 cluster.yaml](https://rancher.com/docs/rke/latest/en/example-yamls/#minimal-cluster-yml-example)
 * [rke1 deploy guide](https://rancher.com/docs/rke/latest/en/installation/#prepare-the-nodes-for-the-kubernetes-cluster)

Projekt byl inspirován workshopem [SUSE @Home].

[SUSE @Home]: https://github.com/SUSE/suse-at-home
[experiment]: https://hackweek.suse.com/20/projects/create-openqa-multimachine-tests-for-deploying-kubernetes-on-tumbleweed-using-both-k3s-and-rke1
[GitHub]: https://github.com/os-autoinst/os-autoinst-distri-opensuse/pull/12200
