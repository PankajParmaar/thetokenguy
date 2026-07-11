---
layout: post
title: "Permission Boundaries"
date: 2026-07-12
categories: ["AI & Identity", "Copilot Security"]
tags: ["permission-boundaries", "microsoft-copilot", "access-control", "m365"]
author: pankaj
---
Somewhere in the middle of testing Copilot's behavior against a deliberately messy test tenant, I realized the phrase "permission boundary" means something slightly different once an AI assistant is the one exercising the permission, and that difference is worth being precise about instead of glossing over with the usual "least privilege" language.

## Static boundaries vs. dynamic exercise

A permission boundary in the classical sense is a static thing - a role definition, an access control list, a conditional access policy that says this user can reach this resource under these conditions. That boundary doesn't change whether the user opens a file manually or asks an AI to summarize it, because the boundary is enforced at the resource layer, not at the point of intent. What changes is how thoroughly and how quickly the boundary gets exercised. A human browsing SharePoint manually samples a small fraction of what they technically have access to, because browsing is slow and most people only look where they already expect to find something. Copilot, when asked a broad question, effectively queries the entire boundary at once, pulling from every resource the user's permissions touch that's relevant to the prompt. The boundary hasn't moved. The rate and breadth of access against that boundary has changed enormously.

## Where boundaries turn out to be softer than assumed

In my test tenant, I found permission boundaries that looked fine on paper but weren't actually enforcing what I assumed. A Teams channel with a "private" label still had an underlying SharePoint site with broader sharing settings that predated the channel's creation, and Copilot pulled from the SharePoint layer rather than respecting the mental model of "private channel means private." Nothing here was a bug - it was a mismatch between how a boundary is labeled at the collaboration layer and how it's actually enforced at the storage layer, and that mismatch is exactly the kind of thing that stays invisible until something starts querying every layer simultaneously and reporting back conversationally, instead of a human clicking through folders and eventually giving up.

## Boundaries that Copilot itself introduces

Beyond the existing tenant permissions, Copilot and similar assistants add their own boundary layer through configuration - things like sensitivity labels, DLP policies scoped specifically to Copilot, and admin controls that can exclude certain SharePoint sites or content types from being indexed for retrieval at all. These are genuinely useful controls, but they're opt-in and require someone to have configured them deliberately, which in a tenant that stood up Copilot quickly to meet a rollout deadline often hasn't happened. I tested this directly: without an explicit sensitivity label restricting AI access, a document I marked confidential through normal classification was still fully retrievable by Copilot for any user with underlying file access, because the classification label alone doesn't automatically translate into an AI-specific restriction unless the corresponding policy is switched on.

## The boundary that actually matters

The practical shift this forces is thinking about permission boundaries as something that needs to be actively tested against an AI retrieval pattern, not just documented on paper as a role or ACL and assumed correct. Running a set of broad, exploratory prompts against a new Copilot deployment before general rollout - the equivalent of a penetration test aimed specifically at boundary assumptions - surfaces exactly the kind of soft boundary I found in my own lab tenant, the ones that were technically compliant with a policy document but never actually tested against something that queries everything at once and answers in plain language regardless of which layer the data came from.
