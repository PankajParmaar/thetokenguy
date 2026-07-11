---
layout: post
title: "VPN vs Zero Trust Network Access"
date: 2026-07-12
categories: ["Zero Trust", "Network Access Control"]
tags: ["vpn", "ztna", "network-security"]
author: pankaj
---
The pitch for Zero Trust Network Access is usually framed as "VPN is dead, replace it," which oversimplifies a decision that actually comes down to what kind of access grant each model produces and whether that grant matches what you're trying to protect. I've set both up in a lab to compare them directly rather than take the marketing framing at face value, and the difference is less about which technology is newer and more about the shape of the trust each one hands out.

## What a VPN actually grants

A traditional VPN authenticates the user once, at connection time, and then places their device on the corporate network segment, or a routed equivalent of it. From that point forward, the device can typically reach a broad range of internal resources based on network routing and firewall rules, not based on what that specific user actually needs for their specific job. The access decision was made once, at the login prompt, and everything after that is governed by network topology rather than continuous evaluation of who's asking and for what. If that VPN session or the credential behind it gets compromised, the blast radius is whatever that network segment can reach, which in a lot of legacy topologies is uncomfortably large.

## What ZTNA grants instead

ZTNA, whether that's Entra Private Access, Zscaler Private Access, or a similar broker, doesn't put the user on the network at all. It brokers access to individual applications one at a time, evaluating identity, device compliance, and policy for each connection rather than once at login. The user never gets an IP on the internal network segment; they get a tunnel scoped to the specific app they're authorized for, and that authorization is re-evaluated continuously rather than assumed valid for the life of a VPN session. In my lab test, connecting to an internal web app through a ZTNA broker versus through a VPN made the difference concrete: with the VPN, the same client could reach a handful of other internal services on the same subnet with no additional check. With ZTNA, reaching a different app required its own explicit policy grant, and there was no ambient network reachability to piggyback on.

## Where each one still makes sense

VPN isn't obsolete for every scenario. Site-to-site connectivity, certain legacy protocols that ZTNA brokers don't cleanly support, and environments with a genuinely small number of tightly controlled applications can still function fine on a well-segmented VPN. The problem isn't the technology itself, it's the tendency for VPN deployments to get broader over time as more resources get added to the same trusted segment, because it's operationally easier to add a new subnet to the existing VPN route than to stand up per-app access control for a single new service.

## The real trade-off

The honest comparison isn't "ZTNA is more secure" as an abstract claim, it's that ZTNA structurally limits blast radius per compromised credential in a way flat VPN access can't, because there's no shared network segment to laterally move across in the first place. What it costs you is complexity — every application needs its own broker configuration, its own policy, and typically more upfront work per app than just routing it onto an existing VPN segment. For an organization migrating off VPN, the realistic path is app-by-app, prioritizing the resources where a compromised credential would do the most damage, rather than a wholesale rip-and-replace that tries to broker every legacy service on day one and stalls out on the handful that don't fit the model cleanly.
