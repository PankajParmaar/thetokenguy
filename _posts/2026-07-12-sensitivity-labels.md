---
layout: post
title: "Sensitivity Labels"
date: 2026-07-12
categories: ["Zero Trust", "Data Protection"]
tags: ["sensitivity-labels", "information-protection", "data-protection"]
author: pankaj
---
Sensitivity labels are the piece of data protection that most closely resembles a metadata tagging exercise on the surface, and that surface impression is exactly why so many tenants stop at "we've applied labels" without realizing the labels themselves do almost nothing on their own. A label is a classifier. What makes it useful is everything wired to act on that classification after it's applied, and that wiring is where most of the real work sits.

## What a label actually is

A sensitivity label in Microsoft Purview Information Protection is metadata attached to a document or email — Confidential, Highly Confidential, Public, whatever taxonomy the organization defines — that can carry protection settings alongside the classification itself. Those settings can include encryption, restricting forwarding or copying, applying a watermark, or requiring specific permissions to open the content at all. The label travels with the file. Move it from SharePoint to a USB drive to an email attachment, and the label and any encryption tied to it persist, because it's embedded in the file itself rather than enforced only by whatever platform happened to store it at the time.

That persistence is the actual value proposition over platform-level access controls. A SharePoint permission only protects the file while it's in SharePoint. The moment someone downloads it, the SharePoint-level protection is gone, and whatever protection remains is only what got baked into the file through a label at save time. This is the distinction that gets lost when sensitivity labels get pitched as "just a tagging feature."

## How labeling actually happens

In a lab tenant, I set up a small label taxonomy — Public, Internal, Confidential — and tested both manual and automatic labeling. Manual labeling relies entirely on the user recognizing the sensitivity of what they're creating and applying the right label themselves, which works fine for a security-conscious pilot user and works considerably worse at scale, because most people either don't think about it or default to whatever label requires the least friction. Automatic labeling, based on sensitive information types like credit card numbers or specific regex patterns, or trainable classifiers for less structured content, picks up a meaningful share of what manual labeling misses, but it isn't perfect either — trainable classifiers need decent training data to be accurate, and a classifier trained on someone else's sample documents may not generalize well to your organization's actual content patterns.

The gap between manual and automatic labeling is exactly where sensitive content ends up unlabeled in practice: a user creates a document with genuinely confidential content, doesn't apply a label, and no automatic condition happens to match it because the content doesn't contain a recognizable pattern like a national ID number, just contextually sensitive business information a classifier hasn't been trained to catch.

## Where labels alone fall short

The trap I'd flag for anyone building this out is assuming a label is protection by itself. A label with no encryption or access restriction attached is purely informational — useful for search, reporting, and downstream DLP policies that key off the label, but it does not stop anyone from opening, copying, or forwarding the file. The protective value only kicks in once the label is tied to actual restrictions, and even then, protection settings need to be tested against real workflows, because an overly restrictive label — say, one that blocks external sharing entirely — applied to a document that genuinely needs to go to an external partner just gets worked around, usually by someone screenshotting the content or retyping it into an unlabeled document, which defeats the entire purpose of having applied a label in the first place.
