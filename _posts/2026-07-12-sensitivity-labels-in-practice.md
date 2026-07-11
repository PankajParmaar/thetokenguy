---
layout: post
title: "Sensitivity Labels in Practice"
date: 2026-07-12
categories: ["Microsoft 365", "SharePoint & OneDrive"]
tags: ["sensitivitylabels", "purview", "informationprotection", "sharepoint"]
author: pankaj
---
Sensitivity labels look straightforward in the Purview compliance portal: define a label, attach some protection settings, publish it to users. What's less obvious until you've actually deployed a few is how differently a label behaves depending on what it's applied to — a Word document, an email, a SharePoint site, or a Teams container all treat the same label name in meaningfully different ways, and conflating them is where deployments go sideways.

## Document and email labels

At the file level, a sensitivity label can apply visual markings (headers, footers, watermarks), enforce encryption tied to specific permissions (view-only, no forward, no print), and control whether the file can leave the organization at all. This is the layer people usually picture when they hear "sensitivity label" — a "Confidential" label on a Word doc that encrypts it so only named individuals or the organization can open it, even if the file gets emailed outside the tenant by mistake. The protection travels with the file itself, which is the real value proposition: unlike a SharePoint permission, an encrypted label survives the document being downloaded, copied, and emailed elsewhere, because the rights are baked into the file's protection metadata, not into a permission list somewhere in SharePoint.

Auto-labeling based on content (a document containing what looks like a credit card number, or matching a custom sensitive information type) is worth turning on but deserves a pilot period, because auto-labeling that's too aggressive results in constant business-appropriate documents getting flagged, and users learn to override the suggested label reflexively, which defeats the entire point of automatic classification.

## Container labels behave completely differently

Applied to a Microsoft 365 Group, a Team, or a SharePoint site, a sensitivity label doesn't encrypt anything — there's no file-level protection at the container level. Instead it controls container-level settings: whether the site or Team allows guest access at all, whether it's public or private, and whether external sharing is even possible for content inside it. A "Highly Confidential" container label might simply force a Team to be private and block guest access entirely, which is a coarser but genuinely useful control, since it means every file inside that Team inherits a baseline restriction without anyone having to remember to individually label each document.

The mistake I've seen made more than once is expecting a container label to do what a document label does — assuming that labeling a whole SharePoint site "Confidential" somehow encrypts every file inside it. It doesn't. Container labels and document labels are separate label taxonomies that happen to share the labeling UI, and a deployment plan needs to treat them as two different tools solving two different problems, not one label type doing double duty.

## Rollout order matters more than label design

The label taxonomy itself — how many tiers, what each is called — matters less than most initial workshops spend time on. What actually determines whether a label rollout succeeds is sequencing: publish labels in audit-only mode first, watch actual usage patterns through the label analytics in Purview, then move to mandatory labeling (forcing a label choice before a document can be saved) only once you've seen that the taxonomy maps sensibly onto what people are actually creating. Skipping straight to mandatory labeling with a taxonomy nobody's validated against real content produces a predictable pattern: label analytics a month in show 90% of documents tagged with whatever sits at the top of the picker — usually "General" — because that's the fastest click to get past the save prompt, and by then the label no longer tells you anything about what's actually in the file.

Sensitivity labels are one of the few controls in Microsoft 365 that follow the data rather than the location it happens to sit in right now, which is exactly why getting the document-versus-container distinction right at the design stage is worth the extra care — a label that means one thing on a file