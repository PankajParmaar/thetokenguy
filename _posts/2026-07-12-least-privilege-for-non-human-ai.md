---
layout: post
title: "Least Privilege for Non-Human AI"
date: 2026-07-12
categories: ["AI & Identity", "AI Agents & Access"]
tags: ["least-privilege", "ai-agents", "non-human-identity", "access-control"]
author: pankaj
description: "Least privilege gets abandoned the moment a deadline shows up. With human users that habit is bad but bounded. With AI agents, it isn't bounded at all."
image:
  path: /assets/img/og-default.png
  alt: "Least Privilege for Non-Human AI"
---
Least privilege is one of those principles that shows up in every access control policy document and gets abandoned the moment a deadline shows up. I've done it myself in lab setups - given an agent owner-level access to a resource group because scoping down a custom role would have taken another hour, telling myself I'd tighten it later. With human users that habit is bad but bounded, because a person's access maps roughly to their job function and someone eventually notices if it's wildly wrong. With AI agents the same habit compounds differently, because an agent doesn't have a job function in the traditional sense - it has a task list that changes, and the permission set that felt reasonable for task one quietly becomes way too broad for task fifty.

## Why the usual least-privilege playbook doesn't map cleanly

Least privilege for a human is usually implemented through role-based access control tied to a job title, reviewed on some cadence, revoked on offboarding. None of those anchors exist cleanly for an agent. There's no HR system triggering deprovisioning, no manager attesting the access is still needed, and often no clear owner accountable for reviewing what the agent can touch six months after someone stood it up. In my own lab experiments, an agent identity I created to test a document-summarization workflow was still sitting in the tenant with Reader access three months later, because nothing prompted me to go back and check. Multiply that by however many agents a real organization spins up for pilots and proofs of concept, and you get a long tail of standing access with no owner listed in any access review, sitting there until a security assessment stumbles onto it.

## Scoping by task, not by role

The more workable model is scoping access to the specific task the agent is executing right now, using time-boxed or just-in-time credentials rather than standing permissions. If an agent needs to read a specific SharePoint site for a document review, it should get a credential scoped to that site for the duration of the job, not a service principal with Sites.Read.All sitting around indefinitely. This is more setup work up front - it means the orchestration layer has to actually track what resource each task touches and request access accordingly - but it collapses the standing attack surface down to whatever's active at any given moment, rather than the sum of everything an agent has ever been asked to do.

## The audit gap

The part that gets missed most often is that least privilege isn't just about the initial grant, it's about the review cycle catching drift. Agent permissions tend to get added incrementally - someone hits a permission error, adds the missing scope, and moves on - and almost never get subtracted, because removing access from a working agent feels riskier than leaving it alone. I've caught this pattern in my own test environments: an agent accumulating Mail.Send, Files.ReadWrite, and Calendars.ReadWrite over a few weeks of iterative development, only two of which were ever actually exercised by real workflow paths.

Getting least privilege right for non-human AI identities means treating access as something that expires by default rather than something that accumulates by default, and building the review habit into the agent's operational lifecycle instead of leaving it on a someday list, because in my own lab environment the only