---
layout: post
title: "Conditional Access Policy Backup"
date: 2026-07-12
categories: ["PowerShell & Graph", "Security Automation"]
tags: ["powershell", "conditional-access", "entra-id", "backup"]
author: pankaj
description: "No native version history exists for Conditional Access beyond a basic audit entry. What to build before a cleanup pass accidentally locks out the org."
image:
  path: /assets/img/og-default.png
  alt: "Conditional Access Policy Backup"
---
Conditional access policies are some of the highest-stakes configuration in an entire tenant, and there's no native version history in the portal beyond the basic audit log entry showing that a policy changed. If someone edits a policy and locks out a chunk of the org, or a well-meaning admin "simplifies" a policy during a cleanup pass and removes a condition that mattered, there's no built-in rollback button. That gap is exactly why backing up conditional access policies as code, on a schedule, is one of the more valuable small habits in a tenant.

## Pulling the current state

Microsoft Graph exposes conditional access policies through the identity.signIns module, and pulling the full set as JSON is a handful of lines:

```powershell
Connect-MgGraph -Scopes "Policy.Read.All"
$policies = Get-MgIdentityConditionalAccessPolicy -All
$policies | ConvertTo-Json -Depth 10 |
    Out-File ".\ca-backup-$(Get-Date -Format 'yyyyMMdd').json"
```

The `-Depth 10` matters more than it looks like it should — conditional access policy objects nest conditions, grant controls, and session controls several layers deep, and the default ConvertTo-Json depth of 2 truncates most of the useful detail into flattened, unreadable object references. I learned that the hard way comparing two backups that looked identical because the interesting parts had been silently cut off.

## Making the backup actually useful

A raw JSON dump is a snapshot, but the real value comes from diffing it against the previous day's backup so you know what changed without reading the whole file:

```powershell
$today = Get-Content ".\ca-backup-$(Get-Date -Format 'yyyyMMdd').json" | ConvertFrom-Json
$yesterday = Get-Content ".\ca-backup-$(Get-Date (Get-Date).AddDays(-1) -Format 'yyyyMMdd').json" | ConvertFrom-Json

Compare-Object $today.displayName $yesterday.displayName
```

That's a crude diff on names alone, but even that surfaces added or removed policies immediately. For deeper comparison I've found it more reliable to diff specific fields — state, conditions.users, grantControls — rather than trying to compare entire nested objects, since ConvertFrom-Json produces PSCustomObjects that don't compare cleanly with each other even when their content is identical.

## Restoring isn't automatic

Backing up is the easy half. Restoring from a JSON backup means recreating the policy through `New-MgIdentityConditionalAccessPolicy`, and Graph won't accept the exported object as-is — read-only fields like id, createdDateTime, and modifiedDateTime need to be stripped before resubmission, or the call fails outright. I keep a small cleanup function specifically for this, since the failure mode of trying to restore with those fields intact is an unhelpful Graph error rather than a clear "remove these properties" message.

Treating conditional access as configuration that deserves version history — rather than something that only exists live in the portal — turns a policy mistake from a scramble to remember what it used to say into a two-minute diff against yesterday's backup.
