---
layout: post
title: "Service Principal for AI Workloads"
date: 2026-07-12
categories: ["AI & Identity", "Identity in AI Pipelines"]
tags: ["service-principal", "azure-ad", "ai-pipelines", "identity"]
author: pankaj
---
Setting up a service principal for an AI pipeline in my lab tenant felt, at first, like the same task I'd done a hundred times for regular application integrations - register the app, grant it API permissions, generate a credential, wire it into the pipeline config. It took building the actual pipeline end to end before I noticed the ways an AI workload strains that pattern differently than a traditional service integration does, mostly around how many distinct downstream calls a single AI pipeline ends up making and how much broader the resulting permission footprint tends to be if you don't fight it deliberately.

## Why AI pipelines accumulate broader permissions than typical apps

A traditional service integration usually has a narrow, well-defined job - sync data from system A to system B, on this schedule, using these fields. Its service principal's permission needs are correspondingly narrow and stable over time. An AI pipeline, especially one built around retrieval-augmented generation or agentic orchestration, often needs to reach a wide and evolving set of data sources because the whole point is flexibility - pull from SharePoint today, a database tomorrow, a third-party API next sprint as the use case expands. I watched this happen in my own test pipeline: what started as Files.Read against one document library grew, over a few weeks of iteration, into Files.Read.All, Sites.Read.All, and a Graph API scope I'd forgotten I'd added, none of which got removed when the corresponding feature was later cut, because pruning a permission set that's still working ranks below whatever's on this sprint's board, and I never opened a ticket for it either.

## The single-service-principal trap

The most common pattern I've seen described (and reproduced myself for convenience) is standing up one service principal for an entire AI pipeline and reusing it across every stage - ingestion, embedding generation, retrieval, and the generation step that actually talks to the model. That's operationally simple and exactly the wrong shape from a blast-radius standpoint, because a vulnerability or misconfiguration in the generation step (the part most exposed to untrusted user input and prompt injection risk) inherits the full permission set that the ingestion step needed to read every document in the pipeline's source data. Splitting the pipeline's stages across separate service principals, each scoped to only what that stage requires, is more setup overhead but keeps a compromise of the most exposed component from automatically granting reach into the parts of the pipeline that never should have been touchable from there.

## Credential handling specific to pipelines

AI pipelines also tend to run through orchestration tools - Azure Data Factory, Airflow, custom Python schedulers - that need to store the service principal's credential somewhere accessible to scheduled jobs, and I've seen (and in early lab iterations, done myself) the shortcut of dropping a client secret directly into pipeline configuration rather than pulling it from a proper secrets store like Key Vault at runtime. That shortcut works fine until the pipeline configuration gets exported, shared, or backed up somewhere with looser access controls than the original vault would have had, at which point the "temporary" plaintext secret is sitting in a backup archive or an exported YAML file that outlives the pipeline it was written for, with no expiry and no record of where else it was copied.

The practical discipline that actually holds up is treating each stage of an AI pipeline as its own principal with its own narrow scope, pulling credentials from a managed secrets store rather than pipeline configuration directly, and revisiting the permission set on a real cadence rather than assuming that what was gra