---
layout: post
title: "Stale Account Detection"
date: 2026-07-12
categories: ["PowerShell & Graph", "Reporting & Auditing"]
tags: ["powershell", "stale-accounts", "entra-id", "offboarding"]
author: pankaj
description: "Stale accounts are the quietest risk in an identity environment precisely because nothing about them looks urgent — until one turns out to really matter."
image:
  path: /assets/img/og-default.png
  alt: "Stale Account Detection"
---
Stale accounts are one of the quietest risks in an identity environment, precisely because nothing about them looks urgent. Nobody gets paged when an account sits unused for a year with an active license and standing group memberships — it just sits there as a slightly wider attack surface until someone notices during an audit, or worse, until it turns out to be the account an attacker used because it had valid credentials and nobody was watching it.

## Defining "stale" precisely

The first mistake in stale account scripts is not defining the threshold clearly before writing the query. Ninety days of no sign-in is a common baseline, but it needs qualifying — no interactive sign-in, no non-interactive sign-in either (service principal activity on behalf of the account counts and can mask a genuinely unused account), and a check against account type, since service accounts and break-glass accounts are supposed to look stale by design and shouldn't trigger the same alert as a departed employee's account.

```powershell
Connect-MgGraph -Scopes "User.Read.All"

$threshold = (Get-Date).AddDays(-90)
Get-MgUser -All -Property DisplayName,UserPrincipalName,SignInActivity,AccountEnabled |
    Where-Object {
        $_.AccountEnabled -eq $true -and
        ($null -eq $_.SignInActivity.LastSignInDateTime -or
         [datetime]$_.SignInActivity.LastSignInDateTime -lt $threshold)
    } |
    Select-Object DisplayName, UserPrincipalName, @{N='LastSignIn';E={$_.SignInActivity.LastSignInDateTime}}
```

Accounts with a null LastSignInDateTime are worth flagging separately from accounts with an old but present date — a null often means the account has never signed in at all, which is a different problem (an orphaned provisioning record) than an account that went dormant after active use.

## Cross-referencing against HR data

Sign-in activity alone tells you an account is unused, not whether the person is still employed — those are different questions that get conflated constantly. A contractor on approved leave looks identical to a departed employee in raw sign-in data. The reports that hold up under scrutiny cross-reference stale sign-in activity against an HR export or an authoritative employment status source, flagging only the overlap rather than treating every dormant account as automatically actionable.

```powershell
$staleAccounts = Import-Csv .\stale-signin-report.csv
$hrActive = Import-Csv .\hr-active-employees.csv

$staleAccounts | Where-Object {
    $_.UserPrincipalName -notin $hrActive.UPN
} | Export-Csv .\confirmed-stale.csv -NoTypeInformation
```

## Acting on the list, carefully

Detection is the easy half. The action side — disabling rather than deleting, giving a grace window before deprovisioning, notifying the manager before pulling access — is where stale account cleanup either becomes routine hygiene or becomes the incident where someone's account got disabled mid-parental-leave because a script didn't account for approved absence. I've learned to treat every automated disable action as something with a human review step in front of it, no matter how clean the underlying query looks, because the query only ever answers "is this account unused" and never answers "is this safe to act on."
