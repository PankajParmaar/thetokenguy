---
layout: post
title: "Why PowerShell for IAM"
date: 2026-07-12
categories: ["PowerShell & Graph", "Scripting Foundations"]
tags: ["powershell", "iam", "automation", "graph-api"]
author: pankaj
description: "Every IAM admin hits the wall where the portal stops being useful — checking MFA for four thousand users isn't a task, it's a punishment. Enter PowerShell."
image:
  path: /assets/img/og-default.png
  alt: "Why PowerShell for IAM"
---
Every IAM admin eventually hits the wall where the portal stops being useful. You need to check MFA status for four thousand users, or find every guest account that hasn't signed in for ninety days, or bulk-update licenses after a merger. Clicking through Entra admin center for that is not a plan, it's a punishment. This is the point where PowerShell stops being optional and becomes the actual job.

## The portal-to-script gap

Microsoft's admin portals are built for one-off changes, not bulk operations or repeatable audits. The moment a task touches more than a handful of objects, you're looking at either a script or an afternoon of clicking. I've lost count of the times I built a "quick manual check" that turned into a fifty-user spreadsheet exercise, and the second time it came up I finally wrote the ten lines of PowerShell I should have written the first time. That's the pattern worth internalizing early: if you're doing it twice, script it.

PowerShell also happens to be the native tongue of the Microsoft identity stack. Azure AD (now Entra ID), Exchange Online, SharePoint, Intune — all of it exposes management surfaces through PowerShell modules and, increasingly, through Microsoft Graph, which PowerShell talks to natively via the Microsoft.Graph module. That's not true of most other scripting languages without extra wrapper work.

## What it actually buys you

The obvious win is bulk operations — disabling fifty accounts after an offboarding batch, or reassigning licenses across a department. But the less obvious win is auditability. A script is a record of exactly what you did and why, versus a portal action that leaves you guessing later what filter you applied. When someone asks "how did you pull that stale account list six months ago," a saved .ps1 file with comments answers it in seconds. A memory of clicking through blades does not.

```powershell
Connect-MgGraph -Scopes "User.Read.All","AuditLog.Read.All"
Get-MgUser -All -Property DisplayName,SignInActivity |
    Select-Object DisplayName, @{N='LastSignIn';E={$_.SignInActivity.LastSignInDateTime}}
```

That's four lines and it already beats an afternoon in the portal. It's also reusable — save it, parameterize it, and it becomes a report you can run monthly without rethinking the logic each time.

## Where it breaks

The catch is that PowerShell for IAM isn't just "learn PowerShell." It's learning the object model of Entra ID, the quirks of Graph's paging and throttling, and the permission scopes that gate what your script can actually see. A script that works against a tenant with delegated permissions might fail silently against one where the app registration only has application permissions for a narrower scope. I've had scripts return empty arrays for twenty minutes before realizing the issue was a missing consent grant, not a bug in my filter logic.

None of this makes PowerShell a silver bullet. It's a floor, not a ceiling — the baseline competency that turns identity administration from a series of manual interventions into something closer to engineering. Once that clicks, the rest of the toolchain — modules, error handling, Graph queries — starts making a lot more sense as a coherent system rather than a pile of separate tricks.
