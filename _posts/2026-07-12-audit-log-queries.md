---
layout: post
title: "Audit Log Queries"
date: 2026-07-12
categories: ["PowerShell & Graph", "Reporting & Auditing"]
tags: ["powershell", "audit-logs", "entra-id", "compliance"]
author: pankaj
---
The directory audit log is where every administrative action in Entra ID gets recorded — role assignments, policy changes, group creation, password resets triggered by an admin rather than the user themselves. It's also one of the least glamorous parts of the tenant, right up until an incident review or a compliance audit needs to answer "who changed this and when," and the answer needs to come from actual log data rather than someone's recollection of a Slack thread from three months ago.

## Pulling targeted audit events

The audit log endpoint supports filtering by activity type and date range, which matters because pulling the entire log unfiltered in an active tenant returns an unmanageable volume fast:

```powershell
Connect-MgGraph -Scopes "AuditLog.Read.All"

$since = (Get-Date).AddDays(-30).ToString("yyyy-MM-ddTHH:mm:ssZ")
Get-MgAuditLogDirectoryAudit -Filter "activityDateTime ge $since and activityDisplayName eq 'Add member to role'" -All |
    Select-Object ActivityDateTime, InitiatedBy, TargetResources
```

The InitiatedBy and TargetResources properties are nested objects rather than flat strings, which is the detail that trips people up the first time they try to export this straight to CSV and end up with unreadable object references instead of names. I flatten them explicitly before export:

```powershell
Get-MgAuditLogDirectoryAudit -Filter "activityDateTime ge $since" -All |
    Select-Object ActivityDateTime, ActivityDisplayName,
        @{N='InitiatedBy';E={$_.InitiatedBy.User.UserPrincipalName}},
        @{N='Target';E={$_.TargetResources[0].DisplayName}}
```

## Role assignment changes as the priority query

If there's one audit query worth having on a schedule, it's privileged role assignment changes — additions and removals to roles like Global Administrator, Privileged Role Administrator, or any custom role with broad permissions. Standing privileged access is one of the more common findings in a security review, and catching an unexpected role addition within a day of it happening is a very different conversation than discovering it during an annual audit.

```powershell
Get-MgAuditLogDirectoryAudit -Filter "activityDisplayName eq 'Add member to role'" -All |
    Where-Object { $_.TargetResources.DisplayName -match "Global Administrator|Privileged Role Administrator" }
```

## The retention gap that catches people out

Microsoft Graph's audit log API has a default retention window — 30 days for most tenants without an E5 license or a separate Microsoft Purview retention policy — and that window is shorter than most compliance frameworks actually require. A script that runs monthly is exactly on the edge of missing data if it's delayed even a few days, which is why I run audit exports weekly rather than monthly and archive them externally, treating the Graph API's native retention as a floor rather than the actual archive.

Audit queries are the least exciting part of the identity toolkit and also the part that ends up mattering most in exactly the moments you can't plan for on short notice — an unexpected privilege escalation, a dispute over who approved a change, an auditor asking for evidence that's supposed to already exist and the native 30-day retention window has already rolled past it. Having the e