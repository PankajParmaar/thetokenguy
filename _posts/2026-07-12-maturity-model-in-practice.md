---
layout: post
title: "Maturity Model in Practice"
date: 2026-07-12
categories: ["Zero Trust", "Principles & Architecture"]
tags: ["zero-trust", "maturity-model", "security-architecture"]
author: pankaj
---
CISA and Microsoft both publish Zero Trust maturity models with the same three-stage shape: traditional, advanced, optimal. They are useful as a shared vocabulary, but the version that actually helps in a working tenant is less about which stage you claim to be in and more about being honest per pillar, because almost no organization sits at a uniform maturity level across identity, device, network, application, and data at the same time.

## The pillars rarely move together

I have set up a lab tenant to walk through this deliberately, treating each pillar separately rather than assigning the whole environment one grade. Identity was the easiest to push toward "optimal" — Conditional Access with risk-based sign-in evaluation, phishing-resistant MFA, and PIM for privileged roles gets you most of the way there without enormous cost. Device trust lagged behind, sitting closer to "advanced," because compliance policies existed but enforcement wasn't consistently blocking non-compliant devices from every sensitive resource, only flagging them. Network segmentation was the weakest pillar by far, still resembling "traditional" with a flat structure that Conditional Access could restrict access into but couldn't meaningfully segment once someone was inside.

This unevenness is the normal state, not an exception, and it is the reason maturity models fail when treated as a single organizational score. A report that says "we are Advanced maturity" hides the fact that one pillar might be optimal and another might be barely started, and that gap is exactly where an attacker who understands the architecture will look first.

## Where the model becomes useful

The maturity model earns its keep when used as a diagnostic per pillar rather than a badge for the whole program. For identity, the traditional-to-optimal axis is roughly: password-only auth, to MFA everywhere, to risk-based and phishing-resistant authentication with continuous evaluation. For devices, it runs from unmanaged endpoints, to basic compliance policies, to real-time device risk signals feeding directly into access decisions. For networks, it moves from flat trusted zones, to segmented VLANs with some inspection, to fully segmented microperimeters with per-application access brokered by identity rather than IP range. Mapping your actual tenant against these axes, pillar by pillar, produces a more honest picture than any composite score, because it tells you exactly where the next investment of time and budget should go.

## The mistake of chasing the label

The trap I would flag for anyone building this out in their own environment, lab or otherwise, is optimizing for the label rather than the underlying control. Chasing "optimal" on identity by turning on every available Entra feature is not the same as verifying those features are actually enforced consistently across every app and every user population, including the guest accounts and service principals that tend to get forgotten in maturity assessments. A tenant can score well on paper while carrying legacy authentication protocols still enabled for a handful of line-of-business apps — an on-prem accounting system with a basic-auth integration, a scanner-to-email service account, a vendor portal that never got migrated off POP3 — because disabling legacy auth tenant-wide broke one of them in a prior attempt and the exception has been renewed at every review since. T