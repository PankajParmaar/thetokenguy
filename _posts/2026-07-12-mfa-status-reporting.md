---
layout: post
title: "MFA Status Reporting"
date: 2026-07-12
categories: ["PowerShell & Graph", "Security Automation"]
tags: ["powershell", "mfa", "entra-id", "reporting"]
author: pankaj
---
"How many users don't have MFA registered" is one of those questions that sounds simple until you try to answer it precisely, because MFA status in Entra ID isn't a single flag — it's spread across authentication methods registration, conditional access enforcement, and legacy per-user MFA settings that some tenants still have lingering from before conditional access existed. Getting a clean report means understanding which of these you're actually measuring.

## The authentication methods approach

The modern way to check MFA registration status is through the authentication methods Graph endpoint, which reports what methods each user actually has registered:

```powershell
Connect-MgGraph -Scopes "Reports.Read.All","UserAuthenticationMethod.Read.All"

Get-MgReportAuthenticationMethodUserRegistrationDetail -All |
    Select-Object UserPrincipalName, IsMfaRegistered, MethodsRegistered |
    Export-Csv .\mfa-status.csv -NoTypeInformation
```

This is the report I trust most because it reflects actual registered methods rather than a policy assumption. It's also the one most people don't know exists — a lot of older scripts still floating around online query per-user legacy MFA settings through the deprecated MSOnline module, which reports on an authentication model many tenants have already moved away from and can give a misleadingly clean or misleadingly alarming picture depending on which legacy setting happens to still be set.

## Where the gaps hide

The report above tells you who has MFA registered. It does not tell you whether MFA is actually enforced for that user in every sign-in scenario, which is a separate question answered by conditional access coverage. I cross-reference the registration report against the sign-in logs, filtering for sign-ins where the authentication requirement satisfied was single-factor, to find accounts that are technically registered for MFA but aren't being challenged for it in practice. I found this exact gap once with a "Break-Glass-Exclusion" group created for two emergency-access accounts that had quietly picked up eleven regular user accounts over the following year, all sailing through single-factor.

```powershell
Get-MgAuditLogSignIn -Filter "userPrincipalName eq '$upn'" -Top 20 |
    Select-Object CreatedDateTime, AppDisplayName, @{N='AuthRequirement';E={$_.AuthenticationRequirement}}
```

Break-glass and service accounts are the other recurring gap. They're routinely excluded from MFA by design, which is defensible, but an MFA report that doesn't explicitly flag which accounts are excluded and why tends to get read by an auditor as an unexplained finding rather than a documented exception. I've started appending an explicit exclusion column to every report I generate, mapping each unregistered account back to the conditional access exclusion or emergency-access designation that explains it.

## What actually gets used

A one-time MFA report is a snapshot that's stale within a month as new hires and role changes shift the picture. The version that actually gets used is a small scheduled export feeding a simple dashboard or even a weekly email — not because the data changes dramatically week to week, but because the report only has