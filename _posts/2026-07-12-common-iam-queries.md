---
layout: post
title: "Common IAM Queries"
date: 2026-07-12
categories: ["PowerShell & Graph", "Graph API"]
tags: ["microsoft-graph", "powershell", "queries", "entra-id"]
author: pankaj
description: "The same handful of Graph questions come up constantly across identity work. A personal reference worth keeping instead of re-deriving them every time."
image:
  path: /assets/img/og-default.png
  alt: "Common IAM Queries"
---
There's a small set of Graph queries that show up constantly across identity work, regardless of what the specific task of the day happens to be — someone asks a variation of the same handful of questions often enough that it's worth keeping a personal reference of the exact syntax rather than re-deriving it from documentation every time. This is the shortlist I actually reach for.

## Users without MFA registered

```powershell
Get-MgReportAuthenticationMethodUserRegistrationDetail -All |
    Where-Object { -not $_.IsMfaRegistered } |
    Select-Object UserPrincipalName, UserDisplayName
```

## Guest accounts older than a threshold with no recent activity

Guest accounts are a recurring audit finding because they're provisioned for a specific project and then forgotten once that project ends:

```powershell
Get-MgUser -All -Filter "userType eq 'Guest'" -Property DisplayName,SignInActivity,CreatedDateTime |
    Where-Object {
        $_.CreatedDateTime -lt (Get-Date).AddDays(-180) -and
        (-not $_.SignInActivity.LastSignInDateTime -or
         [datetime]$_.SignInActivity.LastSignInDateTime -lt (Get-Date).AddDays(-90))
    }
```

## Applications with high-privilege permission grants

Checking what an app registration or enterprise app can actually do, rather than trusting its name:

```powershell
Get-MgServicePrincipal -All | ForEach-Object {
    $roles = Get-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $_.Id
    if ($roles.AppRoleId -contains "$adminConsentRoleId") {
        [PSCustomObject]@{ App = $_.DisplayName; Id = $_.Id }
    }
}
```

This one matters more than it looks — a lot of tenant risk sits in third-party app registrations that were consented once during a proof-of-concept and never revisited, holding permissions far broader than the app currently needs.

## Users assigned a specific admin role

```powershell
$roleId = (Get-MgDirectoryRole -Filter "displayName eq 'Global Administrator'").Id
Get-MgDirectoryRoleMember -DirectoryRoleId $roleId |
    Select-Object @{N='Name';E={$_.AdditionalProperties.displayName}}
```

Any tenant where this list is longer than four or five people is worth a conversation, since Global Administrator sprawl is one of the more common gaps between a tenant's documented access model and what's actually assigned.

## Recently created applications or service principals

Useful as a change-detection query — new app registrations are worth reviewing individually rather than discovering months later during an unrelated investigation:

```powershell
Get-MgApplication -All -Property DisplayName,CreatedDateTime |
    Where-Object { $_.CreatedDateTime -gt (Get-Date).AddDays(-7) }
```

## Building your own reference

None of these queries are complicated on their own — the value is having them written down, tested, and ready rather than reconstructing filter syntax from memory every time a variation of the same question comes up. I keep mine in a single reference script organized by topic, each query commented with what permission scope it needs and what it's actually checking for, because six months from now I won't remember why a specific filter clause is there, and neither will whoever inherits the script after me.
