---
layout: post
title: "Information Protection Basics"
date: 2026-07-12
categories: ["Zero Trust", "Data Protection"]
tags: ["information-protection", "purview", "data-protection"]
author: pankaj
description: "Information protection is bigger than the sensitivity labels people associate with it. Classification, encryption, enforcement each fail silently alone."
image:
  path: /assets/img/og-default.png
  alt: "Information Protection Basics"
---
Information protection as a discipline covers more ground than the sensitivity labels most people associate with it, and it's worth separating the pieces because they solve different problems that only add up to real data protection when combined deliberately. The three pieces I've worked through in a lab Purview setup are classification, encryption, and policy enforcement, and each one fails silently if the others aren't in place around it.

## Classification comes first

Before you can protect anything, you have to know what you have and how sensitive it is, which is what classification actually does — sensitive information types matching known patterns like national ID numbers or credit card formats, trainable classifiers for content that doesn't reduce to a clean regex, and exact data match for cases where you want to check against an actual known dataset rather than a pattern. Classification without any enforcement attached is still useful on its own, purely for visibility: running content explorer or activity explorer in Purview over an existing SharePoint environment tends to surface far more sensitive content in unexpected places than most administrators expect, sitting in general-purpose document libraries with no label and no restriction at all.

## Encryption is the part that actually protects

Classification tells you what's sensitive; encryption is what stops an unauthorized party from reading it even if they get their hands on the file. Purview's rights management service ties encryption to identity, meaning a protected file isn't just password-locked, it requires the opening user to authenticate and have their identity checked against the permissions embedded in the file, which can include restrictions on printing, copying, or forwarding independent of whatever platform the file currently sits in. I tested this by encrypting a test document with a restrictive label and then moving it outside the tenant entirely — the encryption held, and opening the file required authentication against the original tenant even from an external environment, which is the actual guarantee this layer provides that a file-share permission alone cannot.

## Policy enforcement closes the loop

Classification and encryption still need something actively checking that policy is being followed, which is where DLP policies, auto-labeling policies, and conditional access based on data sensitivity come in. Without enforcement, a user can create sensitive content, have it correctly classified by an automatic rule, and still email it to a personal account unrestricted, because classification alone doesn't block anything — it only informs whatever policy is watching for that classification and configured to act on it. Enforcement is the step that turns "we know this is sensitive" into "we stopped this from leaving."

## Where the whole system tends to fall apart

The failure mode I've seen described repeatedly, and reproduced in test scenarios, is building out strong classification and encryption capability and then leaving enforcement policies in audit-only mode indefinitely because turning on active blocking risks disrupting a workflow that was never formally documented — a monthly export of customer records to a third-party billing vendor over unencrypted email, running for years, that only shows up in the audit log because nobody had reason to look at it until the DLP policy started tagging it. Audit mode is a reasonable and necessary phase for tuning false positives, but it isn't a permanent state, and a lot of information protection programs get stuck there because moving to active enforcement requires someone to own the risk of breaking that export, and that ownership tends to sit with whoever configured the policy rather than being a shared organizational decision. The honest measure of an information protection program's maturity isn't how granular the classification 