---
layout: post
title: "Permission Inheritance"
date: 2026-07-12
categories: ["Microsoft 365", "SharePoint & OneDrive"]
tags: ["sharepoint", "permissions", "inheritance", "governance"]
author: pankaj
---
Permission inheritance in SharePoint is one of those concepts that's simple to explain and genuinely difficult to keep straight once a site has been alive for a couple of years with people breaking inheritance here and there to solve one-off problems. Understanding the model properly is what separates "I can predict who can see this folder" from "I have to actually go check every time," which matters a lot when you're trying to reason about exposure during a review.

## The default: everything inherits from the site

A SharePoint site has a top-level permission set — usually mapped to Owners, Members, and Visitors groups, each with a corresponding permission level (Full Control, Edit, Read). By default, every document library, folder, and file underneath that site inherits those same permissions. Nothing at a lower level has its own independent permission list; it just points up to whatever the site says. This is why adding someone to a site's Members group grants them access to everything in that site instantly, and why it's the cleanest way to manage access when the model isn't broken by exceptions.

## Breaking inheritance is where things get messy

The moment someone stops inheritance on a folder — usually to give one external partner access to just that folder without giving them the whole site — that folder gets its own independent permission list, completely disconnected from whatever happens at the site level afterward. If the site owner later removes someone from the site's Members group because they've left the project, that person still has access to the folder with broken inheritance, because that folder isn't listening to the site's group membership anymore. This is the single most common cause of "why does this person still have access" surprises during an access review, and it's rarely malicious — it's almost always someone solving an immediate sharing need years ago and nobody circling back.

Broken inheritance can happen at the library level, the folder level, or even the individual file level, and each one it happens at is one more place that now needs to be checked independently during any review. A site with inheritance broken at forty different folders effectively has forty separate access lists to audit instead of one, and SharePoint's UI doesn't make that fan-out obvious unless you specifically go looking with something like the site permissions report or PowerShell (`Get-PnPListItem` combined with checking `HasUniqueRoleAssignments` at each level).

## Practical guidance

The healthiest pattern is resisting broken inheritance wherever possible and instead solving access problems through the site's group structure — creating an additional SharePoint group scoped to a specific need, or using a Microsoft 365 Group-linked Team's channel structure to create a naturally separate site instead of carving exceptions into an existing one. When breaking inheritance is genuinely necessary (a folder with regulatory-restricted content that only three people in a fifty-person site should see), documenting why it was broken, and by whom, saves whoever reviews it later from having to reverse-engineer the intent.

Running a periodic report of unique permissions across a tenant's SharePoint sites — Microsoft's own compliance and audit tooling, or a third-party governance tool, can do this — is worth treating as a recurring task rather than a one-time cleanup, because broken inheritance doesn't announce itself; it just quietly accumulates until an access review turns it into a several-day archaeology project instead of a five-minute confirmation that the site-level groups are correct.
