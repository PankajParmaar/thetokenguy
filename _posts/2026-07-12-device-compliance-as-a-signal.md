---
layout: post
title: "Device Compliance as a Signal"
date: 2026-07-12
categories: ["Zero Trust", "Device & Endpoint Trust"]
tags: ["device-compliance", "intune", "conditional-access"]
author: pankaj
---
Device compliance gets treated in a lot of tenants as a binary gatekeeper — compliant devices get in, non-compliant ones get blocked — and that framing misses what makes it actually useful in a Zero Trust architecture. Compliance state is a signal, one input feeding a broader access decision, not a standalone security control that does its job the moment it is switched on.

## What compliance actually measures

An Intune compliance policy checks a defined set of conditions on a managed device: is disk encryption enabled, is the OS version above a minimum threshold, is a passcode set, is the device jailbroken or rooted, is antivirus running and up to date. Pass all of them and the device gets marked compliant in Entra ID; fail any one and it's marked non-compliant. That compliance state then becomes a claim available to Conditional Access, which is where it earns its keep — a policy can require "compliant device" as a grant control for any sign-in to a sensitive app, meaning even a correct password and successful MFA won't be enough if the device itself fails the bar.

The thing worth sitting with is what compliance does not measure. It doesn't mean the device is free of malware beyond what the configured antivirus catches. It doesn't mean the user hasn't been phished on that device five minutes earlier. It doesn't mean the device isn't sitting on a compromised home network. Compliance is a point-in-time snapshot against a fixed checklist, refreshed on whatever sync interval you've configured, typically measured in hours rather than being continuously real-time. Treating a green "compliant" badge as proof of a trustworthy device overstates what the check actually verifies.

## Where it fits in the bigger decision

In a lab tenant, I set device compliance as one of several grant controls in a Conditional Access policy rather than the only one, alongside MFA and, for sensitive apps, a sign-in risk check from Identity Protection. That layering matters because each signal covers a different failure mode. Compliance catches a device configuration that's fallen out of policy. Sign-in risk catches a credential being used from an unusual context. MFA catches a stolen password on its own. None of them substitutes for the others, and a policy that relies on device compliance alone would still let a compliant device with a compromised user session straight through.

## Where compliance signals get stale or gamed

The practical failure I've seen written up, and reproduced in test scenarios, is compliance drift between sync intervals. A device can fall out of compliance the moment a user disables encryption or lets an OS update lapse, but Entra ID won't reflect that until the next Intune check-in reports it, leaving a window where the device is actually non-compliant but still presents as compliant to Conditional Access. Tightening the sync interval helps but never closes the gap entirely, because there's always some delay between state change and detection.

The other gap is scope. Compliance policies only apply to devices actually enrolled in Intune. BYOD and unmanaged devices have no compliance state to report at all, and if a Conditional Access policy is written loosely enough — say, requiring MFA but not explicitly requiring a compliant or hybrid-joined device — an unmanaged device can satisfy the policy just as easily as a fully managed one. Getting device compliance to function as a real signal, not a theoretical one, means being deliberate about which apps actually require it as a grant control, and auditing regularly for policies that quietly let unmanaged devices through the same door as managed ones.
