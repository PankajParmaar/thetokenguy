---
layout: post
title: "Zero Trust Is Not a Product"
date: 2026-07-12
categories: ["Zero Trust", "Principles & Architecture"]
tags: ["zero-trust", "security-architecture", "identity"]
author: pankaj
description: "Every vendor booth has a Zero Trust banner implying you just buy the box and the problem goes away. That framing has done more damage than anything else."
image:
  path: /assets/img/og-default.png
  alt: "Zero Trust Is Not a Product"
---
Every vendor booth I have walked past in the last few years has a banner with "Zero Trust" on it somewhere, usually next to a checkmark implying you buy the box and the problem goes away. That framing has done more damage to the concept than almost anything else, because it convinces buyers that Zero Trust is a SKU you procure rather than an architecture you build over years, across identity, device, network, application, and data layers that were never designed to talk to each other in the first place.

## What it actually is

Zero Trust is a design philosophy that assumes no implicit trust based on network location. A request coming from inside the corporate LAN gets exactly as much scrutiny as one coming from a coffee shop. That single sentence is the whole model, and everything else — Conditional Access policies, device compliance, microsegmentation, sensitivity labels — is an implementation detail in service of it. The moment you buy a product because its marketing says "Zero Trust," you have already misunderstood what you are trying to do, because no single control point can verify identity, device health, network context, and data sensitivity all at once.

In my lab environment I have set up Entra Conditional Access, Intune compliance, and Microsoft Defender for Cloud Apps to work together, and what becomes obvious quickly is that each of these tools solves one narrow slice of the trust decision. Conditional Access can block a sign-in based on device compliance state, but it has no idea whether the data the user is about to touch is classified as confidential. That gap is closed by sensitivity labels and DLP policies, which live in an entirely different console with an entirely different policy engine. Zero Trust as an architecture is the discipline of stitching these signals together into one coherent access decision, not any one of the tools involved.

## Where the product framing breaks

The failure mode I have seen described in postmortems and case studies, and reproduced myself in test tenants, is buying the "Zero Trust" capable license tier and assuming the architecture is now in place. It is not. A tenant can have every Entra ID P2 feature enabled and still have standing global admin roles, flat networks with no segmentation, and shared service accounts with passwords that never rotate. The product exists; the architecture does not. Zero Trust maturity is measured by how few implicit trust decisions remain in your environment, not by which licenses appear on your invoice.

The other trap is treating Zero Trust as a project with an end date. Because it touches every layer of the stack, it behaves more like a continuously reassessed posture. New SaaS apps get onboarded, new device types show up, new API integrations get built, and each one reintroduces implicit trust unless someone deliberately closes the gap. Treating it as a finished initiative is how organizations end up with a Zero Trust slide in the security roadmap deck and a flat, over-permissioned network sitting right underneath it.

## The starting point that actually works

If I were advising someone standing up this architecture from scratch in a lab or a small tenant, I would start with identity, because it is the one signal available in every other layer's policy engine. Device compliance state, network location, and even DLP conditions can all be evaluated as claims tied to an authenticated identity. Get identity verification and Conditional Access right first, then layer device trust, then network segmentation, then data controls. Buying the product in whatever order the vendor's pricing page suggests is how you end up with expensive shelfware and the same implicit trust you started with.
