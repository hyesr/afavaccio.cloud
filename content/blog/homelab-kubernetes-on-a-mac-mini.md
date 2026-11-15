---
title: "A Kubernetes homelab on a Mac mini"
description: "Three K3s nodes on one Mac mini, built from code with Lima and Ansible and run with Flux: what works so far and the choices behind it."
date: 2026-11-12
---

I am building a home server: a three-node Kubernetes cluster on a Mac mini
(M4, 16 GB), created entirely from code and managed with GitOps. The
repository is [hyesr/homelab](https://github.com/hyesr/homelab). It is at an
early stage, and this post covers what runs today and why it is built the way
it is.

## The stack

| Layer | Tool | Role |
| --- | --- | --- |
| Virtual machines | [Lima](https://lima-vm.io) | Three plain Ubuntu arm64 VMs that can reach each other |
| Provisioning | [Ansible](https://docs.ansible.com) | Installs and configures K3s on the VMs |
| Kubernetes | [K3s](https://k3s.io) | One control plane and two workers |
| GitOps | [Flux](https://fluxcd.io) | Reconciles the cluster from the repository |
| TLS | [cert-manager](https://cert-manager.io) | Internal certificate authority for the apps |
| First app | [Homepage](https://gethomepage.dev) | A dashboard for the homelab |

Each VM has 2 vCPUs, 2 GiB of RAM and a 20 GiB disk. The whole cluster comes
up with one command:

```bash
make up                        # create the VMs, install K3s
eval "$(make -s kubeconfig)"   # point kubectl at the cluster
kubectl get nodes              # three nodes, all Ready
```

## Real nodes, one machine

I wanted a real multi-node cluster, where each node is a full operating
system, and not containers pretending to be nodes. Lima runs arm64 Ubuntu VMs
natively on Apple Silicon and puts them on a network where they reach each
other by name.

The trade-off is plain: it is one physical machine, so there is no real high
availability. Three nodes teach scheduling and failure handling, but they do
not survive a hardware failure.

## Lima creates, Ansible installs

Lima templates can install K3s on their own, and that would be quicker. I kept
Lima to creating plain Ubuntu VMs and gave the installation to Ansible, so the
same playbooks can later run unchanged on a mini PC or a VPS. It is more code,
and I maintain the install logic myself.

The workers join with an agent-only token that the server generates and the
playbook reads at run time. It is never stored in Git, and unlike the server
token it cannot be used to take over the cluster.

## Flux without `flux bootstrap`

Flux fetches the repository every minute and applies what is under
`clusters/homelab/`. Its own manifests live there too, so upgrading Flux is a
pull request like any other change.

I did not use `flux bootstrap`. It needs a personal access token that can push
to the repository, and it commits straight to `main`, which my branch rules
reject: every change goes through a pull request and a lint check. So I
generated the same manifests with the Flux CLI, merged them through a pull
request, and created the SSH key inside the cluster. GitHub holds only the
public half, as a read-only deploy key.

The cost is a first install of a few manual commands instead of one, and a new
deploy key after every rebuild, because the private key exists only in the
cluster.

## An internal CA that can only sign `home.arpa`

The apps use `home.arpa`, the domain reserved for home networks. No public CA
issues certificates for it, so cert-manager runs a CA inside the cluster.

A device that trusts a CA accepts whatever it signs, and this CA's key is a
Secret in the cluster. To limit the damage if the cluster were compromised,
the CA carries X.509 name constraints: `home.arpa` only, no IP addresses. I
tested it with OpenSSL and the macOS verifier: `dashboard.home.arpa` passes,
`github.com` and an IP address are rejected.

## The first app

The first app is Homepage, a start page for the homelab. It is stateless, so
it needs no persistent volume or backup, yet it exercises the whole path:
Flux, Ingress through Traefik, a certificate from the internal CA and a
NetworkPolicy. New apps add themselves to it with annotations on their
Ingress.

Its login is off for now, because the password would be the first secret in
the repository and secret encryption is not set up yet. Until then the
dashboard is reachable only from the Mac mini itself.

## What comes next

The roadmap is ticked only for things that actually run. Still open:

- Reach the apps from the LAN
- Pi-hole
- Secrets with SOPS and age
- Backups of the persistent volumes
- Lightweight monitoring
- Gateway API instead of Ingress
- A kubeadm-based cluster built the same way, to compare with K3s

Every choice above, with its trade-offs, is recorded in
[docs/decisions.md](https://github.com/hyesr/homelab/blob/main/docs/decisions.md).
