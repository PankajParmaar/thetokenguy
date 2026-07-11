---
layout: post
title: "Named Locations and Trusted IPs: One of the Most Misused Controls in Conditional Access"
date: 2026-07-12
categories: ["Microsoft Entra", "Conditional Access"]
tags: ["microsoft entra", "conditional access", "named locations", "trusted ip", "identity security", "zero trust", "powershell"]
author: pankaj
---
# Named Locations and Trusted IPs: One of the Most Misused Controls in Conditional Access

Open almost any mature Microsoft 365 tenant and you'll eventually find a Conditional Access policy that says "require MFA for all users except trusted locations." Sometimes that's a deliberate design decision. More often, it's a policy that was written when the company occupied a single office with a single internet circuit, and it's still sitting in the policy list unchanged three office moves and one ISP migration later. The problem isn't Named Locations themselves — the problem is treating them as an identity signal instead of what they actually are, a network attribute that Conditional Access evaluates alongside dozens of other signals. An IP address tells you where a connection entered Microsoft's edge. It tells you very little about whether the identity behind that connection should be trusted.

## A Named Location is just another input

Conditional Access doesn't think in terms of "safe networks." It evaluates a collection of signals before deciding whether Microsoft Entra should issue a token, and network location is simply one of those signals. Whether the request originated from a corporate office, an Azure Virtual Desktop host, a branch office, a VPN concentrator, or somebody's home broadband makes no difference to the policy engine — it's another condition to evaluate alongside user risk, authentication strength, device compliance, and everything else available during sign-in. The moment a Named Location becomes a shortcut for trust, policy design starts drifting away from Zero Trust.

## Trusted IPs are inherited assumptions

Most Named Location strategies are inherited from the on-premises era, when the thinking was straightforward: users inside the corporate network were probably legitimate, users outside probably weren't. Exchange, SharePoint, and domain controllers all lived behind the same perimeter, internet breakout happened through one or two firewalls, and seeing the corporate public IP actually meant something. That model falls apart once identities, applications, and network edges stop living in the same place — users work from anywhere, devices constantly change networks, SD-WAN shifts internet breakout between sites, Secure Web Gateways proxy traffic through cloud POPs, and Azure Virtual Desktop sessions can make a user sitting in Bengaluru appear to originate from Singapore. The identity hasn't changed; the network path has, and Conditional Access sees the path, not the person.

## VPNs make location even noisier

One of the quickest ways to confuse yourself is to assume the reported sign-in location represents where the user actually is. A user sitting in London may authenticate through a VPN gateway in Amsterdam, exit through a Secure Web Gateway in Frankfurt, and arrive at Microsoft through infrastructure registered in Dublin — every system involved reports a different location, and none of them are necessarily wrong. Once VPNs, ZTNA platforms, cloud proxies, SD-WAN, and regional internet breakout become part of the architecture, IP geolocation turns into infrastructure telemetry rather than user telemetry. It's useful context, but it's a poor foundation for deciding whether an authentication request should be trusted.

## Excluding trusted locations deserves scrutiny

A common Conditional Access policy requires MFA for all users accessing all cloud apps, except from trusted locations. There's nothing technically wrong with that configuration — the question is whether the assumption behind it is still valid. You're effectively saying that requests from those public IP ranges deserve less scrutiny than requests from everywhere else, and five years ago that often reflected reality. Today, many organizations require MFA regardless of network location and use Named Locations for more targeted scenarios instead — Identity Protection, administrative access, session controls, or country-based restrictions — rather than as a blanket trust signal.

## IPv4 ranges age badly

Named Locations rarely become a problem because they're configured incorrectly; they become a problem because they're never revisited after the network around them changes. An office relocates, the networking team changes ISPs, a VPN platform gets replaced, Azure Firewall gets rebuilt, ExpressRoute is redesigned — and months later someone notices that users have been bypassing, or unnecessarily triggering, Conditional Access because an old public IP range is still marked as trusted. Unlike certificates, Named Locations don't expire. They quietly become inaccurate instead.

## Auditing Named Locations

The following script inventories every Named Location and highlights trusted IP ranges that deserve review.

```powershell
Connect-MgGraph -Scopes "Policy.Read.All"

$locations = Get-MgIdentityConditionalAccessNamedLocation

$report = foreach ($location in $locations) {

    if ($location.AdditionalProperties.'@odata.type' -eq "#microsoft.graph.ipNamedLocation") {

        [PSCustomObject]@{
            Name           = $location.DisplayName
            Trusted        = $location.IsTrusted
            IPv4Ranges     = ($location.IpRanges.CidrAddress -join ", ")
            RangeCount     = $location.IpRanges.Count
            Created        = $location.CreatedDateTime
            Modified       = $location.ModifiedDateTime
        }
    }
}

$report |
Sort-Object Name |
Format-Table -AutoSize

# Optional
$report | Export-Csv ".\NamedLocationAudit.csv" -NoTypeInformation
```

Now compare the output against your actual WAN architecture. It's common to find locations representing offices that closed years ago, VPN ranges that disappeared during a migration, or cloud egress addresses with no ticket, change record, or comment field explaining why they were added.

## Mapping location dependencies

Knowing which Named Locations exist is only half the job. The next step is understanding which Conditional Access policies still depend on them, because it's surprisingly common to discover Named Locations that haven't been referenced by a policy in years, or policies still pointing at a location that was decommissioned when the branch office closed but never removed from the exclusion list.

```powershell
Connect-MgGraph -Scopes "Policy.Read.All"

Get-MgIdentityConditionalAccessPolicy |
ForEach-Object {

    [PSCustomObject]@{
        Policy           = $_.DisplayName
        IncludeLocations = ($_.Conditions.Locations.IncludeLocations -join ", ")
        ExcludeLocations = ($_.Conditions.Locations.ExcludeLocations -join ", ")
        State            = $_.State
    }

} |
Format-Table -AutoSize
```

## Operational reality

Named Locations have a habit of becoming historical artifacts. Someone migrated to a new ISP but forgot to update the public IP range. The networking team deployed a new VPN concentrator without telling the identity team. A Secure Web Gateway changed every user's apparent source country overnight. Remote administrators started coming through Azure Virtual Desktop while Conditional Access still assumed they were working from headquarters. Six months later the help desk is fielding tickets about Conditional Access "behaving inconsistently," when the real problem is that the network assumptions baked into the policies no longer match the network that actually exists.

An IP address tells you where a request entered your environment. It doesn't tell you whether the identity behind that request deserves to be trusted — that's a decision a static CIDR block has no business making on its own.
                                                                                                                                                                                                                                                                                                                                                                                                                                                                    