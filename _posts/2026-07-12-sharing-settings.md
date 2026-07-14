---
layout: post
title: "Sharing Settings"
date: 2026-07-12
categories: ["Microsoft 365", "SharePoint & OneDrive"]
tags: ["sharepoint", "onedrive", "externalsharing", "governance"]
author: pankaj
description: "SharePoint and OneDrive sharing is layered — tenant, site, link level, each capped by the one above. Assuming a site setting protects you is the mistake."
image:
  path: /assets/img/og-default.png
  alt: "Sharing Settings"
---
External sharing in SharePoint and OneDrive operates on a layered model that's easy to misjudge if you only ever look at one layer of it. There's a tenant-level setting, a site-level setting that can only be as permissive as the tenant setting (never more), and a link-level choice made by whoever's actually sharing a file. Getting the layering backwards — assuming a restrictive site setting protects you when the tenant level is wide open, or vice versa — is where most sharing surprises come from.

## The tenant-level ceiling

The SharePoint admin center's sharing page sets the outer boundary for the entire tenant, with four broad options ranging from "Anyone" (anonymous links work, no sign-in required) down to "Only people in your organization" (no external sharing at all, full stop). Whatever you pick here is the absolute ceiling — a site can be configured more restrictively than the tenant default, but it can never be configured more permissively. If the tenant is set to "New and existing guests" (sign-in required, but external accounts are allowed), no individual site can enable anonymous "Anyone" links no matter what that site's own sharing settings say, and that's worth confirming directly if a site owner insists they've configured anonymous links and they're not working.

## Site-level scoping

Below the tenant ceiling, individual sites can dial sharing down further, which is the right move for anything holding sensitive data. A site holding HR or legal documents should be set to "Only people in your organization" even if the tenant default allows external sharing broadly for less sensitive collaboration sites. This is also where the sharing settings interact with sensitivity labels — a container-level label applied to a site can enforce a sharing restriction that overrides what an individual site owner might otherwise configure, which is a stronger control than relying on every site owner making the right manual choice.

## Link types: the setting most people get wrong

When someone shares a file, they're choosing a link type, and this is the layer with the most day-to-day impact on actual exposure. "Anyone with the link" creates an anonymous link that works for literally anyone who has the URL, with no sign-in and no expiration unless one's configured — and it doesn't stop at the person you sent it to; if they forward the link, whoever receives it has the same access. "Specific people" scopes the link to named individuals or groups, checked at open time regardless of who physically has the URL. "People in your organization" and "People with existing access" round out the options, with the former limiting to internal accounts and the latter creating a link that grants no new access at all, useful for reminding someone who should already have access where to find something.

The default link type presented to users in the sharing dialog is itself a tenant setting worth checking. Leaving the default at "Anyone" trains users to reach for the most permissive option out of habit, since it's the first thing they see, even when "Specific people" would have been the appropriate choice for a sensitive file. Changing the tenant default to "Specific people," while still leaving "Anyone" available as an explicit second click for cases that genuinely need it, shifts the path of least resistance toward the safer choice without removing functionality anyone actually needs.

## Expiration and access review, not just the initial grant

Anonymous links can and should have expiration dates set — either per-link or enforced as a tenant policy — because an anonymous link with no expiration is effectively a permanent, unrevoked door into a file that outlives the reason it was created. Pairing expiration policy with periodic sharing link reports (available through the SharePoint admin center or Purview) is what turns sharing settings from a one-time configuration decision into an ongoing, checkable control rather than a policy that only exists on paper.
