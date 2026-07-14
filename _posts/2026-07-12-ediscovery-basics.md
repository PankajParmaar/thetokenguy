---
layout: post
title: "eDiscovery Basics"
date: 2026-07-12
categories: ["Microsoft 365", "Compliance & Data Protection"]
tags: ["ediscovery", "purview", "compliance", "legalhold"]
author: pankaj
description: "Nobody learns eDiscovery until legal needs it urgently — the worst time to learn a case from a search from a hold. Get familiar with it before that day."
image:
  path: /assets/img/og-default.png
  alt: "eDiscovery Basics"
---
eDiscovery is one of those Microsoft 365 capabilities most admins never touch until the day someone from legal or HR needs it urgently, which is exactly the wrong time to be learning the difference between a case, a search, and a hold for the first time. Getting familiar with the mechanics ahead of an actual need is worth the time, because the tooling has enough moving parts that improvising under pressure is a good way to miss something a court or regulator later asks about.

## The three tiers, and which one you actually need

Purview offers eDiscovery in three tiers of increasing capability. Content search is the lightest-weight option — it lets you query across mailboxes, SharePoint sites, and OneDrive accounts using keyword and condition-based search (sender, date range, specific file types), and export the results. It's useful for quick internal lookups but has no case management, no hold capability tied to custodians, and no review workflow, so it's genuinely the wrong tool for anything that might end up in litigation. eDiscovery (Standard) adds case management and the ability to place holds on specific mailboxes and sites tied to a named case, which is the right tier for most internal investigations. eDiscovery (Premium) adds custodian management, legal hold notifications sent directly to custodians, review sets with deduplication and threading, and analytics like near-duplicate detection, which is what you actually need once a matter is heading toward external counsel or regulatory production.

## Holds preserve independently of retention labels

This is worth restating clearly because it's a common point of confusion: a hold placed through eDiscovery preserves content for the custodians and locations named in that hold, regardless of whatever retention label or policy would otherwise apply, and regardless of whether the user tries to delete the content. A custodian's mailbox under hold behaves normally from their perspective — they can still delete an email from their view — but the underlying item is preserved in a hidden location and remains fully discoverable. The hold doesn't stop working because a retention label's deletion schedule says the item should be gone by now; the hold explicitly overrides that.

## Search scope and the false confidence of "search everything"

A content search or eDiscovery search only covers locations explicitly included in its scope, and by default a newly created search doesn't automatically include every possible location type — Teams chat data, Yammer, and third-party data connectors sometimes need to be explicitly added depending on how the search or case was configured. Running a search and getting zero or few results can mean genuinely nothing relevant exists, or it can mean the scope silently excluded the location where the relevant content actually lives. Confirming scope explicitly, rather than trusting a search's default inclusion list, is the difference between a defensible negative result and one that falls apart under scrutiny later.

## Export and chain of custody

Exporting search results produces a package with the actual content plus a report detailing what was searched, what matched, and export metadata — this report is part of what makes the process defensible if it's ever challenged, since it documents the query logic and scope rather than just handing over a folder of files with no record of how they were selected. Treating the export report as disposable paperwork rather than part of the actual deliverable is a mistake that only becomes obvious when someone later asks how a specific document was identified and included.

The practical takeaway for anyone studying this rather than running live legal matters: build a habit of running a content search occasionally against a known, low-stakes question just to get comfortable with query syntax, hold placement, and export mechanics, because the day you need eDiscovery for something real is not the day you want to be learning the interface from scratch.
