---
layout: post
title: "Lateral Movement Detection"
date: 2026-07-12
categories: ["Microsoft Defender", "Identity Defence"]
tags: ["lateral movement", "defender for identity", "active directory security"]
author: pankaj
description: "Initial access gets the writeups because it's dramatic. What determines the blast radius is how an attacker moves after — a different detection problem."
image:
  path: /assets/img/og-default.png
  alt: "Lateral Movement Detection"
---
Initial access gets most of the attention in security writeups because it's the dramatic entry point, but the part of an intrusion that actually determines the blast radius is what happens after: how an attacker moves from the first compromised box to the systems that actually matter. Detecting that movement is a genuinely different problem from detecting the initial foothold, and it's worth understanding why the signal looks so different.

## Why lateral movement doesn't look like malware

Lateral movement techniques rely almost entirely on legitimate administrative tools and protocols: SMB, WMI, RDP, PsExec, remote PowerShell. None of that is inherently malicious, which is exactly why signature-based detection is the wrong tool for this stage of an intrusion. Defender for Identity and Defender for Endpoint approach this instead through behavioral baselining, tracking which accounts normally use which protocols against which hosts, and flagging deviations from that pattern rather than flagging the tools themselves.

A classic detection here is Defender for Identity's reconnaissance and lateral movement path analysis, which maps out the chain of accounts and machines an attacker could traverse to reach a sensitive account, like a domain admin, based on existing session data and permission structures. That mapping exists before any attack happens, as a proactive exposure view, but the same underlying data feeds real-time detections when an actual traversal pattern starts matching one of those mapped paths.

## The specific patterns worth knowing

Pass-the-hash and pass-the-ticket activity, where a captured credential material is reused across hosts without a fresh interactive logon, is one of the clearest lateral movement signals because the resulting authentication pattern doesn't match how a real interactive logon looks. Another strong signal is a single account authenticating to an unusually high number of distinct hosts in a short window, since that's consistent with an attacker sweeping a network looking for a foothold with elevated access, a pattern normal user or even normal service account behavior rarely produces. Defender for Endpoint contributes its own angle by watching for suspicious remote process creation, unusual use of admin shares, and abnormal WMI activity tied to remote execution.

## Where detection gets missed

The most common gap is incomplete sensor coverage, the same issue that shows up across most Defender for Identity discussions. If a segment of the domain isn't monitored by an MDI sensor, or if certain domain controllers were never onboarded, lateral movement that transits exclusively through that blind spot won't generate any signal at all, regardless of how anomalous the actual behavior is. The second gap is baseline pollution: if the environment already has messy, inconsistent admin practices, like IT staff routinely RDPing into arbitrary servers with domain admin credentials for convenience, the baseline that Defender for Identity builds absorbs that noise as "normal," which raises the bar for what counts as anomalous and can mask genuinely suspicious activity that happens to resemble existing bad habits.

Detecting lateral movement well is less about a single clever detection rule and more about having a clean enough baseline of normal admin behavior that a real deviation actually stands out, which is as much an IT hygiene problem as it is a Defender configuration problem.
