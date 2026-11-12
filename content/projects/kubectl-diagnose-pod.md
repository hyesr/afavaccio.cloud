---
title: "kubectl diagnose-pod"
description: "A kubectl plugin that explains what is wrong with a pod in one read-only report."
date: 2026-10-18
---

A kubectl plugin that puts everything about a pod in one report: what is
wrong, why, what changed, and what to run next. It gathers what `describe`,
`logs`, `events` and `rollout history` would tell you, runs 29 rules over it
and prints ranked findings with the commands to try.

It is read-only by design: it sends only GET and HEAD requests, never reads
Secret values and writes nothing to disk.

Written in Go. Not public yet.
