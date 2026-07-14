---
layout: post
title: "Remediation Prioritisation"
date: 2026-07-12
categories: ["Microsoft Defender", "Vulnerability Management"]
tags: ["remediation", "vulnerability management", "patch prioritization"]
author: pankaj
description: "Every vulnerability scan outpaces any patching team's capacity to clear it. CVSS alone was never a good enough filter for what actually matters first."
image:
  path: /assets/img/og-default.png
  alt: "Remediation Prioritisation"
---
Every vulnerability scan produces a longer list than any patching team can realistically clear before the next scan runs, which means prioritization isn't an optional refinement step, it's the actual job. CVSS score alone has never been a good enough filter for that, and Defender Vulnerability Management's approach is built around that exact problem.

## CVSS is a starting point, not a ranking

A CVE with a CVSS base score of 9.8 sitting on an isolated dev VM with no internet access is a lower real-world priority than a CVSS 7.5 sitting on an internet-facing server with a public exploit already circulating. Defender's remediation recommendations factor in more than the base severity score: whether the vulnerability has a known exploit available, whether it's flagged in active threat campaigns Microsoft is tracking, whether the affected software is internet-facing, and how many devices in the org are actually affected. That combination is what shows up as the "breach likelihood" and threat context tagged onto each recommendation, and it's the part worth reading before accepting the default ranking at face value.

## Security recommendations as the actual work queue

Rather than working straight off a raw CVE list, the practical workflow runs through the security recommendations view, which groups findings by remediation action rather than by individual vulnerability. A single outdated version of a common library might map to dozens of CVEs, but the actual remediation is one update action, and the recommendations view collapses that down so the patching team isn't working a list that's artificially inflated by CVE count rather than by distinct actions required.

Each recommendation can be converted into a remediation request that integrates with Intune, creating a task with an assigned owner and due date, which closes a gap that used to exist between the security team identifying something and the endpoint team actually being accountable for fixing it on a timeline.

## Exceptions are a feature, not a workaround

Not every recommendation is going to get remediated on the timeline the score suggests, and Defender Vulnerability Management supports formal exceptions with a business justification and expiration date rather than just letting a recommendation quietly age out of anyone's attention. Using that mechanism properly, with a real expiration date and a documented reason like "vendor hasn't released a compatible patch yet," keeps the exception visible and re-reviewable instead of turning into a permanently accepted risk that surfaces two years later in an audit with no record of who approved it or why.

## Where prioritization goes wrong

The most common failure is prioritizing purely on CVSS score without checking exploitability and exposure context, which burns patching cycles on theoretically severe but practically low-risk findings while genuinely exploited vulnerabilities sit further down the queue. The second common failure is treating prioritization as a one-time ranking exercise rather than a continuously shifting list, since exploit availability and threat intelligence context change after the initial scan, and a recommendation that was low priority last month can jump the queue once active exploitation gets confirmed.

Getting remediation prioritization right isn't about finding a perfect scoring formula, it's about making sure the context that actually predicts real-world exploitation, not just theoretical sev