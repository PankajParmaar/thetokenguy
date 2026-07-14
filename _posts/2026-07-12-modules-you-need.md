---
layout: post
title: "Modules You Need"
date: 2026-07-12
categories: ["PowerShell & Graph", "Scripting Foundations"]
tags: ["powershell", "modules", "microsoft-graph", "entra-id"]
author: pankaj
description: "A whole afternoon lost to module conflicts before reaching the actual script logic. Which PowerShell module for IAM still works, versus outdated advice."
image:
  path: /assets/img/og-default.png
  alt: "Modules You Need"
---
I've watched people spend a whole afternoon trying to run an identity script only to get stuck on module conflicts before they even reach the actual logic. The Microsoft PowerShell ecosystem for IAM has been through several generations of modules, and half the confusion out there online is old blog posts referencing modules that are deprecated or being sunset. Sorting out which module actually does what saves real time.

## The core set

For anything Entra ID and Microsoft 365 identity related, the module you want now is Microsoft.Graph, specifically its submodules rather than the entire monolith:

```powershell
Install-Module Microsoft.Graph.Authentication -Scope CurrentUser
Install-Module Microsoft.Graph.Users -Scope CurrentUser
Install-Module Microsoft.Graph.Groups -Scope CurrentUser
Install-Module Microsoft.Graph.Identity.SignIns -Scope CurrentUser
```

Installing the full Microsoft.Graph module pulls in hundreds of sub-modules and takes forever, and you almost never need all of it. I install just the pieces I'm working with — Users for account management, Groups for group operations, Identity.SignIns for conditional access and sign-in log work, Reports for usage and audit reporting. It's a smaller footprint and it makes version conflicts far less painful to debug.

The older AzureAD and MSOnline modules are deprecated — Microsoft has been retiring them in stages, and running them against a tenant is increasingly a gamble on how much longer specific cmdlets keep working. If you're maintaining scripts that still use `Get-AzureADUser` or similar, that's technical debt worth migrating off, not a stable foundation to build new work on.

## Exchange and beyond

If your IAM work touches mailbox permissions, distribution groups, or mail-enabled security groups, ExchangeOnlineManagement is still its own separate module — Graph's mail coverage doesn't fully replace it yet for administrative tasks like mailbox delegation:

```powershell
Install-Module ExchangeOnlineManagement -Scope CurrentUser
Connect-ExchangeOnline -UserPrincipalName admin@contoso.com
```

For Intune-adjacent identity work — device compliance tied to conditional access — Microsoft.Graph.DeviceManagement covers it, though I find myself reaching for the Graph Explorer first to confirm the exact query shape before writing it into a script.

## The version trap

One thing that bites people constantly: having multiple versions of Microsoft.Graph installed side by side, then wondering why a script behaves differently on a colleague's machine. `Get-InstalledModule Microsoft.Graph* | Select Name, Version` is worth running before debugging anything else when a script that "worked yesterday" suddenly throws cmdlet-not-found errors. I keep a habit of pinning versions in shared scripts with `#Requires -Modules @{ModuleName='Microsoft.Graph.Users';ModuleVersion='2.x'}` at the top, which has saved me from more than one "works on my machine" argument.

Module sprawl is a quiet tax on identity automation — not dramatic, just a slow drain on debugging time until you settle on a deliberate, minimal set and stick with it across your scripts.
