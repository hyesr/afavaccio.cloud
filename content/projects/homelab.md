---
title: "Homelab"
description: "A three-node Kubernetes cluster on a Mac mini, built from code and managed with GitOps."
date: 2026-10-17
weight: 1
repo: "https://github.com/hyesr/homelab"
---

A three-node Kubernetes cluster on a Mac mini (M4, 16 GB), built entirely from
code and managed with GitOps. Early stage: the VMs, CI, the K3s cluster, Flux,
the internal CA and a first app (a dashboard) work.

I wrote about the design in
[A Kubernetes homelab on a Mac mini]({{< relref "/blog/homelab-kubernetes-on-a-mac-mini" >}}).

## Stack

- [Lima](https://lima-vm.io): three Ubuntu arm64 VMs
- [Ansible](https://docs.ansible.com): installs and configures K3s
- [K3s](https://k3s.io): one control plane and two workers
- [Flux](https://fluxcd.io): reconciles the cluster from the repository
- [cert-manager](https://cert-manager.io): internal CA limited to `home.arpa`
- [Homepage](https://gethomepage.dev): the dashboard
- GitHub Actions: lints every push and validates the Kubernetes manifests
