---
layout: post
title: "Shared Mailboxes vs Distribution Lists"
date: 2026-07-12
categories: ["Microsoft 365", "Exchange Online"]
tags: ["exchangeonline", "sharedmailbox", "distributionlist", "groups"]
author: pankaj
description: "A shared mailbox created where a distribution list belonged usually happens because someone reached for whichever they'd used last. Different problems."
image:
  path: /assets/img/og-default.png
  alt: "Shared Mailboxes vs Distribution Lists"
---
This is one of those distinctions that seems obvious once you've worked with both, but I've seen plenty of setups where a shared mailbox got created for something that should have been a distribution list, or vice versa, purely because the person setting it up reached for whichever one they'd used most recently. They solve genuinely different problems, and the wrong choice tends to surface as an operational annoyance months later rather than an immediate failure.

## Shared mailboxes are a place, not a list

A shared mailbox is an actual mailbox — it has its own inbox, sent items, calendar, and storage, and members open it in Outlook as an additional mailbox alongside their own. It's the right tool when a team needs to collaboratively work a queue of incoming mail: a `support@contoso.com` inbox where anyone on the team can see what's been replied to, assign themselves a message, and check sent history to avoid duplicate replies. Shared mailboxes don't require a license as long as they stay under 50GB, which makes them cheap to create, but that also means people create far more of them than they actually need, and a tenant with two hundred shared mailboxes nobody remembers the purpose of is a real governance headache during an access review.

Access to a shared mailbox is granted through Full Access and Send As (or Send on Behalf) permissions, assigned per user or, better, through a mail-enabled security group so you're not touching individual mailbox permissions every time someone joins or leaves the team. Auto-mapping is on by default with Full Access, which is convenient until you have someone with Full Access on forty shared mailboxes and Outlook takes forever to load all of them.

## Distribution lists are a routing address, not a place

A distribution list has no mailbox of its own. It's purely an alias that expands to a set of recipients at the moment a message is sent — mail addressed to the DL fans out into each member's own inbox, and there's no shared inbox, no shared calendar, nothing collaborative about it. That makes DLs the right choice for one-way or broadcast communication: "everyone in Finance," "all managers," a list you email but never need to reply-all-and-triage-as-a-team.

Microsoft has been steering people toward Microsoft 365 Groups as the modern replacement for classic distribution lists, and Groups do carry more capability — a shared calendar, a Teams-linked SharePoint site, a Planner if you want it — but that added capability is exactly why a Group is the wrong choice if all you actually wanted was a mail alias. Spinning up a full Microsoft 365 Group, with its own SharePoint site and Teams provisioning, just to send a monthly newsletter to a list of names is unnecessary overhead and adds another SharePoint site to the pile someone eventually has to govern.

## The decision that actually matters

The real question to ask before creating either is: does this need collaborative history that lives in one shared place, or does it just need to reach a set of inboxes? Support queues, shared calendars for a function, and role-based mailboxes like `hr@` belong in shared mailboxes. Announcement lists, notification recipients, and anything that's purely fan-out belong in a distribution list or a mail-enabled security group. Getting this backwards doesn't break anything on day one, but it does mean someone eventually asks why the "support" distribution list has forty people getting the same email with no way to see who already replied — at which point you're migrating to a shared mailbox anyway, just with more history to carry over.
