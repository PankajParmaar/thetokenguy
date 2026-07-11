---
layout: post
title: "Attack Surface Reduction"
date: 2026-07-12
categories: ["Microsoft Defender", "Threat Protection"]
tags: ["attack surface reduction", "defender for endpoint", "hardening"]
author: pankaj
---
Attack surface reduction rules are one of those Defender features that get mentioned in every hardening checklist but rarely get the attention they deserve, mostly because turning them on in audit mode and actually reading the results is tedious work that competes with everything else on a security team's plate.

## What ASR rules actually do

ASR rules are a set of pre-built behavioral blocking rules delivered through Microsoft Defender for Endpoint that target the specific techniques malware and living-off-the-land attacks rely on, rather than trying to detect malicious files by signature. Instead of asking "is this file bad," ASR asks "is this specific behavior ever legitimate." Rules like blocking Office applications from creating child processes, blocking credential stealing from the LSASS process, or blocking executable content from email clients and webmail are targeting technique, not payload. That's the appeal: a rule blocking Office from spawning child processes stops a huge range of macro-based droppers regardless of what the actual second-stage payload is.

Each rule can be set to block, audit, warn, or disabled, and every rule is identified by a GUID rather than a friendly name in the underlying policy, which is a small but constant source of confusion when you're troubleshooting through PowerShell or Intune and have to cross-reference the GUID list from Microsoft's docs every time.

## Why audit mode is not optional

Turning ASR rules straight to block mode in a live environment is a reliable way to break something you didn't know depended on the blocked behavior. Audit mode logs what would have been blocked without actually blocking it, and that data lands in the Defender for Endpoint advanced hunting tables where you can query it before committing to enforcement. The rule blocking process creations from PSExec and WMI commands, for instance, is exactly the kind of thing that looks great in theory until you discover the systems management tooling your infra team relies on uses the same mechanism.

Running rules in audit for a minimum of a few weeks, across a representative sample of device groups rather than just the security team's own laptops, is the difference between a rollout that holds and one that gets rolled back after the helpdesk gets flooded with tickets.

## Where the friction shows up

The rule most likely to cause pushback is blocking win32 API calls from Office macros, because plenty of legitimate business processes still run on old macro-driven Excel workflows built by someone who left the company years ago, with no budget line ever opened to rebuild them. Similarly, the credential theft rule targeting LSASS can conflict with certain legitimate remote access and backup agents that read process memory for reasons unrelated to credential dumping, so exclusions end up necessary and need to be scoped tightly rather than blanket-applied to whole device groups.

Reporting also needs a second look. ASR events show up in the Defender portal's attack surface reduction rules report, but the raw hunting data in `DeviceEvents` gives far more granular context on exactly what triggered a block, which matters when you're trying to explain to an application owner why their tool got flagged.

ASR rules are genuinely one of the higher leverage things you can turn on in Defender for Endpoint, but the leverage only pays off if someone actually reads 