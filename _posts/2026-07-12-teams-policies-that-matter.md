---
layout: post
title: "Teams Policies That Matter"
date: 2026-07-12
categories: ["Microsoft 365", "Teams & Collaboration"]
tags: ["teams", "policies", "governance", "admincenter"]
author: pankaj
---
The Teams admin center has policy categories for nearly everything — messaging, meetings, calling, live events, app permissions — and it's easy to spend an afternoon in there without touching the handful of settings that actually shape day-to-day risk and user experience. A few of these are worth understanding properly rather than leaving on whatever the tenant defaulted to.

## Meeting policies and anonymous join

The setting that trips people up most is who can bypass the lobby. The default in a lot of tenants allows "everyone in your organization" to skip the lobby, which is reasonable, but the setting for anonymous or external participants is worth checking explicitly rather than assuming. If your org runs meetings with external guests regularly, "people in my org and guests" bypassing the lobby is usually fine, but leaving anonymous users able to bypass the lobby on a meeting policy applied broadly means literally anyone with the join link walks straight into a call without anyone approving it. I've seen this default catch orgs off guard when a meeting link got forwarded further than intended and someone unexpected was suddenly in the room before the host noticed.

Recording and transcription policies matter for a different reason: once you enable meeting recording, recordings land in the organizer's OneDrive or the Team's SharePoint site by default, and retention on that content is governed by whatever OneDrive/SharePoint retention policy applies, not by a Teams-specific setting. That means enabling recording tenant-wide has real downstream storage and compliance implications that aren't obvious from the Teams admin center screen where you flip the toggle.

## Messaging policies and the delete/edit question

Whether users can delete sent messages, edit sent messages, and use Giphy or memes in chat all live in messaging policies, and they get assigned per user or per group, not just tenant-wide. The one worth thinking through deliberately is message deletion: if your organization has any eDiscovery or regulatory retention obligation, letting users delete messages from the Teams client doesn't actually remove them from the compliance record (retention policies and legal holds preserve the underlying data), but it does remove them from the conversation as everyone sees it, which is a UX and trust question worth a deliberate decision rather than an accidental default.

## App permission policies

Third-party and custom Teams apps get governed through app permission policies, and the default posture in many tenants allows all Microsoft apps, all third-party apps from the store, and custom apps uploaded by users. That's a broad surface — a rogue or poorly reviewed third-party app requesting Graph API permissions through Teams is a real supply-chain-style risk, not a theoretical one. Scoping this down to an explicit allow-list of vetted apps, and turning off the ability for end users to upload custom/sideloaded apps without admin approval, is a policy change that costs almost nothing in productivity for most orgs and closes off a path that's genuinely been abused elsewhere.

## Policy assignment is per-user, and that's the point

The thing that makes Teams policies actually useful rather than a single tenant-wide switch is that nearly all of them support per-user and per-group assignment through policy packages or direct assignment. A pilot group testing a new meeting policy, an executive group with tighter external access restrictions, a contractor group with locked-down app permissions — all of that is normal, supported configuration, not a hack. The mistake is treating Teams policy as one global setting when the tooling was built from the start to be granular, and not using that granularity means everyone gets the lowest-common-denominator policy instead of the one that actually fits their risk profile.
