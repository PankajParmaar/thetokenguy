---
layout: post
title: "Automating License Assignment"
date: 2026-07-12
categories: ["PowerShell & Graph", "Identity Automation"]
tags: ["powershell", "licensing", "entra-id", "m365"]
author: pankaj
---
License assignment sounds like the least interesting corner of IAM until you're the one explaining to finance why the tenant has four hundred unused E5 licenses sitting on accounts that left the company eight months ago. Manual license management drifts quietly because license removal usually isn't a hard step in the offboarding checklist — the account gets disabled, the ticket gets closed, and the license just rides along on a dead account until someone finally cross-references the assignment list against active headcount during a renewal cycle.

## Direct assignment vs group-based licensing

Microsoft gives you two real paths: assigning licenses directly to user objects, or assigning them to a group and letting group-based licensing propagate to members. Group-based licensing is the one worth defaulting to for anything beyond a handful of exceptions, because it turns license management into group membership management, which is a problem you already have tooling for. Direct assignment through Graph still matters for exceptions and for auditing what's actually attached to an account regardless of which method put it there.

```powershell
Get-MgUserLicenseDetail -UserId $user.Id |
    Select-Object SkuPartNumber, SkuId
```

That's the query I run first whenever someone asks "does this user actually have an E5" — trusting the portal's summary view without checking the underlying detail has burned me before, especially with users who ended up with licenses through more than one group.

## Assigning at scale

For direct bulk assignment — say, a new department that needs E3 licenses immediately rather than waiting for group sync to catch up:

```powershell
$skuId = (Get-MgSubscribedSku | Where-Object SkuPartNumber -eq "ENTERPRISEPACK").SkuId

foreach ($user in $newDeptUsers) {
    Set-MgUserLicense -UserId $user.Id `
        -AddLicenses @{SkuId = $skuId} `
        -RemoveLicenses @()
}
```

The SkuPartNumber names are one of Microsoft's more enduring annoyances — "ENTERPRISEPACK" is E3, "SPE_E5" is a Microsoft 365 E5 variant, and the mapping isn't intuitive from the name alone. I keep a small reference table in every licensing script rather than trusting memory, since guessing wrong here means either under-licensing someone or silently burning a license you didn't mean to assign.

## Catching the drift

The audit side is where automation earns its keep. A scheduled script that cross-references AccountEnabled status against active license assignments catches the disabled-but-still-licensed accounts that manual processes miss:

```powershell
Get-MgUser -All -Property DisplayName,AccountEnabled,AssignedLicenses |
    Where-Object { -not $_.AccountEnabled -and $_.AssignedLicenses.Count -gt 0 } |
    Select-Object DisplayName, @{N='LicenseCount';E={$_.AssignedLicenses.Count}}
```

Running that monthly against a tenant is a small thing, but it's the difference between catching license waste as a minor cleanup task versus finding it as an unexplained fou