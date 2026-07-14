---
layout: post
title: "The Three Principles"
date: 2026-07-12
categories: ["Zero Trust", "Principles & Architecture"]
tags: ["zero-trust", "security-architecture", "least-privilege"]
author: pankaj
description: "Verify explicitly, use least privilege, assume breach — Zero Trust's principles read like a poster. Each maps to decisions easy to state, hard to finish."
image:
  path: /assets/img/og-default.png
  alt: "The Three Principles"
---
Microsoft's Zero Trust documentation, and most of the vendor literature that copies it, boils the model down to three principles: verify explicitly, use least privilege access, and assume breach. They read like a poster on a wall, but each one maps to a specific set of decisions you have to make in a real tenant, and each one is easy to state and genuinely hard to implement completely.

## Verify explicitly

This principle says that every access request should be authenticated and authorized based on all available data points, not just a valid password. In practice that means the Conditional Access engine at sign-in time is evaluating identity risk, device compliance, location, and application sensitivity together, rather than trusting a correct password as proof of identity. The gap I see most often in test tenants and in write-ups from other practitioners is stopping at "MFA is on" and calling verification complete. MFA answers one question — did the right person type the right code — but it says nothing about whether the device is compromised or whether the sign-in pattern is anomalous for that user. Explicit verification is a composite judgment, and building it out means wiring Identity Protection risk signals and device compliance into every Conditional Access policy, not just the ones for admin roles.

## Least privilege access

This is the principle that shows up cleanest in the policy documentation and breaks down fastest in a real tenant, because least privilege is a moving target. The access a person needs on their first day is different from what they need six months in, and standing permissions tend to accumulate rather than shrink as people change roles — a user who moves from the finance team to sales still carries the finance security group and its SharePoint site access, because offboarding a role change was never built into the transfer process the way offboarding a termination was. Just-In-Time access through PIM, time-bound role activation, and entitlement management packages with expiration dates are the tools that make least privilege enforceable rather than aspirational. Without them, least privilege degrades into a one-time access review that gets stale within a quarter.

## Assume breach

The third principle is the one that gets skipped most often because it requires admitting failure is inevitable rather than preventable. Assume breach means designing your environment so that a single compromised credential or endpoint cannot move laterally and reach everything. Concretely that is microsegmentation on the network side, tiered administration so a compromised standard user account cannot touch Tier 0 assets, and logging and detection built to catch lateral movement rather than just the initial compromise. In a lab setup, I have found the easiest way to test whether "assume breach" is real in an environment is to pick one compromised low-privilege account and ask how far it could actually go — through group memberships, through shared service accounts, through overly broad RBAC assignments in Azure. If the answer is "quite far," the assume-breach principle exists on paper only.

## Why the three have to move together

The trap in treating these as a checklist is implementing them in isolation. Verify explicitly without least privilege means you have a well-authenticated user who still holds standing global admin rights. Least privilege without assume-breach means a properly scoped account still sits on a flat network where lateral movement is trivial once that account is compromised. The three principles only produce a Zero Trust posture when they compound — tight verification narrows who gets in, least privilege narrows what they can touch, and assume-