---
layout: post
title: "Iron Bar — The Access Token"
hero_title: "Iron Bar &mdash; The <em>Access Token</em>"
date: 2026-07-12 00:01:00 +0530
categories: [homepage]
tags: [identity, access-tokens, history, jwt, zero-trust]
author: pankaj
description: "Sometime around the ninth century, if you couldn't prove who you were — a church official would hand you a bar of iron that had been sitting in a fire. This is where identity begins."
toc_items:
  - Trial by Ordeal
  - The Doppelganger problem
  - Yellow Pages → Active Directory
  - Scale and consequence
  - The judgment we automated away
---

Sometime in the ninth century, if you wanted to prove you were who you said you were, they handed you a bar of iron pulled from a fire.
You'd carry it nine steps. Then they'd bandage your hand and check back in three days. Healed cleanly? God had verified your identity. Still a mess? Also God, but with a different verdict.
This was called Trial by Ordeal. It was the official identity and access management system of medieval Europe for about four hundred years. And here's the part that should make every security professional quietly put down their coffee — it mostly worked. Not because burning iron is a reliable authentication mechanism, but because the priests who administered it would subtly cool the bar for people they already trusted. The system had a backdoor. The backdoor was called judgment. And the people with admin access to the backdoor were the same people who ran the system.
Some things don't change as much as we'd like.
A few centuries later, in a different part of Europe, a different problem. Two people, same name, same town. One of them has debts. One of them doesn't. Neither can definitively prove which is which to a magistrate who's never met either of them. The one who argues better, or knows the right people, walks away with the other person's life — or leaves them with their own accumulated disasters. No MFA. No audit log. No helpdesk to escalate to. Just two humans in a room and someone making a judgment call that would follow the loser for the rest of their life.
Ask someone in IT where identity begins and some of the older ones will say the Yellow Pages — and they're not entirely wrong. X.500, the standard that eventually became the conceptual backbone of Active Directory, was explicitly modelled on directory services. A lookup table. A name, some attributes, a way to find things. The Yellow Pages had entries. X.500 had entries. Active Directory has entries. Entra ID has entries. The technology reinvents itself every decade or so and the underlying idea sits there unchanged, patient as a monk with a hot iron bar, waiting for someone to rediscover it and call it innovation.
What actually changed — what always changes — is scale and consequence.
A medieval village had a few hundred people. Everyone knew everyone. Identity was mostly solved by proximity and memory, with the occasional catastrophic failure mode involving fire. A modern enterprise tenant has hundreds of thousands of objects. Users, devices, applications, service principals, managed identities, AI agents that authenticate every few seconds without anyone having explicitly decided they should. The directory got bigger. The authentication got faster. The judgment calls got automated. And somewhere in that automation, the thing that used to be obvious — does this entity actually belong here, and should it have access to this — became a policy document that nobody reads until something breaks.
I've spent fifteen years inside this. I once watched a very confident administrator run a full sync on a live production tenant instead of a delta. Not because they didn't know the difference. Because confidence is comfortable and discomfort is where learning lives. What followed was the kind of meeting where a senior stakeholder says "I want to step back a bit" — which is corporate language for "I need a moment to not be visibly associated with what is currently happening."
Every generation of identity verification has been the best available technology at the time. It started with appearance and voice — the oldest credentials, the ones you were born with. Then names. Then seals and parish records. Then passports. Then fingerprints. Then smart cards and RSA tokens. The credential has always been a proxy for the same underlying question: is this actually you? If the Big Bang started with a bang, every modern organisation starts with an identity layer. Everything else is built on top of it. Your AI agents are presenting tokens to your directory right now, every few seconds, without anyone having explicitly decided they should. The pattern hasn't changed. Only the technology carrying it forward has.
The medieval priest with the cooled iron bar at least knew who he was letting through and why.
We automated the ordeal. We forgot the judgment.
