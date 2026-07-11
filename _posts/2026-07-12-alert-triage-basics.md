---
layout: post
title: "Alert Triage Basics"
date: 2026-07-12
categories: ["Microsoft Defender", "Threat Protection"]
tags: ["alert triage", "soc", "microsoft defender"]
author: pankaj
---
Anyone who's spent time in front of a Defender alert queue knows the real problem isn't a lack of detections, it's deciding which of the forty alerts sitting there actually deserve the next twenty minutes of your attention. Triage is the unglamorous skill that separates a functioning security operation from one that's just generating a Teams channel full of alert notifications that pile up faster than anyone can work through them.

## Severity is a starting point, not an answer

Microsoft 365 Defender assigns severity (informational, low, medium, high) based on the detection logic behind each alert, but that severity is calculated without knowledge of your specific environment. A "medium" alert for a suspicious sign-in from an unfamiliar location means something very different for a company with a fully remote, globally distributed workforce than it does for one where everyone badges into the same building every day. The first real triage step is applying environmental context that Microsoft's scoring can't have: is this account tied to a VIP, is this device managed or BYOD, does this user have a travel pattern that explains the geography.

## Incident correlation over individual alerts

One habit worth building early is triaging at the incident level, not the alert level. Microsoft 365 Defender automatically groups related alerts across Defender for Endpoint, Identity, Office 365, and Cloud Apps into a single incident graph when it can establish a shared entity, like the same user or device appearing in multiple alerts. Working alerts individually, ignoring the incident grouping, is how you end up manually recreating a correlation the platform already built for you. Open the incident, look at the alert story it's assembled, and only drop down into individual alert detail when you need the specific evidence for a decision.

## A workable triage order

In practice, a decent triage pass runs through a few checks in sequence:

- Is this entity (user, device, mailbox) already flagged in an open incident elsewhere.
- What's the confidence and evidence behind the alert, not just the severity label.
- Is there a known benign explanation already documented, like a scheduled pentest or an approved admin script.
- Does this alert correlate with anything from another Defender workload in the same time window.

That order matters because it front-loads the cheap checks. Checking for an existing incident or a documented false positive takes seconds and can close out a chunk of the queue before you spend real analysis time on anything.

## Where teams get stuck

The most common trap is treating every alert with equal ceremony regardless of context, which burns analyst time on low-value investigation and leaves the genuinely dangerous alert sitting in the queue behind five informational ones. The second trap is closing alerts as false positive without documenting why, which means the next analyst who sees a similar pattern has to redo the same investigation from scratch. A short, honest note on the closure reason is worth more than people give it credit for, because triage quality compounds over time only if the reasoning behind past decisions is actually recoverable.

Triage isn't about clearing the queue fast, it's about building a rep