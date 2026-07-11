---
layout: post
title: "Governance for Teams"
date: 2026-07-12
categories: ["Microsoft 365", "Teams & Collaboration"]
tags: ["teams", "governance", "lifecycle", "microsoft365groups"]
author: pankaj
---
Every Team you create is backed by a Microsoft 365 Group, which in turn provisions a SharePoint site, a shared mailbox, a Planner if used, and now a group in Entra ID with its own membership and lifecycle. That's the part people miss when they think of "creating a Team" as a lightweight action — it isn't lightweight underneath, and a tenant that lets anyone create a Team without any governance wrapped around it ends up, within a year or two, with hundreds of Teams and no clear picture of which ones are still active, who owns them, or what's actually in the SharePoint site behind each one.

## Who can create a Team, and why that decision matters early

By default, any licensed user can create a new Team, which is convenient but means Team sprawl starts on day one rather than after some threshold is crossed. Restricting Team creation to a specific security group, and routing creation through a request process (even something as simple as a form that provisions the Team with a naming convention and required owner), is the single highest-leverage governance decision available, and it's dramatically easier to put in place before Teams have already multiplied than after. Retrofitting a naming convention onto three hundred existing Teams named "Team," "Team (1)," and "Project Thing" is not a fun cleanup project.

## Ownership is the thing that actually decays

The recurring failure mode in Teams governance isn't creation, it's ownership. A Team gets created with one owner, that person leaves the organization or changes roles, and the Team is left with zero owners — at which point nobody can add or remove members, change settings, or archive it, because Team ownership isn't something admins can casually reassign without going through the Microsoft 365 Groups management surface in the admin center or PowerShell. Requiring at least two owners at creation time, and running a periodic check for Teams that have dropped to zero owners, catches this before it becomes an access-review nightmare where a Team full of sensitive project files has literally nobody responsible for it.

## Expiration policies and archiving

Microsoft 365 Groups support an expiration policy that forces owners to periodically confirm a Team is still needed, after which it's either renewed or deleted (with a 30-day soft-delete recovery window). Turning this on tenant-wide, with a sensible interval like 180 or 365 days, is a low-effort way to force the "is this still active" question to actually get asked instead of Teams just accumulating indefinitely. Archiving is the softer alternative for Teams that finished their purpose but whose content still needs to be retrievable — an archived Team goes read-only for chat and files but stays searchable, which is usually the right call for a completed project rather than outright deletion.

## Sensitivity labels at the container level

Beyond the individual file-level sensitivity labels covered elsewhere, Microsoft 365 Groups and Teams themselves can carry a container-level sensitivity label, which controls things like whether guests can be added, whether the Team is public or private by default, and whether external sharing is even possible for that Team's SharePoint site. Applying a "Confidential" container label to a Team dealing with sensitive data enforces those restrictions at creation time rather than relying on every member remembering to configure sharing settings correctly afterward, which holds up a lot better than a governance wiki page that hasn't been opened since the project it documented wrapped up.

The pattern underneath all of this is the same one that shows up across Microsoft 365: the platform makes it trivially easy to create collaborative surfaces and offers almost no default friction to stop sprawl, so governance has to be a deliberate layer someone builds 