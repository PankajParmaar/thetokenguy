---
layout: post
title: "Claims-Based Access"
date: 2026-07-12
categories: ["Zero Trust", "Identity as the Perimeter"]
tags: ["claims", "identity", "authorization"]
author: pankaj
description: "You can describe what a Conditional Access policy does without knowing what a claim is. The mechanism that makes identity-first security work underneath."
image:
  path: /assets/img/og-default.png
  alt: "Claims-Based Access"
---
Claims-based access is the mechanism that makes identity-first security actually work under the hood, and it is worth slowing down on because most people who use Entra ID daily can describe what a Conditional Access policy does without being able to describe what a claim actually is or how it gets from the identity provider into the application making the authorization decision.

## What a claim actually is

A claim is a statement about a subject, packaged into a token, and signed by an identity provider the relying application trusts. When a user signs into an application through Entra ID, the token issued back to that application isn't just a yes-or-no on whether authentication succeeded. It carries a set of claims: the user's object ID, their group memberships or role assignments, the tenant they authenticated in, sometimes device state, sometimes the authentication method used. The application reads those claims and makes its own authorization decisions based on them, without ever needing to go back and ask Entra ID "is this actually allowed" a second time. The trust is embedded in the signed token itself.

This is a fundamentally different model from the older approach where an application checks a user's identity against its own local user store, or worse, against a shared directory lookup performed independently by every app. Claims decouple the authentication event from the authorization decision. Entra ID vouches for who the user is and what claims apply to them; the application decides what those claims are allowed to do inside its own scope. That separation is what lets one identity provider serve dozens of applications with completely different authorization models, because each app only needs to trust the claims, not reimplement identity verification itself.

## Where it shows up in practice

In a lab setup, the easiest way to see claims in action is to decode a token issued to a test application registered in Entra ID — the roles claim, the groups claim, the tenant ID, the app ID the token was issued for. Conditional Access policies also operate on claims, evaluating things like the authentication methods claim to enforce MFA, or the device ID claim to check compliance state before the token gets issued in the first place. Group-based or app-role-based authorization inside the application itself is just the app reading a claims array and mapping it to its own permission model, which is why getting group and role design right at the Entra ID layer matters so much — a messy claims payload becomes a messy authorization decision downstream, in every application that trusts it.

## Where claims-based models break down

The failure mode I've run into most often, both in my own test tenants and in write-ups from other practitioners, is claims bloat. Every group a user belongs to can end up in the token unless the app is scoped to specific app roles instead, and tokens have size limits. A user in enough security groups can produce a token that exceeds what some applications can parse, causing silent authorization failures that have nothing to do with the actual permission model and everything to do with token size. The fix is almost always moving from raw group claims to app roles or group-to-role mapping at the enterprise application level, which trims the claims payload to only what that specific app actually needs to make its decision.

The other break is staleness. A claim is only as current as the token it was issued in, and tokens are cached for their lifetime. Revoke a user's access mid-session and the claims in their already-issued token don't update until that token expires or gets refreshed, which is exactly why Continuous Access Evaluation exists — to shrink the window between a real-time authorization change and the claims actually reflecting it, rather than waiting on a token's natural expiry to catch up.
