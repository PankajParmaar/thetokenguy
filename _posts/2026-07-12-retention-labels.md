---
layout: post
title: "Retention Labels"
date: 2026-07-12
categories: ["Microsoft 365", "Compliance & Data Protection"]
tags: ["retention", "purview", "compliance", "recordsmanagement"]
author: pankaj
---
Retention in Microsoft Purview comes in two forms that get used almost interchangeably in casual conversation but behave quite differently: retention policies, which apply broadly across a location (a whole mailbox, a whole SharePoint site), and retention labels, which apply at the individual item level and can travel with content as it moves. Understanding when you actually need a label instead of a broad policy is the difference between a retention setup that matches how records actually get created and one that just applies a blanket rule and hopes it's close enough.

## Retain versus delete, and why the combination matters

A retention label's core job is to answer two questions for whatever it's applied to: how long should this be kept, and what happens at the end of that period. The "keep" side prevents permanent deletion even if a user tries to delete the item, which is the mechanism that actually protects content from accidental or intentional loss during the retention window — a user hitting delete on a labeled email doesn't actually destroy it; it moves to a hidden recoverable items location governed by the label, invisible to the user but retrievable through eDiscovery. The "then what" side determines whether the item is automatically deleted once retention expires, kept indefinitely, or routed to a disposition review where someone has to manually approve final deletion, which is the right setting for anything with regulatory weight where you want a human decision point before content is destroyed for good.

## Labels can be applied manually, automatically, or as defaults

Manual application means a user picks the label themselves, which relies on them recognizing that a document is a record worth labeling — reasonable for a small, trained population, unreliable at scale. Auto-apply policies based on sensitive information types or keyword conditions handle the scale problem but need the same tuning discipline as DLP conditions, since an auto-apply rule that's too broad ends up retention-locking content that never needed it, and one that's too narrow misses the records it was actually built to catch. Default labels applied at the SharePoint library or Teams channel level are often the most practical middle ground — every document landing in a specific library (say, a Finance approvals library) gets the same retention treatment automatically, without needing content inspection or user action at all.

## Retention labels and legal hold are not the same lever

It's worth being precise about this distinction because it comes up during actual legal or HR matters, not just theoretical governance discussions. A retention label sets a general lifecycle policy for a category of content. A legal hold (or an eDiscovery hold placed on specific custodians) is a separate, targeted mechanism that preserves content regardless of any retention label's disposition setting, specifically because a hold needs to override normal deletion the moment litigation or an investigation is reasonably anticipated. Content under both a retention label's "delete after X years" setting and an active hold will not be deleted — the hold wins — but relying on retention labels alone to satisfy a genuine legal hold obligation is a mistake, because labels are a lifecycle policy, not a preservation order tied to a specific matter.

## The conflict resolution people don't expect

When multiple retention labels or policies apply to the same item with different retention periods, Purview resolves the conflict using a documented set of precedence rules — generally, the longer retention period wins over a shorter one, and an explicit "retain" wins over a "delete," but the exact resolution logic (label versus policy, event-based versus static duration) is detailed enough that it's worth reading Microsoft's actual precedence documentation rather than assuming intuitively which rule should win. I've seen a tenant assume a shorter, more specific label would override a broader policy, when in fact the retention duration itself, not the specificity of the rule, was what determined precedence, and the assumption cost a bit of confused troubleshooting before the actual precedence order clicked.

Retention only works as a control if the labels map to how records actually get created in practice, which means the deployment work is less about the Purview configuration screens themselves and more about understanding where the content that actually needs governing gets generated in the first place.
