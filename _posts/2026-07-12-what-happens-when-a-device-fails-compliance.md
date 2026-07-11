---
layout: post
title: "What Happens When a Device Fails Compliance"
date: 2026-07-12
categories: ["Zero Trust", "Device & Endpoint Trust"]
tags: ["device-compliance", "intune", "conditional-access"]
author: pankaj
---
Most explanations of device compliance stop at "the policy checks the device and marks it compliant or not," which leaves out the part practitioners actually need to know: what happens in the minutes and hours after a device slips out of compliance, and how much of that sequence is automatic versus something someone has to notice and act on.

## The actual sequence

When a managed device stops meeting a compliance policy — say a user disables BitLocker, or an OS update lapses past the grace period — the change isn't detected instantly. Intune relies on the device checking in on its configured evaluation schedule, and depending on how that's set, the gap between the device actually falling out of compliance and Intune marking it as such can range from under an hour to most of a day. During that window, the device still presents as compliant to Entra ID and any Conditional Access policy relying on that state, which is the exact stale-signal problem worth designing around rather than assuming away.

Once Intune does report the new state, Entra ID updates the device's compliance claim, and the next time that device tries to authenticate against an app protected by a policy requiring compliant devices, the sign-in gets blocked at that point, not retroactively for any session already established. This is an important distinction I tested directly in a lab tenant: a user already signed into an app in an active session isn't immediately kicked out the moment their device becomes non-compliant, because the existing token was issued while the device still satisfied the policy. The block applies to the next fresh authentication attempt, which is one of the reasons Continuous Access Evaluation exists — to shrink that gap for supported scenarios rather than waiting for natural token expiry.

## What the user actually sees

From the end user's side, a blocked sign-in due to non-compliance typically surfaces as an access-denied message that, if the tenant has bothered to configure it, points to a support URL or explains which compliance requirement failed. A lot of tenants skip this configuration step and leave users with a generic "your organization requires this device to be compliant" message with no actionable next step, which turns every one of these events into a help desk ticket instead of a self-service fix. Configuring the compliance policy's custom URL and instructions in Intune, so the message actually tells the user what to remediate, is a small setup step that pays for itself the first time a whole group of devices drifts out of compliance because of a common misconfiguration like an expired certificate.

## Remediation and re-entry

Getting back to compliant is just a matter of the device meeting the failed condition again and reporting that back to Intune on its next check-in, at which point Entra ID's compliance claim updates and access resumes without any manual intervention required on the Entra ID side. Where this breaks down in practice is when the underlying cause isn't something the user can fix themselves — a certificate that needs reissuing, an update that's failing silently, a management agent that's stopped checking in entirely. In those cases the device can sit in a non-compliant state indefinitely, generating repeated access denials, until someone on the endpoint management side actually investigates why the check-in itself has gone stale, since a device that has stopped reporting to Intune at all looks identical, from the compliance policy's perspective, to one that is actively failing its checks.
