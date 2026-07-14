---
layout: post
title: "What the exp Claim Doesn't Protect"
date: 2026-07-12
categories: ["AI & Identity", "Prompt & Token Risk"]
tags: ["jwt", "token-expiry", "identity-security", "agentic-ai"]
author: pankaj
description: "A JWT's exp claim, fifty-nine minutes out, gives the reassurance it's designed to — accurate as far as it goes, covering a narrower window than assumed."
image:
  path: /assets/img/og-default.png
  alt: "What the exp Claim Doesn't Protect"
---
I decoded a JWT from a test agent flow in my lab a while back mostly out of curiosity, and the `exp` claim sitting there - a Unix timestamp fifty-nine minutes in the future - gave me the same reassurance it's designed to give anyone who checks it. The token expires. Within the hour, whatever this credential can do stops working. That reassurance is accurate as far as it goes, but it covers a much narrower slice of risk than the presence of the claim tends to imply, especially once you're reasoning about agentic flows rather than a single human browser session.

## What exp actually guarantees

The `exp` claim guarantees exactly one thing: a resource server checking the token correctly will reject it after that timestamp. That's a real and useful property - it bounds how long a leaked access token stays useful if nobody notices the leak and revokes it manually. But it says nothing about what happens inside that window, and for an agentic flow, a lot can happen inside fifty-nine minutes. An agent with a valid, unexpired token can chain together dozens of automated actions in that window - reading files, sending messages, kicking off downstream workflows - and the `exp` claim does nothing to constrain what happens during the time the token is valid, only when it stops being valid at all.

## The refresh token undermines the whole premise

The bigger issue is that `exp` on the access token often means very little in practice because the access token gets silently renewed via a refresh token before it ever expires. I traced a token lifecycle in a test OAuth flow and watched the client refresh the access token automatically at the fifty-minute mark, well before the fifty-nine-minute expiry, meaning the actual session persisted far longer than the short-lived access token's `exp` claim would suggest to anyone reading it in isolation. If that refresh token itself has a long lifetime - or worse, no expiry configured at all, which I've seen in test app registrations left at default settings - the short-lived access token is mostly theater from a risk standpoint, giving the appearance of a tightly bounded credential while the actual session backing it can run for weeks.

## What exp doesn't say about scope or intent

`exp` also says nothing about whether the token is still being used for the purpose it was originally issued for. A token minted for an agent to summarize a document remains just as valid, from the resource server's perspective, if that agent gets manipulated mid-task into attempting something entirely different - the timestamp doesn't know the difference between the intended use and a hijacked one. This is where the earlier point about prompt injection connects directly to token design: expiry protects against a credential outliving its usefulness, not against a credential being misused for something other than its original intent while it's still valid.

## What actually needs to sit alongside exp

The claims and mechanisms that address the gaps `exp` leaves open are things like short refresh token lifetimes paired with continuous access evaluation, sender-constrained tokens so a leaked bearer token can't be replayed from a different client, and scope claims that are actually enforced narrowly at the resource server rather than treated as a formality. None of these replace `exp` - they sit alongside it, because expiry is a necessary but genuinely insufficient control on its own.

The habit worth building is reading a token's `exp` claim as one line in a longer story rather than the whole plot - it tells you when this specific credential stops being valid, and says nothing about how long the session behind it actually persists, what the token can be used for in the meantime, or whether it's still being used the way it was originally intended, which in an agentic flow moving through several automated steps a minute is exactly the part that needs watching.
