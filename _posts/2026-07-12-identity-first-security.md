---
layout: post
title: "Identity-First Security"
date: 2026-07-12
categories: ["Zero Trust", "Identity as the Perimeter"]
tags: ["identity", "zero-trust", "conditional-access"]
author: pankaj
---
Identity-first security is one of those phrases that sounds like marketing until you actually try to build an access model around it, at which point it becomes a genuinely different way of designing controls compared to the network-perimeter habits most of us grew up with. The core idea is straightforward: instead of asking "where is this request coming from," you ask "who is making this request, on what device, doing what, and how confident are we in all three." Identity becomes the anchor that every other signal attaches to.

## What changes in practice

The clearest way I have found to see this shift is to compare how a single access decision gets made under each model. Under a network-first model, a user connects to the VPN, gets an internal IP, and from that point forward most internal resources trust the connection because it originated from inside the perimeter. Under identity-first, the same user authenticates directly against Entra ID for each resource, and every single request carries its own evaluation — is the account healthy, has MFA been satisfied recently, is the device compliant, is the sign-in pattern consistent with that user's normal behavior. There is no single moment where trust gets granted once and reused for the rest of the session across every resource.

Building this in a lab tenant, the practical shift shows up mostly in Conditional Access design. Policies stop being organized around network zones and start being organized around personas and resource sensitivity: this policy applies to anyone accessing a finance app, this one applies to any sign-in flagged as risky by Identity Protection, this one applies to any admin role activation. The policies are indifferent to whether the user is on the office network or on a hotel connection, because network location stopped being the variable that mattered.

## Where it actually gets hard

The hard part of identity-first security is not the Conditional Access configuration, which Microsoft has made reasonably approachable. It is the identity hygiene underneath it. Identity-first only works if the identity itself is trustworthy, which means the join process, the lifecycle management, and the credential issuance all have to be solid before the access policies built on top of them mean anything. A Conditional Access policy that checks "is this user's risk level low" is only as good as the signals feeding Identity Protection, and those signals degrade quickly if legacy authentication protocols are still enabled somewhere, because legacy auth bypasses most of the modern signal collection entirely.

The other place this gets hard is non-human identities. Service principals, managed identities, and API-to-API calls don't fit neatly into a "verify the user, check the device" model, and yet in an identity-first architecture they still need to be governed with the same rigor, because a compromised service principal with broad permissions is functionally equivalent to a compromised user account with the same permissions. Environments that build strong identity-first controls for human sign-in and leave application identities on static secrets and broad role assignments have only solved half the problem, and it's usually the half that gets the attention because it is the half users interact with directly.

Identity-first security is less a product decision and more a discipline about where you put your architectural weight. Once you accept that the network can no longer answer "should this access be allowed," identity is what's left to answer it, and every downstream control — device compliance, network segmentation, data protection — becomes a refinement of that same identity-anchored decision rather than an independent gate.
