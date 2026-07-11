---
layout: post
title: "Data Oversharing Risk"
date: 2026-07-12
categories: ["AI & Identity", "Copilot Security"]
tags: ["oversharing", "data-governance", "m365", "copilot"]
author: pankaj
---
Oversharing existed long before Copilot did. Every organization I've read about or poked at in a lab has some version of the same story: a SharePoint site created for a project that ended, still shared "everyone in the company," sitting there because the offboarding checklist for that project never included a step to revoke the sharing link. That kind of sprawl was tolerable for years because the practical risk was bounded by how hard it is for a human to stumble onto the wrong file. Search tools existed, but keyword search against millions of documents mostly returns noise unless you already roughly know what you're looking for. AI assistants change that calculus completely, because they don't need you to know what you're looking for - they just need you to ask a question in plain language, and they'll go find whatever's technically accessible that seems relevant.

## Why this is a different category of risk, not just more of the same

The instinct is to treat this as the same oversharing problem organizations have had for years, just with a faster search tool bolted on. I think that undersells it. The difference is that keyword search requires the searcher to guess the right words, while an AI assistant can be handed an intent - "what's our churn rate by region" - and it will independently reason its way to documents that never used those exact words but are conceptually relevant, synthesizing them into a confident answer. In a lab test I ran, a spreadsheet named with an internal project codename that had zero descriptive text in its filename still got pulled into a Copilot answer about departmental budgets, because the assistant read the actual cell contents rather than relying on filename matching the way a naive keyword search would. That's a genuinely new capability being pointed at old, unmanaged sharing settings, and the combination is what makes this urgent rather than merely inconvenient.

## Where the exposure actually accumulates

Oversharing tends to accumulate through completely reasonable, individually small decisions: a manager shares a folder broadly because it's faster than managing individual permissions for a growing team, a departing employee's OneDrive gets bulk-transferred to a shared location without anyone re-scoping the sharing settings, a Teams site inherits permissive defaults because the person creating it accepted whatever the template pre-selected and moved on. None of these are malicious or even obviously careless in isolation. The risk only becomes visible once something starts querying across all of it simultaneously and summarizing what it finds in a way that reads as authoritative, regardless of whether the content should have been that reachable in the first place.

## What actually reduces the exposure

The tooling that helps here isn't really about Copilot configuration at all - it's the unglamorous data governance work that should have happened regardless: running sensitivity label coverage reports, using tools like Microsoft Purview's data access governance capabilities to actually surface which SharePoint sites and OneDrive accounts have broad, unused sharing grants, and doing the tedious work of walking those down before flipping on an AI assistant tenant-wide. In my own test environment, running an access review against sites with organization-wide sharing surfaced far more stale grants than I expected going in, and every one of them became a genuine exposure the moment I let Copilot query against that same tenant.

The uncomfortable truth is that most organizations rolling out Copilot are really running an involuntary audit of years of sharing decisions that outlived the projects and people who made them, and the AI assistant isn't creating the risk so much as it's the first tool with enough reach and fluency to actually make that risk cash out into some