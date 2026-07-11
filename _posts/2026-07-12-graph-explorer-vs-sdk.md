---
layout: post
title: "Graph Explorer vs SDK"
date: 2026-07-12
categories: ["PowerShell & Graph", "Graph API"]
tags: ["microsoft-graph", "graph-explorer", "powershell-sdk", "api"]
author: pankaj
---
Every Graph query I've ever written in a script started life as a query I tested somewhere else first, because writing raw Graph calls blind and debugging permission errors inside a PowerShell script is a slower path than confirming the query shape works before it goes anywhere near production code. Knowing when to reach for Graph Explorer versus the PowerShell SDK directly is less about one being better and more about which stage of the work you're actually in.

## Graph Explorer as the scratchpad

Graph Explorer (developer.microsoft.com/graph/graph-explorer) is a browser-based tool for firing raw HTTP requests against Graph and seeing the response immediately, without writing a line of PowerShell. It's the fastest way to answer "does this endpoint even return the field I think it does" or "what does the nested object actually look like" before committing to a script structure around it.

```
GET https://graph.microsoft.com/v1.0/users?$select=displayName,signInActivity
```

I use it constantly for exploring a new endpoint's response shape, checking whether a field exists in v1.0 or only in the beta endpoint, and testing OData filter syntax before hardcoding it into a script. Getting `$filter` syntax wrong in Graph Explorer costs you a failed request and a readable error message. Getting it wrong buried inside a fifty-line PowerShell script costs you twenty minutes of misdirected debugging wondering if the problem is your PowerShell logic instead.

## The SDK for anything that needs to run more than once

The Microsoft.Graph PowerShell SDK is where actual automation lives — anything that needs to loop, handle errors, run on a schedule, or integrate with other PowerShell logic. It also handles pagination, authentication token refresh, and throttling retry logic that you'd otherwise have to hand-roll if calling the raw API directly with Invoke-RestMethod.

```powershell
Connect-MgGraph -Scopes "User.Read.All"
Get-MgUser -All -Filter "department eq 'Finance'"
```

That `-All` switch alone is worth the SDK over raw HTTP calls — Graph paginates results by default, and manually following `@odata.nextLink` in a raw script is exactly the kind of boilerplate the SDK exists to remove.

## Where each one breaks down

Graph Explorer's permission model is tied to whatever account is signed into the browser session, which means testing there doesn't always reflect what an app registration with application permissions will actually see — a query that works fine in Explorer under your own delegated admin permissions can fail entirely once it's running unattended under a service principal with a narrower application permission grant. I've been caught by this more than once, building a script against a query that tested clean in Explorer, only to have it fail silently in the scheduled job because the app registration's consented permissions didn't match my personal ones.

The workflow that's held up for me is Explorer first for shape and syntax, SDK second for anything that needs to actually run unattended, and always a separate permission check against the exact identity — user or service principal — that will run the production version, rather than assuming Explorer's green checkmark means the script is ready to ship.
