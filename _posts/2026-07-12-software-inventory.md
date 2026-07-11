---
layout: post
title: "Software Inventory"
date: 2026-07-12
categories: ["Microsoft Defender", "Vulnerability Management"]
tags: ["software inventory", "asset management", "defender vulnerability management"]
author: pankaj
---
Vulnerability management conversations tend to jump straight to CVEs and patch cycles, but none of that scoring means anything without an accurate answer to the much more basic question underneath it: what software is actually installed across the estate. Software inventory in Defender Vulnerability Management is the unglamorous foundation everything else sits on, and it's also where I've seen the most quietly wrong assumptions live.

## How the inventory actually gets built

Defender for Endpoint's sensor continuously collects installed application data directly from onboarded devices, no separate agent or scan required beyond the standard Defender for Endpoint onboarding. That inventory covers not just OS-level components but a wide range of third-party applications, and it's matched against Microsoft's own catalog to identify known CVEs tied to each version detected. The inventory view breaks down by product, by version, and by the specific devices that have each version installed, which is the level of detail that actually matters when you're deciding whether a given CVE is a five-device problem or a five-hundred-device problem.

End-of-life and end-of-support tracking is folded into the same inventory, flagging software versions that have stopped receiving vendor patches even if no active CVE has been published against them recently. That's a distinction worth remembering: a piece of software with zero currently listed vulnerabilities can still be a serious exposure if it's past end of support, because a new vulnerability discovered against it will simply never get patched by the vendor.

## Where the inventory has blind spots

The inventory only reflects what's installed on devices actually onboarded to Defender for Endpoint. Unmanaged devices, servers running an OS that isn't supported by the current sensor version, and anything living outside the Defender fleet entirely, like certain network appliances or IoT devices, won't show up here at all, which means the software inventory can look complete while quietly excluding a meaningful chunk of the real estate. Cross-checking device coverage against a separate CMDB or network discovery tool is the only reliable way to catch that gap, since the Defender inventory has no way to flag what it can't see.

Browser extensions and certain application plugins are another common blind spot, since they're not always captured with the same fidelity as full application installs, and a vulnerable browser extension can be a real attack path that doesn't show up cleanly in a standard software inventory pass.

## Making the inventory actually useful

The inventory is most valuable when it's queried rather than browsed, and that's where advanced hunting against the `DeviceTvmSoftwareInventory` table earns its keep, letting you ask specific questions like which devices are running a named vulnerable version of a specific application, rather than paging through the portal's UI one product at a time. Building a handful of saved queries against that table for the software your org actually cares about, rather than relying purely on the built-in recommendation feed, turns inventory from a passive report into something you can proactively check whenever a new advisory drops.

Software inventory is the least exciting part of vulnerability management and also the part that determines whether every downstream score and recommendation is grounded in reality or built on a fleet picture with holes that only surface 