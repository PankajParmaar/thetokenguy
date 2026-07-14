---
layout: post
title: "Session Controls"
date: 2026-07-12
categories: ["Microsoft Defender", "Cloud App Security"]
tags: ["session controls", "conditional access app control", "reverse proxy"]
author: pankaj
description: "Conditional Access discussions stop at the front door: allow or block sign-in. Session controls keep working after, via a mechanic that isn't intuitive."
image:
  path: /assets/img/og-default.png
  alt: "Session Controls"
---
Most conditional access discussions stop at the front door: allow or block the sign-in. Session controls are the part that keeps working after the user is already in, and they're the part that trips people up because the mechanism behind them, a reverse proxy, isn't intuitive if you're used to thinking of conditional access as a simple gate check.

## The reverse proxy mechanic

Conditional access app control, the feature that delivers session controls, works by routing traffic through the Defender for Cloud Apps reverse proxy rather than letting the browser talk to the SaaS app directly. When a conditional access policy targets an app for session control, Azure AD redirects the authentication flow so that instead of getting a token straight to, say, the real login endpoint, the user's session gets proxied. URLs get rewritten on the fly so the browser is actually talking through Defender for Cloud Apps the entire session, which is what lets Microsoft inspect and control the traffic in real time rather than only at initial authentication.

This is meaningfully different from the block/allow logic in standard conditional access, because it lets you apply granular controls after the user has already authenticated successfully. Someone can pass MFA cleanly and still hit a session control that blocks a file download because they're on an unmanaged device, or that applies a watermark and restricts printing on sensitive documents opened in a browser.

## What you can actually enforce mid-session

The practical controls that get used most are blocking downloads of sensitive content based on sensitivity labels, blocking cut/copy/paste and printing for sessions from unmanaged devices, applying real-time DLP scanning on uploads, and monitoring sessions for risky users without blocking anything, just to gather evidence. That last one, monitor-only, is underused. It's a reasonable way to observe what a flagged user is actually doing in a sensitive app before deciding whether tighter session controls are warranted, rather than jumping straight to blocking and generating help desk tickets.

## Where it breaks

Session controls only work for apps that support the proxy mechanism cleanly, and not every SaaS app renders correctly once its URLs get rewritten by the proxy. Apps with heavy client-side JavaScript routing, or ones that pin certificates and reject traffic that doesn't match expected origin behavior, can break in ways that look like an outage to the end user rather than a security control doing its job. Testing session control policies against a pilot group before rolling out broadly isn't optional if the app in question wasn't already validated by Microsoft's catalog of apps confirmed to work well with the proxy.

The other practical issue is performance perception. Routing every request for a session through an extra hop adds latency, and while it's usually small, it's noticeable enough on latency-sensitive workflows that users will complain the app feels slower, and that complaint often lands on the security team's desk before anyone traces it back to the proxy.

Session controls close a real gap between authentication-time decisions and what actually happens during a session, but they're a proxy in the literal sense, and anything that reroutes traffic through a middle layer needs the same scrutiny you'd give any other man-in-the-middle architecture, even when your own team put it there on purpose.
