---
layout: post
title: "External Access vs Guest Access"
date: 2026-07-12
categories: ["Microsoft 365", "Teams & Collaboration"]
tags: ["teams", "guestaccess", "externalaccess", "b2b"]
author: pankaj
---
These two settings sit right next to each other in the Teams admin center and get confused constantly, partly because Microsoft's own naming hasn't always made the distinction obvious, and partly because they solve overlapping-sounding problems with completely different security models underneath.

## External access is federation, not identity

External access, sometimes still called federation in older documentation, is about letting users from your tenant chat, call, and see presence for users in another organization's tenant — and vice versa — without either party joining the other's directory. When you enable external access for a specific domain, someone at that partner org can find you by their own email address, message you directly, and add you to a call, but they never show up in your Entra ID directory, never get assigned a license from you, and can't be added to a Team or channel as a member. It's essentially cross-tenant presence and chat, scoped at the domain level, and it's the right fit for ongoing informal contact with a partner organization — a consultancy you talk to regularly, a vendor's account team.

The domain-level allow/block list is the main lever here, and the default in most tenants is "allow all external domains except blocked ones," which is more permissive than people expect the first time they check. If you want a tighter posture, flipping it to an explicit allow-list of named partner domains is worth doing early, because switching from open to allow-list after external chats are already established means existing conversations with un-listed domains just stop working, and that support ticket is a confusing one to receive.

## Guest access is identity, with real object presence

Guest access, by contrast, actually brings the external person into your directory as a guest user object in Entra ID, provisioned through Azure AD B2B. A guest user can be added as a genuine member of a specific Team, see that Team's channels, files, and chat history, and participate exactly like an internal member within the scope of what they've been invited into. This is the right tool when you need someone from outside the org working inside a specific project team, seeing SharePoint documents tied to that Team, not just chatting with one person.

Because guest users are real objects in your directory, they're subject to Conditional Access policies, can be governed through access reviews, and show up in audit logs the same way internal accounts do — which is a meaningfully stronger security posture than external access, but it also means guest access carries actual governance overhead that federation doesn't. An org that turns on guest access tenant-wide without a plan for reviewing and removing stale guest accounts ends up, a year later, with guest users still sitting in Teams from projects that wrapped up months earlier, still able to see files they no longer have any reason to access.

## Choosing between them

The distinction that actually matters day to day: if you need presence, chat, and calls with someone at another company on an ongoing basis, that's external access. If you need someone from outside actively collaborating inside a specific team's files and channels, that's guest access, and it needs an offboarding plan from the moment you turn it on — ideally an access review tied to the project's expected end date, not a manual cleanup someone remembers to do eventually. Both settings can be on simultaneously and usually are, but conflating them when troubleshooting ("why can't this external person see the files") wastes time that a five-second check of which mechanism actually applies would save.
