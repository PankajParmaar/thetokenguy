---
layout: post
title: "What Copilot Can See"
date: 2026-07-12
categories: ["AI & Identity", "Copilot Security"]
tags: ["microsoft-copilot", "data-access", "permissions", "m365"]
author: pankaj
---
I set up a test Microsoft 365 tenant a while back specifically to poke at Copilot's data access model, because the marketing language around it - "Copilot respects your existing permissions" - is true in a narrow, technically accurate way that hides a much messier practical reality. Copilot doesn't get its own separate grant of access to your tenant's data; it runs queries against Microsoft Graph using the identity of whoever is asking it a question, which means in principle it can only surface what that user could already open manually. The catch is that "could already open manually" and "would ever actually find" are two very different things, and Copilot closes that gap in a way that changes the practical risk profile even when the permission model hasn't changed at all.

## The permission model is real, but narrow

When a user asks Copilot a question, the query runs under that user's own Graph permissions, respecting whatever access controls already exist on SharePoint sites, OneDrive files, Teams messages, and Exchange mailboxes. If a document has restrictive sharing and that user isn't on the list, Copilot won't surface it, at least in the reference implementations Microsoft has documented. That part checks out in my own testing - I locked a folder down to a specific security group and confirmed a user outside that group couldn't get Copilot to surface content from it, no matter how I phrased the prompt.

## Where the practical risk actually lives

The problem is that most organizations have permission sprawl that predates Copilot by years, and Copilot is extremely good at finding it. A SharePoint site with "everyone in the organization" sharing enabled because someone needed a quick way to distribute a file three years ago is exactly the kind of over-shared content Copilot can now surface instantly through natural language search, where before it required someone to know the file existed and go looking for it deliberately. In my test tenant, I deliberately left a handful of documents with organization-wide sharing links (mimicking the kind of legacy sprawl I've read about in real deployments) and Copilot pulled content from them within a single query, correctly reasoning that since the current user technically had access, it was fair game to summarize. Technically correct, practically alarming, because that sharing link was set up for a project that wrapped up two years earlier, and it hadn't shown up on any access review since - Copilot was the first thing to actually use it.

## Oversharing becomes visible, fast

This is really an oversharing discovery problem wearing an AI security label. Copilot isn't creating new access paths - it's making existing, forgotten ones dramatically more discoverable, and it's doing it at a speed and breadth no human searching manually ever could. A user who'd have needed to guess a file name or browse a folder structure manually can now just ask "what's our current pricing strategy" and get a synthesized answer pulling from whatever oversharing exists across the tenant, phrased so naturally that the user may not even realize they're seeing content from a source they'd normally have never stumbled onto.

The practical lesson from my own poking around is that "Copilot respects permissions" is true and also not the reassurance it sounds like, because the permission model in most tenants was built assuming discovery required a person to guess a filename or browse a folder tree - a tool that answers plain-language questions across the whole tenant was never part of that threat model - which means the actual pre-requisite for deploying Copilot