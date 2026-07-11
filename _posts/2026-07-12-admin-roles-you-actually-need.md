---
layout: post
title: "Admin Roles You Actually Need"
date: 2026-07-12
categories: ["Microsoft 365", "Tenant Administration"]
tags: ["microsoft365", "entraid", "rbac", "admin-roles"]
author: pankaj
---
Entra ID ships with well over a hundred built-in roles, and the temptation in a small tenant is to just hand out Global Administrator, since it's the role name that shows up first in every setup guide and it never fails to grant the access someone's asking for. It works because it grants access to everything, which is exactly the problem. Every Global Admin account is a full compromise of the tenant if that account's credentials or session gets stolen, and most people doing day-to-day admin work don't need anywhere near that blast radius.

## The roles that cover most real work

For a typical small-to-mid tenant, four or five scoped roles cover almost everything a working admin team actually does. User Administrator handles password resets, license assignment, and user lifecycle work without touching security policy or Exchange configuration. Exchange Administrator scopes down to mail flow, mailbox permissions, and transport rules, which is the right fit for whoever owns messaging without needing to touch Entra Conditional Access. SharePoint Administrator and Teams Administrator do the same job for their respective workloads. Security Administrator is worth calling out separately because it grants read/write to Conditional Access, Identity Protection, and Defender configuration without granting the ability to reset arbitrary user passwords or manage licensing, which makes it a better fit for a security engineer than Global Admin ever is.

Authentication Administrator deserves specific mention because it's easy to misjudge its scope. It can reset MFA and require re-registration for non-admin users, and for a decent set of admin roles below it in the hierarchy, but it cannot touch Global Administrators or other Authentication Administrators, which is intentional and worth knowing before you assume it's a full password-reset role.

## Where Global Admin actually belongs

Global Administrator should be reserved for break-glass accounts and maybe one or two people who genuinely need tenant-wide configuration authority — things like configuring organization-wide security defaults, managing other admins' roles, or handling cross-workload settings that don't map cleanly to a single scoped role. If you find yourself granting Global Admin to make an integration or app registration work faster, that's usually a sign you need Application Administrator or Cloud Application Administrator instead, both of which manage app registrations and enterprise apps without the rest of the tenant coming along for the ride.

## Privileged Identity Management, even in a small setup

If you're running Entra ID P2, turning on PIM for admin roles is worth doing even in a lab-sized tenant, because it changes the default posture from "this account has standing access" to "this account can activate access for a defined window, with justification logged." Watching how PIM activation requests show up in the audit log, and how eligible-versus-active role assignment actually behaves, is a useful thing to understand before you're relying on it to protect a real environment. Even without PIM, regularly running the Entra ID access reviews feature against your admin role assignments catches the accounts that picked up a role for a one-off task and never had it removed.

The underlying discipline is the same regardless of tenant size: match the role to the job, keep Global Admin rare and well-guarded, and revisit assignments on a schedule instead of only during 