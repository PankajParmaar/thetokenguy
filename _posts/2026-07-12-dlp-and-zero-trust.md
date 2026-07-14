---
layout: post
title: "DLP and Zero Trust"
date: 2026-07-12
categories: ["Zero Trust", "Data Protection"]
tags: ["dlp", "zero-trust", "data-protection"]
author: pankaj
description: "DLP gets bolted onto Zero Trust conversations as an afterthought, which is backwards. It answers what other pillars can't: data access after the fact."
image:
  path: /assets/img/og-default.png
  alt: "DLP and Zero Trust"
---
Data Loss Prevention tends to get bolted onto Zero Trust conversations as an afterthought, mentioned briefly at the end after identity, device, and network controls have had their turn. That ordering is backwards in an important sense, because DLP is the pillar that answers a question none of the other three can: even if the right person, on the right device, over the right network path, is accessing a resource they're fully authorized to touch, should this specific piece of data actually be allowed to leave in the way they're trying to move it.

## Why identity and device controls can't cover this

A user with a correctly authenticated, MFA-satisfied, fully compliant-device session accessing a document they're genuinely authorized to view is, by every identity and device signal available, a fully trusted request. None of that trust says anything about whether that same user should be allowed to paste the contents of that document into a personal email, or upload it to a consumer cloud storage account, or print it. Identity and device controls answer "should this session be allowed to happen." DLP answers "should this specific data movement, inside an otherwise legitimate session, be allowed to complete." Zero Trust that stops at identity and device coverage has built strong walls around a room and left the door on the far side of that room wide open.

## Where DLP actually plugs into the architecture

In a lab tenant, I set up a DLP policy targeting a defined sensitive information type across Exchange, SharePoint, and endpoint DLP, and tested it against a few different exfiltration paths — email to an external address, upload to an unsanctioned cloud storage site, and copy to a USB device from a managed endpoint. Each path is actually a separate enforcement point in Purview's DLP configuration, and getting comprehensive coverage means explicitly enabling and tuning policy across all of them rather than assuming a single policy blankets every channel automatically. Endpoint DLP in particular requires the Purview endpoint agent to be deployed to devices, which is easy to forget when a DLP rollout focuses mostly on the Exchange and SharePoint locations that are simpler to configure first.

The other integration point worth calling out is how DLP consumes the sensitivity labels and classification discussed elsewhere in this series. A DLP policy is only as precise as the classification feeding it — a rule built purely on keyword matching catches far less, and generates more false positives, than one built on a properly trained classifier or a well-scoped sensitive information type. DLP and information protection aren't really separate programs; DLP is the enforcement mechanism that classification and labeling exist to feed.

## Where it breaks in real deployments

The gap I've seen come up most often, both in lab testing and in incident write-ups from other practitioners, is DLP policies configured for detection and alerting but never moved to actual blocking, for the same reason enforcement stalls in information protection generally — the last attempt to flip a policy to block mode caught a sales director mid-quarter mid-deal, blocking an attachment to a client, and the resulting escalation got the policy reverted to audit-only within the day and never revisited. A DLP policy sitting in audit mode generates a useful stream of visibility into what's actually leaving the organization, but it stops nothing, and organizations that treat the audit log as the finished deliverable have built a monitoring system, not a prevention system.

The other recurring gap is scope creep in the opposite direction — DLP policies so broad and so strictly enforced that they block legitimate business workflows constantly, training users to find workarounds like screenshotting restricted content or retyping it manually into an unmonitored channel. DLP inside a Zero Trust architecture has to be tuned deliberately, closing genuine exfiltration paths without generating so much friction that people route aro