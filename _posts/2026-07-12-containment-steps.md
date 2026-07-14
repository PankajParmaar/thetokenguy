---
layout: post
title: "Containment Steps"
date: 2026-07-12
categories: ["Microsoft Defender", "Incident Response"]
tags: ["containment", "incident response", "defender for endpoint"]
author: pankaj
description: "Containment is where incident response stops observing and starts interfering, and every action tips your hand. How to isolate without pushing further."
image:
  path: /assets/img/og-default.png
  alt: "Containment Steps"
---
Containment is the step where incident response stops being purely observational and starts actively interfering with an attacker's access, and that shift changes the risk calculus entirely. Every containment action tips your hand, and doing it too early or too crudely can push an attacker to accelerate, destroy evidence, or move to a foothold you don't know about yet.

## Isolate the device, not the investigation

Defender for Endpoint's device isolation is the most direct containment lever available, cutting a compromised machine off from network communication while still allowing Defender's own management channel through, so investigators can keep collecting forensic data and running live response commands on the isolated machine even after it's cut off from everything else. That distinction, isolated from the network but not isolated from the tooling investigating it, is what makes this a genuinely useful containment action rather than a blunt instrument that also blinds the responders.

The decision on when to isolate is a real trade-off. Isolating immediately on first detection stops ongoing damage but can alert an attacker who's monitoring for exactly that kind of response, prompting them to burn what access they have left rather than let it be reclaimed cleanly. Waiting to gather more scope first risks letting an active attacker continue moving during that window. There's no universal right answer here, it depends on what's already at stake and how confident you are the attacker isn't already aware they've been spotted.

## Credential-based containment

Disabling or resetting the credentials of any compromised account is usually the second major lever, and it needs care around session and token invalidation, not just a password reset. Resetting a password without also revoking existing refresh tokens and active sessions leaves an attacker who already has a valid token holding access that survives the reset entirely, which is a gap that's caught out more than a few response efforts that assumed a password change alone was sufficient.

For accounts suspected of being used for lateral movement or persistence, checking for and removing any newly created app registrations, OAuth grants, or forwarding rules tied to that account matters just as much as the credential reset itself, since those are common persistence mechanisms that survive a simple password change untouched.

## Network-level containment

Beyond individual device isolation, broader network containment might mean blocking specific IOCs, malicious IPs or domains, at the firewall or through Defender for Endpoint's indicator blocking feature, which pushes a block for a specific file hash, IP, or URL across the entire managed fleet rather than one device at a time. This is useful when scope has expanded past a single machine and the same indicator is showing up across multiple hosts.

## The order matters more than any single action

A reasonable containment sequence goes: isolate confirmed compromised endpoints, disable and fully invalidate sessions for compromised accounts, block confirmed malicious indicators at the perimeter, and only then move to eradication activities like removing persistence mechanisms or malware. Doing eradication before containment is complete risks tipping off an attacker who still has access elsewhere, prompting a scramble to burn evidence before the full scope has even been mapped.

## Where containment goes wrong

The most common mistake is containing the first compromised system found without checking whether the same account or a related account has access elsewhere that hasn't been addressed yet, effectively closing one door while leaving another wide open. The second is containing too visibly and too early relative to how much scope is actually understood, which can convert a quiet, ongoing investigation into a rushed one the moment the attacker notices they've lost access somewhere.

Containment is as much a judgment call about attacker awareness as it is a technical action, and the tooling only gives you the lever, not the timing.
