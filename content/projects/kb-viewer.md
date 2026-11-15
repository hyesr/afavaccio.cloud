---
title: "kb-viewer"
description: "A private web viewer for a folder of Markdown notes, built with security as the first requirement."
date: 2026-10-17
---

A private web viewer for a folder of Markdown notes: full-text search,
backlinks, a graph view and wikilinks. The viewer is
[Quartz](https://quartz.jzhao.xyz); the project is the platform work around
it.

It runs in two hardened containers on my own machine: one builds the site and
has no network at all, the other is nginx serving static files behind a strict
Content Security Policy. A Helm chart for the
[homelab]({{< relref "/projects/homelab" >}}) comes next.

Docker and nginx. Not public yet.
