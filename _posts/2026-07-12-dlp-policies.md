---
layout: post
title: "DLP Policies"
date: 2026-07-12
categories: ["Microsoft 365", "Compliance & Data Protection"]
tags: ["dlp", "purview", "compliance", "datalossprevention"]
author: pankaj
---
Data loss prevention policies in Microsoft Purview cover a wider surface than most people expect the first time they set one up — Exchange, SharePoint, OneDrive, Teams chat, and endpoint devices through the Purview endpoint DLP agent, all evaluated against the same policy definitions. That breadth is the point, but it also means a DLP policy designed with only email in mind can produce unexpected behavior the moment it starts evaluating Teams chat messages or files being copied to a USB drive on a managed endpoint.

## What a policy is actually built from

A DLP policy is a location scope (which of those workloads it applies to), a set of conditions built around sensitive information types (credit card numbers, national ID formats, or custom regex-based types you define), and a rule that ties a confidence threshold and instance count to an action. The instance-count-and-confidence pairing is where a lot of the tuning work actually happens — a rule that fires on "one instance of a credit card number, low confidence" is going to generate a flood of false positives from things like support ticket numbers that happen to match the pattern, while a rule requiring "ten or more instances, high confidence" is a much better signal for genuinely bulk, exposed data.

## The three enforcement tiers

Policies in test mode with policy tips generate alerts and show the end user a warning banner, but don't block anything, which is the right starting posture for any new policy, the same way you'd stage a transport rule. Test mode without policy tips runs silently, useful for baselining what a policy would catch before users even know it exists, which is worth doing before deciding whether a proposed rule is even close to tuned correctly. Full enforcement blocks the action outright — preventing the email from sending, preventing the file from being shared externally, or preventing the copy to removable media — and moving a policy to full enforcement before you've reviewed at least a couple of weeks of test-mode alerts is how you end up with a business-critical process silently broken because a legitimate export tripped a rule nobody tuned for that pattern.

## Where DLP overlaps with other controls

DLP sensitive information type matching is not the same thing as a sensitivity label, and the two get conflated constantly. A DLP policy can use a sensitivity label as one of its conditions (block sharing of anything labeled "Confidential"), but the label itself doesn't do the DLP scanning — it's the policy watching for that label, or watching independently for pattern-matched content, that actually enforces anything. Similarly, DLP and Exchange transport rules can both act on outbound mail, and running both without coordinating them means troubleshooting "why was this email blocked" requires checking two separate policy engines instead of one.

## Endpoint DLP is worth calling out separately

Endpoint DLP, which requires devices onboarded to Microsoft Defender for Endpoint (or at minimum the Purview endpoint DLP agent), extends the same policies down to actions on the device itself: copying a sensitive file to USB, uploading it to a non-sanctioned cloud service through a browser, or printing it. This is the layer that catches the exfiltration path that pure cloud-side DLP simply can't see, since a file copied straight from a local folder to a USB drive never touches Exchange, SharePoint, or Teams at all. Getting endpoint DLP running requires the device onboarding step to actually be in place first, which is a common gap — a tenant with well-tuned cloud DLP policies but no endpoint onboarding has a real blind spot they may not realize exists until an incident makes it obvious.

The recurring lesson with DLP across every rollout I've studied is the same one as most Microsoft 365 governance: start narrow, watch real traffic in test mode, and expand enforcement only once the false-positive rate is low enough that people trust the block message when they see it, rather than immediately requesting an exception because the policy has already cried wolf.
