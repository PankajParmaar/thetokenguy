---
layout: post
title: "Error Handling Patterns"
date: 2026-07-12
categories: ["PowerShell & Graph", "Scripting Foundations"]
tags: ["powershell", "error-handling", "scripting", "reliability"]
author: pankaj
---
A script that runs cleanly in testing and then silently skips half its targets in production is worse than one that crashes outright. I had exactly this happen with an offboarding script — it "completed successfully" against a batch of forty terminated accounts, but a permissions timeout had quietly skipped six of them, and the gap only surfaced three weeks later when one of those six accounts showed up in a sign-in log during an access review. Error handling in identity scripts isn't about making things look tidy — it's about knowing, with certainty, what happened to every single object you touched.

## Try/Catch is the floor, not the ceiling

The basic pattern is familiar to anyone who's written more than a few scripts:

```powershell
try {
    Update-MgUser -UserId $user.Id -AccountEnabled:$false
}
catch {
    Write-Warning "Failed to disable $($user.UserPrincipalName): $($_.Exception.Message)"
}
```

That's fine for a script touching five accounts you're watching run live. It falls apart the moment you're looping over a few thousand users overnight and the console output scrolls off a session nobody's attached to — the only trace of the failure is whatever landed in a log you didn't set up. The real pattern I use logs failures to a file or object array as it goes, so the end of the run produces a clear list of exactly what didn't succeed and why.

```powershell
$failures = @()
foreach ($user in $users) {
    try {
        Update-MgUser -UserId $user.Id -AccountEnabled:$false
    }
    catch {
        $failures += [PSCustomObject]@{
            User  = $user.UserPrincipalName
            Error = $_.Exception.Message
            Time  = Get-Date
        }
    }
}
$failures | Export-Csv .\disable-failures.csv -NoTypeInformation
```

## Graph-specific gotchas

Microsoft Graph throttles, and it does so with a 429 response and a Retry-After header rather than a generic failure. Treating every caught exception the same way means you'll log a throttling pause as a permanent failure when it was actually just Graph asking you to slow down. I handle this by checking the exception for a 429 specifically and sleeping before retrying, rather than assuming every catch block means the operation is dead.

```powershell
catch {
    if ($_.Exception.Response.StatusCode -eq 429) {
        Start-Sleep -Seconds 30
        # retry logic here
    }
    else {
        $failures += $user
    }
}
```

Another one that catches people out: `-ErrorAction Stop`. Without it, a lot of cmdlets emit non-terminating errors that your try/catch never sees at all — the script just moves on, and you're left wondering why the catch block never fired for a failure you watched happen on screen. I set `$ErrorActionPreference = 'Stop'` at the top of bulk scripts specifically so try/catch actually catches things, rather than relying on the default per-cmdlet behavior.

## The habit that matters most

None of this is exotic. What actually separates a reliable identity script from a fragile one is treating every external call — Graph, Exchange, any