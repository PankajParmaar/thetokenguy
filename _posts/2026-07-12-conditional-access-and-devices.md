---
layout: post
title: "Conditional Access and Devices"
date: 2026-07-12
categories: ["Microsoft 365", "Endpoint Management"]
tags: ["conditionalaccess", "intune", "devices", "entraid"]
author: pankaj
---
Device state is one of the strongest signals available to a Conditional Access policy, but it's also one of the most misunderstood, because there are three related-but-distinct device conditions available, and picking the wrong one produces a policy that either blocks people it shouldn't or, worse, lets in devices that were never actually meant to pass.

## Compliant, hybrid joined, or filtered — these are different checks

"Require device to be marked as compliant" checks the Intune compliance state discussed elsewhere — it needs a device enrolled in Intune with a compliance policy assigned and evaluated. "Require hybrid Azure AD joined device" checks a different thing entirely: whether the device is joined to an on-premises Active Directory domain and also registered with Entra ID through Azure AD Connect, which is a common state for traditional domain-joined corporate Windows machines that were never separately Intune-enrolled. A device can be hybrid joined and not compliant, or compliant and not hybrid joined, and a Conditional Access policy that assumes these are interchangeable ends up blocking a legitimate device population that satisfies one condition but not the other.

Device filters are the third, more surgical tool — rather than a binary compliant/non-compliant check, a filter lets a Conditional Access policy target or exclude devices based on specific Entra ID device attributes: manufacturer, model, OS version, or a custom extension attribute. This is the right tool when the requirement isn't "any compliant device" but something narrower, like "block access from any device that isn't one of our approved corporate models," which compliance state alone can't express.

## Grant controls versus session controls, applied to devices

Once a Conditional Access policy identifies a device condition, the actual enforcement splits into grant controls (require compliant device, require MFA, require approved client app — evaluated before access is granted at all) and session controls (app-enforced restrictions, Conditional Access App Control routing through Microsoft Defender for Cloud Apps for real-time session monitoring). A common pattern for unmanaged devices — someone signing in from a personal laptop that's never been enrolled — is to grant access but route the session through app-enforced restrictions that block downloads and printing, rather than blocking access outright. That's a meaningfully different design decision than a hard block, and it needs to be made deliberately rather than defaulting to whichever control happened to be checked first when the policy was built.

## The gap between "no device signal" and "non-compliant device"

A device with no Intune enrollment at all and a device that's enrolled but currently failing a compliance check are both going to fail a "require compliant device" grant control, but they represent very different situations, and a Conditional Access policy alone doesn't distinguish them in its plain-language name. This matters for troubleshooting: when a user reports being blocked, checking the sign-in log's device details tells you whether they're hitting the policy because the device was never enrolled, or because it's enrolled and actively failing a specific compliance rule — and those two paths to resolution are completely different, one requiring enrollment onboarding, the other requiring the user to fix a specific setting like enabling encryption.

## Report-only mode before enforcement, every time

Every Conditional Access policy involving device state deserves a stint in report-only mode before going live, the same discipline as transport rules and DLP policies elsewhere in the stack. Report-only mode evaluates the policy against real sign-ins and logs what would have happened without actually blocking anything, and reviewing a week or two of that data against the sign-in logs is what catches the edge case — a whole team using an old hybrid-joined device population that was never actually Intune-enrolled, say — before it turns into a mass lockout the moment the policy flips to enforced.

Device-aware Conditional Access is genuinely one of the strongest controls available in the whole Microsoft 365 stack, but its strength depends entirely on correctly matching the specific device condition to what you actually mean, rather than reaching for "require compliant device" as a generic device check and assuming it covers every scenario a device-based control might need to handle.
