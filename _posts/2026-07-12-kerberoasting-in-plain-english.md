---
layout: post
title: "Kerberoasting in Plain English"
date: 2026-07-12
categories: ["Microsoft Defender", "Identity Defence"]
tags: ["kerberoasting", "kerberos", "service accounts"]
author: pankaj
description: "Kerberoasting explained without the ticket-flow diagrams first: stealing service account passwords by asking AD for what it hands out, then cracking it."
image:
  path: /assets/img/og-default.png
  alt: "Kerberoasting in Plain English"
---
Kerberoasting gets explained in a lot of overly technical writeups full of Kerberos ticket flow diagrams before anyone actually says what the attack is for, so here's the plain version first: it's a way to steal service account passwords by asking Active Directory for something it's designed to hand out to anyone who asks, then cracking that offline at your own pace with no risk of triggering a lockout.

## The part of Kerberos that makes this possible

When a user wants to access a service that's registered with a Service Principal Name, like a SQL Server instance, their machine requests a service ticket from the domain controller for that specific SPN. That ticket gets encrypted using a key derived from the password of the account the service is running under, usually a service account. Critically, any authenticated domain user can request a service ticket for any SPN registered in the domain. That request isn't privileged, it's a completely normal part of how Kerberos service authentication works, which is exactly why this isn't flagged as an error or a permission violation at the protocol level.

Once an attacker has that ticket, they take it offline and try to crack it, since the ticket is encrypted with a key derived from the service account's password. If that password is weak, which is extremely common for service accounts that were set up once years ago and never rotated, the crack succeeds, and the attacker now has the plaintext password for that service account without ever touching a keyboard on a domain-joined machine again.

## Why service accounts are the perfect target

Service accounts are the sweet spot for this attack for a specific reason: they're set once during initial setup, rarely rotated because rotating them risks breaking whatever service depends on them, and frequently granted more privilege than they actually need because the admin who set it up added Domain Admin to sidestep a permissions error during testing and never circled back to scope it down before go-live. A service account with a stale, weak password and unnecessary domain admin membership is close to the ideal outcome from an attacker's perspective, and it's a combination that shows up more often in real environments than anyone likes to admit.

## Detecting and reducing exposure

Defender for Identity detects Kerberoasting by watching for the specific pattern of a single account requesting an unusually high volume of service tickets in a short window, especially tickets using a weaker encryption type like RC4 rather than the stronger AES options, since RC4 tickets are meaningfully faster to crack offline and their use is itself a signal worth investigating even outside a clear volume anomaly.

The actual fix, more durable than any single detection rule, is treating service account passwords the same way you'd treat any other credential: long, randomly generated, and rotated on a real schedule, ideally using Group Managed Service Accounts where the platform handles rotation automatically rather than relying on a human remembering to do it. Enforcing AES-only ticket encryption where the environment supports it removes the RC4 shortcut entirely, and auditing which accounts actually need their SPNs registered at all closes off targets that don't need to exist in the first place.

Kerberoasting is a good reminder that some of the most durable attack techniques don't exploit a flaw, they exploit a design decision working exactly as intended, combined with the fact that most service 