---
layout: post
title: "Workload Identities"
date: 2026-07-12
categories: ["AI & Identity", "Non-Human Governance"]
tags: ["workload-identity", "managed-identity", "non-human-identity", "cloud-security"]
author: pankaj
description: "The moment a client secret in a config file gets replaced by a managed identity is a small, real relief — one less credential to rotate or leak by accident."
image:
  path: /assets/img/og-default.png
  alt: "Workload Identities"
---
The moment I stopped storing a client secret in a config file and switched a test workload over to a managed identity, I felt a small but real sense of relief - one less credential I had to remember to rotate, one less string that could leak if I accidentally committed a file I shouldn't have. That's the appeal of workload identity in one sentence: it lets a piece of compute prove who it is using something the platform itself attests to, rather than a secret a human has to generate, store, and eventually forget to rotate.

## What a workload identity actually is

A workload identity ties an application's identity to the underlying compute platform rather than to a static credential. In Azure that's a managed identity bound to a VM, function app, or container; in AWS it's an IAM role assumed via an instance profile or, for Kubernetes, IAM Roles for Service Accounts (IRSA); in GCP it's workload identity federation letting a Kubernetes service account impersonate a Google service account. The common thread is that the platform itself vouches for the workload's identity through infrastructure the workload runs on, so there's no secret sitting in a repo, a pipeline variable, or an environment file for someone to find. The credential is short-lived, issued by the platform at runtime, and tied to conditions (which node, which namespace, which service account) that are much harder to smuggle out of an environment than a static key.

## Where it breaks down in practice

The gap I keep running into, even in my own small lab setups, is that workload identity solves the secret-storage problem cleanly but doesn't automatically solve the scoping problem. It's entirely possible to bind a managed identity to a function app and then grant that identity Contributor on the whole subscription because it was faster than writing a custom role during testing. The credential handling improved; the blast radius didn't. I've also seen (and made) the mistake of using the same managed identity across multiple workloads that have nothing to do with each other functionally, because provisioning a new identity for every function felt like unnecessary overhead - which means a bug in one workload's code path can be used to pivot into whatever the shared identity can reach, defeating a good chunk of the isolation workload identity is supposed to provide.

## Cross-cloud and cross-platform gaps

Workload identity also tends to be strongest within a single cloud's own ecosystem and weaker at the boundaries. A workload running in Kubernetes that needs to call an external SaaS API, or a workload in one cloud needing to authenticate to a resource in another, usually falls back to some form of static credential or a federation setup that has to be configured carefully to avoid becoming another broad, poorly scoped trust relationship. I tested a workload identity federation setup between a Kubernetes cluster and Google Cloud once, and the default guidance I found online suggested binding the identity pool at the namespace level - which is workable, but anyone rushing through the setup could just as easily bind it at the cluster level and grant every pod in the cluster the same downstream access, and the resulting misconfiguration wouldn't look obviously wrong unless you already knew to check for it.

Workload identity is a genuine improvement over static secrets, and it removes an entire class of credential-leakage risk almost for free, but it's not a substitute for actually scoping what the workload can do once it authenticates - and treating it as a finished security control rather than half of one is exactly how a well-intentioned migration away from secrets quietly recreates the same over-permissioned sprawl in a shinier wrapper.
