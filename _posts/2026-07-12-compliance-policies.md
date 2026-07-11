---
layout: post
title: "Compliance Policies"
date: 2026-07-12
categories: ["Microsoft 365", "Endpoint Management"]
tags: ["intune", "compliancepolicies", "endpointmanagement", "conditionalaccess"]
author: pankaj
---
Intune compliance policies get their real teeth from what they're connected to, not from what they check on their own. A compliance policy by itself is just an evaluation — it looks at a device's configuration against a set of rules and marks the device compliant or non-compliant in Entra ID. Nothing actually happens as a consequence of non-compliance unless a Conditional Access policy is built to key off that compliance state, which means the policy design work really has two halves that need to be built together rather than the compliance rules being configured in isolation and Conditional Access bolted on as an afterthought.

## What a policy actually checks

Compliance rules vary by platform but generally cover a similar set of concerns: minimum OS version, whether the device has encryption enabled (BitLocker on Windows, device encryption on Android/iOS), whether a passcode or biometric lock is required and how complex it must be, whether the device is jailbroken or rooted, and whether specific security software or Defender for Endpoint risk score thresholds are being met. Each rule can be set to mark a device non-compliant immediately or after a grace period, and the grace period matters practically — setting it to zero on a newly deployed policy, before users have had any chance to remediate (enable BitLocker, update the OS), tends to non-compliant an entire device population overnight and generate a wave of "why can't I access email" tickets that were entirely avoidable with a week or two of grace period built in.

## The compliance-to-Conditional-Access handoff

Once a device is marked compliant or non-compliant, a Conditional Access policy targeting "require device to be marked as compliant" is what actually gates access to resources based on that state. This is the mechanism that turns "we have a compliance policy" into "non-compliant devices can't reach Exchange Online or SharePoint," and skipping this half of the setup is a surprisingly common gap — a tenant with well-designed compliance policies but no Conditional Access referencing them has a dashboard that shows non-compliant devices, and nothing else, which is a monitoring tool at that point, not an access control.

## Marking non-compliance actions beyond blocking access

Compliance policies themselves also support their own action set, independent of Conditional Access: sending an email notification to the user when they go non-compliant, and, after a configurable additional delay, marking the device for retire or remotely locking it. These actions are worth using deliberately rather than jumping straight to the harshest available response — a first-stage notification giving the user a clear, specific reason ("your device is missing the required OS update") and a few days to self-remediate produces far less support load than a policy that silently blocks access with no explanation, leaving the user to guess why Outlook stopped syncing.

## Compliance policy conflicts across enrollment types

A subtlety worth knowing: compliance policy assignment can behave differently depending on enrollment method and platform, particularly around Android's split between fully managed and work-profile devices, where a compliance rule checking for something like a device-wide passcode doesn't map cleanly onto a work-profile device that only has a separate work-profile-specific unlock code. Testing compliance policies against a representative sample of each actual enrollment type in your environment, rather than assuming a single rule set behaves identically everywhere, catches this kind of platform-specific mismatch before it reaches the whole fleet.

Compliance policies are ultimately a signal-generation layer, and the value they deliver is entirely proportional to how deliberately that signal gets consumed downstream by Conditional Access — treating the compliance policy configuration as the finish line, rather than the halfway point, is the most common reason a well-built policy ends up doing far less than whoever designed it assumed it would.
