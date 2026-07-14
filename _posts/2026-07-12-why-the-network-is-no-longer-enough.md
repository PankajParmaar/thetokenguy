---
layout: post
title: "Why the Network Is No Longer Enough"
date: 2026-07-12
categories: ["Zero Trust", "Identity as the Perimeter"]
tags: ["identity", "network-security", "zero-trust"]
author: pankaj
description: "The network perimeter was the primary control — firewalls at the edge, VPN, trust implied by location. That stopped working once apps left the data center."
image:
  path: /assets/img/og-default.png
  alt: "Why the Network Is No Longer Enough"
---
For most of my career the network perimeter was the primary security control. Firewalls at the edge, VPN concentrators for remote access, and an implicit assumption that anything inside the building or on the corporate VPN was trustworthy by default. That model made sense when applications lived in a data center you controlled and users sat at desks connected to a network you built. It stopped making sense the moment SaaS applications, remote work, and BYOD became the default rather than the exception, and yet a surprising number of environments I have looked at in labs and case studies still carry that network-first assumption baked into their access policies.

## The perimeter dissolved, the assumption didn't

The problem is not that firewalls stopped working. It is that the thing they were protecting moved. A user's mailbox, their file storage, their line-of-business apps, and their collaboration tools are now sitting in Microsoft 365, in a SaaS vendor's tenant, in an Azure subscription reachable from anywhere with a browser. None of that traffic has to cross a firewall you own on its way from the user to the resource, which means a network-based trust boundary has nothing left to enforce against. Meanwhile the credential a user types to reach that mailbox is reachable from anywhere too, so the actual point of control shifted to identity whether or not the security architecture caught up with it.

I tested this directly in a small lab tenant by comparing what a network-only control could see versus what an identity-aware control could see for the exact same sign-in. The network control, in this case a simple IP allow-list, could only ever answer one question: is this request coming from an expected range. It had no visibility into whether the account was compromised, whether MFA had been satisfied, or whether the device was managed. Conditional Access evaluating that same sign-in could factor in sign-in risk, device compliance, and application sensitivity together, and produce a decision that a network rule structurally cannot.

## What identity gives you that the network can't

Identity is portable in a way the network is not. A user's identity travels with them from the office, to their home network, to a hotel Wi-Fi, and the access decision needs to travel with it too rather than resetting to "untrusted" or "trusted" purely based on which network they happened to connect through. Identity as the control point also lets you express access policy in terms that actually match risk — this user, on this device, doing this action, against this resource — instead of the blunt instrument of "this IP range is fine." The IP range tells you nothing about the four things that actually matter for the access decision.

None of this means the network layer is irrelevant. Segmentation still limits lateral movement once someone is in, and network-level controls still catch classes of attack identity signals never see, like certain exfiltration paths or infrastructure-level probing. But as the primary gate deciding who gets to touch what, the network lost that role the day the resources it was protecting moved outside its boundary. Anyone still designing access policy with "is this request coming from inside our network" as the first and heaviest-weighted question is solving for an architecture that describes fewer and fewer of the systems people actually use day to day.
