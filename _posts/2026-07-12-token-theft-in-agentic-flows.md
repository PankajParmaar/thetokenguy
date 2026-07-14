---
layout: post
title: "Token Theft in Agentic Flows"
date: 2026-07-12
categories: ["AI & Identity", "Prompt & Token Risk"]
tags: ["token-theft", "agentic-ai", "oauth", "session-security"]
author: pankaj
description: "Token theft isn't new — infostealers and AiTM kits have harvested session tokens for years. What's changed is the blast radius one stolen token now reaches."
image:
  path: /assets/img/og-default.png
  alt: "Token Theft in Agentic Flows"
---
Token theft isn't a new problem - infostealer malware has been harvesting browser session tokens for years, and adversary-in-the-middle phishing kits that steal tokens post-authentication are well documented at this point. What's changed is the shape of the target. In a normal browser session, a stolen token gets you into one application, scoped to whatever that session was doing. In an agentic flow, a single stolen token can sit at the center of a chain of automated actions across multiple systems, and the agent's own architecture often makes that token easier to get at than a browser's would be.

## Why agent architectures widen the target

An agent orchestrating a multi-step task typically holds a token (or several) in memory for the duration of that task, sometimes passing it to sub-agents or tools, sometimes writing it to a local cache or logging it as part of debugging output. Every one of those hops is a place the token can leak. I set up a small test harness once where an orchestration framework logged full HTTP request headers at debug level for troubleshooting - including the bearer token - and that log file sat in a location with broader read access than the token itself would have justified. That's not a sophisticated attack; that's a logging default doing the attacker's work for them. Agentic frameworks, especially newer or hastily built ones, don't always have the same hardened handling around credential material that mature identity libraries have built up over years of getting burned.

## Refresh tokens and long-lived sessions

The bigger structural risk is that agentic flows often lean on longer-lived refresh tokens to avoid re-authenticating a user constantly during a multi-step task, and a stolen refresh token is a much better prize than a stolen access token, because it can be used to mint fresh access tokens indefinitely until it's explicitly revoked or expires. If an agent's refresh token gets exfiltrated - through a compromised host, a leaked log, or a supply-chain issue in a dependency the agent framework pulled in - the attacker doesn't just get one session's worth of access, they get a renewable credential that can keep working long after the original task completed and the human involved has moved on and forgotten about it.

## What actually limits the damage

Token binding techniques - sender-constrained tokens using DPoP or mutual TLS, where the token is cryptographically tied to the specific client that requested it - are one of the more effective mitigations here, because a stolen token without the corresponding private key becomes useless to a different caller. In practice I've seen very few agentic frameworks actually implementing DPoP yet, mostly because it's a genuine engineering lift and the ecosystem is still catching up to needing it. Short token lifetimes help too, even though they add operational friction, because they shrink the window in which a stolen access token is worth anything, forcing an attacker to also have the refresh token to sustain access, which is a meaningfully higher bar.

Continuous access evaluation and anomaly-based session revocation are the other lever worth building toward - if a stolen token starts getting used from an unfamiliar IP, at an unusual hour, or against a resource the original session never touched, that pattern should be enough to kill the session rather than waiting for a scheduled token expiry to eventually catch up. None of this is exotic thinking in identity circles, but agentic flows are exposing how much of it was built with a human clicking through a browser in mind, and how much rebuilding is needed before the same protections hold up against a machine passing tokens around a multi-step chain at machine speed.
