---
layout: post
title: "Why NHI Is the Fastest Growing Attack Surface"
date: 2026-07-12
categories: ["AI & Identity", "Non-Human Governance"]
tags: ["non-human-identity", "attack-surface", "service-accounts", "governance"]
author: pankaj
description: "A rough count in one lab tenant put workload accounts at roughly six times the human user count — and that ratio is on the low end industry-wide still."
image:
  path: /assets/img/og-default.png
  alt: "Why NHI Is the Fastest Growing Attack Surface"
---
I did a rough count once in a lab tenant I'd been building up over a year of testing - service principals, managed identities, API keys, workload accounts, CI/CD credentials - and the number came out to roughly six times the number of actual human users I'd created. That ratio is not unusual; it's actually on the low end of what I've seen referenced in industry reporting on production tenants. Every human identity in most organizations comes with an onboarding process, an offboarding process, and at least some periodic access review, however imperfect. Non-human identities came into existence as a side effect of someone building something, and in a lot of cases the ticket that created them never named an owner, so there's no name to attach to a deprovisioning request when the project ends.

## Where they come from

Non-human identities accumulate from ordinary engineering work, not from some separate provisioning process anyone consciously designed. A developer creates a service principal to let a pipeline deploy to a resource group. Someone spins up a managed identity for a function app. A vendor integration needs an API key stored somewhere. None of these get created through the same request-approve-review flow a human account goes through, because in most organizations there isn't an equivalent flow for machine credentials - the closest thing is usually a Jira ticket and whoever had contributor access that day. That's how you end up with credentials that outlive the project they were created for, the person who created them, and sometimes the resource they were originally scoped to access.

## Why they're a better target than human accounts

A non-human identity is frequently more valuable to compromise than a human one because it usually carries broader, more consistent access and almost never has MFA sitting in front of it - you can't push an authentication prompt to a service principal. It also tends to have a long-lived secret rather than a session that expires, and that secret is often stored in places that get checked less carefully than a password vault: environment variables, CI configuration, a config file that made it into a repository at some point. When I've walked through breach writeups over the past couple of years, a recurring theme is a leaked service account key sitting in a public repo or a misconfigured storage bucket, still carrying the broad permissions it was granted at creation because no one ever filed the follow-up ticket to scope it down.

## The governance gap that makes it worse

The reason this surface keeps growing faster than the tooling to manage it is that most identity governance programs were built around the assumption that identities map to people, and the tooling reflects that assumption everywhere - access reviews ask "does this person still need this role," not "does this pipeline still exist." Ownership is the piece that breaks down hardest: a service principal created by an engineer who left the company two years ago has no obvious accountable owner, so it doesn't show up in anyone's review queue, and it doesn't get flagged as stale because nothing in the review process is looking for it. AI agents are now adding a new layer on top of an already unmanaged one, spinning up their own non-human credentials to call other services, often without the same registration discipline (however weak) that CI/CD pipelines eventually earned after enough incidents.

Closing this gap requires treating non-human identity as its own governance category with its own inventory, ownership model, and rotation cadence, rather than bolting machine accounts onto a human-centric IAM program and hoping the existing controls stretch t