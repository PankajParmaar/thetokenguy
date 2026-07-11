---
layout: post
title: "Defender for Identity Explained"
date: 2026-07-12
categories: ["Microsoft Defender", "Threat Protection"]
tags: ["defender for identity", "active directory", "threat detection"]
author: pankaj
---
The first time I set up Defender for Identity in a lab, I expected another agent-heavy deployment with a dozen moving parts. What actually surprised me was how much of it depends on getting the sensor placement right on domain controllers, and how quickly it starts flagging things you didn't know were happening on your own AD, like a service account doing NTLM auth to a box it has no business talking to.

## What it actually watches

Defender for Identity (MDI) is Microsoft's on-premises identity threat detection product, built from the old Azure ATP lineage. It runs as a sensor installed directly on domain controllers, AD FS servers, and AD CS servers, and it parses network traffic and Windows events locally rather than shipping raw traffic to the cloud. What goes up to the cloud is parsed signal: authentication patterns, group membership changes, DNS queries, LDAP binds. That distinction matters when you're explaining this to a data privacy team, because the sensor isn't a packet capture tool sitting on your DCs, it's a behavioral profiler.

The sensor builds a profile of "normal" for every identity in the domain over roughly the first few weeks. Which machines a user logs into, what times, what protocols. Once that baseline exists, MDI starts scoring deviations. A service account that has only ever authenticated via Kerberos from three application servers suddenly doing an NTLM logon from a workstation in a different subnet is exactly the kind of thing it's built to catch.

## Where it fits with everything else

MDI is one of the four pillars of Microsoft 365 Defender, alongside Defender for Endpoint, Defender for Office 365, and Defender for Cloud Apps. The value isn't really MDI in isolation, it's the correlation. An MDI alert for reconnaissance activity against a domain controller, combined with a Defender for Endpoint alert on the same host for a suspicious PowerShell execution, becomes a single incident in the Microsoft 365 Defender portal instead of two disconnected tickets that two different analysts have to manually stitch together.

## Where it breaks in practice

The most common failure I've seen in test environments isn't the detection logic, it's sensor health. If the sensor falls behind on a busy DC, or if NIC teaming is configured in a way MDI doesn't support well, you get silent gaps in coverage that don't show up unless someone is actually checking the sensor health dashboard. Another gap worth knowing: MDI only sees what touches a monitored DC. If you've got a legacy domain controller that never got a sensor installed, or a workgroup machine, that identity activity is invisible to MDI regardless of how suspicious it is.

Licensing is also worth checking early since MDI comes bundled differently depending on whether you're on Microsoft 365 E5 or buying it standalone, and I've seen teams assume coverage they didn't actually have until an audit surfaced it.

MDI is genuinely good at what it's scoped to do, which is watching the identity plane of an on-prem AD estate for the kind of lateral movement and privilege escalation patterns that show up in almost every real intrusion, but it's only as good as the sensor coverage underneath it, and that's the part that quietly rots without someone own