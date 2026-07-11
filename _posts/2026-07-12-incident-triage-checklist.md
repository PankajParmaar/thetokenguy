---
layout: post
title: "Incident Triage Checklist"
date: 2026-07-12
categories: ["Microsoft Defender", "Incident Response"]
tags: ["incident response", "triage checklist", "security operations"]
author: pankaj
---
The gap between a good incident response and a chaotic one usually isn't detection quality or tooling, it's whether the first fifteen minutes followed a clear, pre-agreed sequence or whether the on-call analyst started pulling logs and disabling accounts before anyone confirmed which entities were actually affected. A checklist doesn't replace judgment, but it removes the cognitive overhead of deciding what to check first while under pressure.

## Confirm before you escalate

The first step, before anything else, is confirming the alert reflects real activity rather than a false positive or an already-known benign action, like an approved pentest or a documented admin script that happens to trip a detection. This sounds obvious written down but it's the step most often skipped when an alert looks scary enough that people jump straight to escalation, only to discover an hour into a full incident bridge that the "attacker" was the infra team's own automation running on schedule.

## Scope, before containment

Once an alert is confirmed real, the next step is establishing scope: which accounts, which devices, which time window. In a Defender-centric environment this usually means pulling the full incident graph in Microsoft 365 Defender rather than working the single triggering alert in isolation, since the incident view will already show correlated activity across Defender for Endpoint, Identity, Cloud Apps, and Office 365 that gives a far more accurate picture of scope than any single alert can on its own. For Sentinel-driven investigations, this is where the entity mapping on the triggering analytics rule earns its keep, since well-mapped entities make it fast to pivot from "this account" to "everywhere else this account shows up" across the workspace.

## The checklist itself

A workable version, adapted to whatever tooling is actually in play, runs through:

- Confirm the alert reflects genuine malicious or unauthorized activity, not a known benign source.
- Identify all affected accounts, devices, and applications tied to the same incident or entity.
- Establish an approximate timeline: first suspicious activity, not just alert time.
- Check for privilege escalation or lateral movement tied to the same entities.
- Determine whether the activity is still active or has already stopped.
- Assign a severity and an owner before moving to containment.

That last point matters more than it looks. An incident without a named owner tends to get worked by whoever happened to see the alert first, and ownership handoffs during active incidents are one of the most common places critical context gets lost between shifts or between analysts.

## Severity is a decision, not a formula

Severity should reflect business impact, not just the technical alert severity Defender or Sentinel assigned automatically. A medium-severity technical alert touching a domain controller or a finance system deserves a higher actual incident severity than a high-severity alert on an isolated test machine with no sensitive data, and that judgment call belongs to a human who understands what's actually at stake, not to the automated scoring alone.

## Where triage checklists fail in practice

The most common failure is treating the checklist as a formality to complete after the interesting technical work rather than the gate that determines whether that technical work is even warranted. The second is skipping the timeline step and jumping straight to containment, which can mean shutting down a device or disabling an account before understanding whether the attacker already has a foothold elsewhere that a hasty containment action would only tip off.

A triage checklist is worth less for what it tells you to check and more for what it stops you from skipping w