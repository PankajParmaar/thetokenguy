---
layout: post
title: "Sentinel Architecture"
date: 2026-07-12
categories: ["Microsoft Defender", "SIEM & Sentinel"]
tags: ["microsoft sentinel", "siem architecture", "log analytics"]
author: pankaj
description: "Sentinel gets sold as a SIEM you just turn on, and functionally it is — but the architecture has dependencies that surface once you design the workspace."
image:
  path: /assets/img/og-default.png
  alt: "Sentinel Architecture"
---
Sentinel gets sold in slide decks as a cloud-native SIEM you can just turn on, and functionally that's true, but the architecture underneath it has a few dependencies that aren't obvious until you're the one designing the workspace layout for a real deployment.

## It's built on Log Analytics, not a separate database

Sentinel isn't a standalone product with its own storage engine, it's an application layer on top of a Log Analytics workspace, the same underlying service Azure Monitor uses. That matters practically because every design decision you'd make for a Log Analytics workspace, retention period, data residency region, workspace-based versus resource-based access control, applies directly to your Sentinel deployment, and Sentinel-specific documentation sometimes glosses over the fact that you're really configuring Azure Monitor infrastructure with a security layer on top of it.

This also means the cost model inherits Log Analytics' ingestion-based pricing directly, per gigabyte ingested per day, plus Sentinel's own analytics charge on top of that ingestion cost. A single noisy data connector, like verbose Windows security event logging turned on across a large server fleet without any filtering, can quietly become the majority of a monthly bill before anyone notices, since ingestion cost doesn't announce itself until the invoice arrives.

## Single workspace vs multi-workspace

The single biggest architectural decision, and the one hardest to reverse after the fact, is whether to run a single Sentinel workspace tenant-wide or split across multiple workspaces by business unit, region, or compliance boundary. A single workspace makes cross-entity correlation and hunting dramatically simpler, since every table lives in one place and a KQL query doesn't need cross-workspace joins. Multiple workspaces make sense when data residency law requires logs from one region to physically stay there, or when a business unit has genuinely separate compliance and access requirements that a single shared workspace with row-level security can't cleanly satisfy.

Cross-workspace queries are technically possible using the `workspace()` function in KQL, but they add real query complexity and, in larger deployments, real latency, so treating multi-workspace as the default without a specific driver requiring it usually creates more operational overhead than it solves.

## Data connectors and normalization

Data flows into Sentinel through connectors, ranging from native Azure service connectors that require no separate agent, to the Azure Monitor Agent for Windows and Linux servers, to Syslog/CEF for on-prem and third-party network devices, to direct API-based connectors for many SaaS platforms. Underneath, Sentinel increasingly leans on the Advanced Security Information Model, a normalized schema that lets a single analytics rule or hunting query work across differently-sourced data, like authentication events from Azure AD and from an on-prem AD, without writing separate logic for each source's raw schema.

## Where the architecture gets painful

The most common regret I've seen described in community writeups is under-planning retention and archive tiers up front. Interactive retention is expensive to keep long, and Sentinel supports a cheaper archive tier for data you need to retain for compliance but query rarely, yet plenty of workspaces get set up with a single retention policy for everything because the initial deployment ticket just said "stand up Sentinel" with no line item for splitting hot query data from cold compliance data at design time.

Getting Sentinel architecture right up front, workspace boundaries, retention tiers, and connector scoping, saves far more pain later than any individual detection rule ever will, because unwinding a bad workspace design after data has been flowing for a year is a genuinely painful migration, not a 