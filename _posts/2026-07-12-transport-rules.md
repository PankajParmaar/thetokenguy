---
layout: post
title: "Transport Rules"
date: 2026-07-12
categories: ["Microsoft 365", "Exchange Online"]
tags: ["exchangeonline", "transportrules", "mailflow", "dlp"]
author: pankaj
---
Transport rules, officially called mail flow rules in the Exchange admin center, are the "if this, then that" layer sitting inside the mail flow pipeline. They're conditional logic applied to every message that passes through Exchange Online's transport service, and they're powerful enough to quietly redirect, modify, or block mail in ways that are hard to spot from the mailbox side unless you know to go looking at the rule list itself.

## What a rule is actually made of

Every transport rule has three parts: a condition (sender is external, subject contains a keyword, attachment matches a file type), an action (add a disclaimer, redirect the message, apply a header, block delivery with an NDR), and an optional exception that carves out cases where the rule shouldn't apply. Rules run in priority order, lowest number first, and by default execution stops for a given message once a rule with a "stop processing more rules" action fires — which is the single most common source of "why didn't my later rule apply" confusion. If rule 3 redirects a message and has that stop flag set, rule 7 further down the list never sees that message at all, regardless of how well-written rule 7 is.

## Common patterns worth knowing

External sender warnings are probably the most widely deployed transport rule in any tenant: a condition checking `sender is located outside the organization`, paired with a prepend action that adds a banner like "This message is from an external sender." It's a blunt tool, but it's cheap and it does meaningfully reduce successful phishing clicks in practice. Disclaimer rules work the same way for outbound mail, appending legal footers based on the sending domain or department.

Attachment-based rules are where things get more consequential. A rule blocking executable attachments, or requiring encryption for messages containing certain sensitive information types, is doing real data-loss-prevention work even before a dedicated DLP policy gets involved — and this is a point of confusion I've run into more than once, because transport rules and DLP policies overlap in capability but are configured in different places (the Exchange admin center versus the Purview compliance portal) and evaluated at different points, so a tenant can end up with two separate mechanisms trying to do the same job with slightly different logic.

## Testing before you enforce

The single best habit for transport rules is using "Test mode" (or the equivalent "test without policy tips" option) before setting a rule live, especially for anything with a block or redirect action. Test mode logs what the rule would have done without actually doing it, which lets you check message trace against real traffic for a few days and confirm the condition logic matches reality rather than what you assumed it would match. I've seen a rule intended to catch messages from one specific external partner domain instead match a substring that also appeared in a completely unrelated internal project name, silently redirecting legitimate mail for weeks before anyone noticed traffic looked thin.

## Rule order discipline

As the rule list grows, keeping a naming convention that encodes priority intent (something as simple as prefixing rule names with a number) saves a lot of squinting at the admin center's reorderable list later. Transport rules are one of those features that starts simple and, without discipline, turns into an unreadable stack of overlapping conditions within a year — worth revisiting the full list periodically rather than only ever adding to the bottom.
