---
layout: post
title: "Sign-in Log Analysis"
date: 2026-07-12
categories: ["PowerShell & Graph", "Security Automation"]
tags: ["powershell", "sign-in-logs", "entra-id", "threat-detection"]
author: pankaj
---
Sign-in logs are the single richest source of behavioral data in a Microsoft tenant, and also the easiest to drown in. A mid-size tenant generates tens of thousands of sign-in events a day, and scrolling through them in the portal to spot an anomaly is roughly as effective as looking for a specific grain of sand on a beach. The value only shows up once you're pulling them through Graph and filtering for the specific patterns that actually matter.

## Getting the data out

The basic pull, scoped to a reasonable window so you're not dragging the entire log history every time:

```powershell
Connect-MgGraph -Scopes "AuditLog.Read.All"

$since = (Get-Date).AddDays(-7).ToString("yyyy-MM-ddTHH:mm:ssZ")
Get-MgAuditLogSignIn -Filter "createdDateTime ge $since" -All |
    Select-Object CreatedDateTime, UserPrincipalName, AppDisplayName, IPAddress, @{N='Result';E={$_.Status.ErrorCode}}
```

An ErrorCode of 0 means success; anything else is a failure code worth looking up, and the meaningful ones for security purposes are usually clustered around 50126 (invalid credentials), 50053 (account locked from repeated failures), and 53003 (blocked by conditional access). Filtering for those specific codes turns a wall of log entries into a short, actionable list.

## Patterns worth building queries around

Impossible travel is the classic one — same user, two sign-ins from geographically distant locations within a window too short for actual travel. Entra ID's own risk detection covers this if you have Identity Protection licensed, but for tenants without it, a rough version is doable by grouping sign-ins per user and flagging country changes within short time gaps:

```powershell
$signins | Group-Object UserPrincipalName | ForEach-Object {
    $sorted = $_.Group | Sort-Object CreatedDateTime
    for ($i = 1; $i -lt $sorted.Count; $i++) {
        if ($sorted[$i].Location.CountryOrRegion -ne $sorted[$i-1].Location.CountryOrRegion) {
            $diff = (New-TimeSpan $sorted[$i-1].CreatedDateTime $sorted[$i].CreatedDateTime).TotalHours
            if ($diff -lt 3) { $sorted[$i] }
        }
    }
}
```

This is a rough heuristic, not a substitute for Identity Protection's actual risk scoring — it'll produce false positives for legitimate VPN usage — but it's a reasonable stopgap for smaller tenants without the E5-tier licensing that unlocks the built-in detection.

## Legacy authentication as the recurring finding

The other query that consistently earns its keep is filtering sign-ins by client app used, specifically hunting for legacy authentication protocols like POP, IMAP, or older Exchange ActiveSync clients that bypass modern conditional access controls entirely. Those protocols don't support MFA challenges the same way modern auth does, and they remain one of the more common gaps between a tenant's stated security posture and what's actually reachable.

```powershell
Get-MgAuditLogSignIn -Filter "createdDateTime ge $since" -All |
    Where-Object { $_.ClientAppUsed -match "IMAP|POP|Exchange ActiveSync" }
```

Any non-zero result from that query is worth investigating immediately, since legacy auth traffic in a modern tenant is either a forgotten service account or an active bypass attempt. I ran this query against a client tenant once expecting zero rows and got back a ser