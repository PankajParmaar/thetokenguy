---
layout: post
title: "The Iron Bar Never Went Away"
hero_title: "The Iron Bar Never Went Away"
date: 2026-07-12 00:10:00 +0530
categories: [featured]
sitemap: false
hidden: true
tags: [identity, access-tokens, history, jwt, zero-trust]
author: pankaj
description: "Six hundred years of identity systems have swapped iron bars for JWTs but never solved the real problem: trust that gets granted once and is never checked again."

image:
  path: /assets/img/og-default.png
  alt: "The Iron Bar Never Went Away"
---

Every identity system for 600 years has answered the same question — who do we trust, and how do we know? None of them reliably answer the harder one: do they still deserve that trust right now? A token can be valid and a permission can be active long after the reason either was granted has disappeared.

In ninth-century Europe, someone accused of a crime might carry a red-hot iron bar nine steps. Three days later, a priest inspected the wound. Clean healing meant God had ruled the person innocent.

The iron had nothing to do with it. The system worked because people trusted the priest to read the result honestly — and the priest could bank the coals cooler for anyone he already believed. Four hundred years of criminal justice ran on that one unwritten permission.

Six centuries later, the iron bar became a token.

A token is a signed promise: someone verified you, and here's what you're allowed to do because of it. It carries an issuer, an audience, and an expiry — three facts, and one hidden assumption: that the promise is still true for as long as the expiry says it is.

In Entra ID, that promise typically lasts an hour. Long enough that you don't have to log in every ten minutes. Long enough, too, that a stolen token works exactly as well as a real one, for the whole hour, no matter which continent it's used from.

This is how AiTM attacks succeed without breaking anything. The user logs in for real. MFA completes for real. The session cookie issues, and the attacker copies it in transit. Every signal a defender would check — the signature, the MFA claim, the sign-in log — comes back clean. The only thing that changed is who's holding the token now, and nothing in the token records that.

The industry's answer was Continuous Access Evaluation. Instead of trusting a token until it expires, the resource can kill it mid-session the moment something changes — the account gets disabled, the password gets reset, someone logs in from a location the policy doesn't allow. CAE doesn't ask "was this valid an hour ago?" It asks "does this still deserve trust right now?" — the same question the priest was supposed to ask and usually didn't.

Where CAE runs end to end, token lifetimes stretch to a full day, because revocation no longer waits for expiry to catch up. An estimated 30 to 40 percent of most enterprise application estates still can't speak the protocol — older line-of-business apps, third-party integrations that were never built for it. For that slice, the old rule returns: a stolen token is good for whatever time is left on the clock, and nobody's watching in between.

Most tenants still run two systems at once: the old per-user MFA and SSPR settings, and the newer, unified Authentication Methods Policy meant to replace them. Since September 2025, admins can no longer edit the old settings — but whatever was already configured there keeps running, respected right alongside the new policy, until someone deliberately switches the tenant over.

That overlap is the trap. A method disabled in the new policy can still work if it's quietly still enabled in the old one — Entra checks all of them, and a user only needs one to say yes. Nothing alerts an admin when that happens. It shows up later, usually when someone asks why an account can still sign in with a method that was supposedly turned off months ago.

Passkeys close the gap AiTM opened. Instead of a password or a code that can be copied in transit, the proof lives on the device and never leaves it — nothing crosses the wire for an attacker to catch. The device also checks that it's talking to the real domain before it responds, so a proxy sitting on a lookalike page gets refused, not relayed.

It's a real fix, but only where it can reach — and only as strong as whatever recovery option sits behind it. A modern app talking to a modern identity provider gets the full benefit. A line-of-business system still authenticating the way it did in 2014 doesn't. The strongest method available means nothing to the systems still running on the weakest one, and an unhardened fallback path is just one more trust relationship left unchecked.

In 2023, a hacking group tied to Chinese intelligence read the email of a U.S. Commerce Secretary and an American ambassador, using a cloud identity provider's signing key that had been sitting unrevoked since 2016. Every token they forged with it was cryptographically valid. Every signature check passed. The provider didn't catch it. A government analyst noticed the access pattern looked wrong, a month in.

The review board that investigated called it "the cryptographic equivalent of crown jewels" — and found the breach came down to a cascade of ordinary failures, not a broken algorithm. Nothing about the math was ever in question. The key just kept working long after anyone should have still trusted it.

The iron bar disappeared a thousand years ago. The habit it ran on is still signing production tokens today.
