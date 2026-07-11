---
layout: post
title: "App Discovery"
date: 2026-07-12
categories: ["Microsoft Defender", "Cloud App Security"]
tags: ["cloud discovery", "shadow it", "defender for cloud apps"]
author: pankaj
---
Every org I've looked at, including my own lab setups where I've deliberately let traffic accumulate before running discovery, turns up a longer list of SaaS apps in use than anyone expects. Cloud discovery in Defender for Cloud Apps is the feature built to surface exactly that gap between the apps IT thinks are in use and the apps actually generating traffic.

## How the data actually gets in

Cloud discovery works by ingesting traffic logs, not by scanning the internet for what your users might be using. Those logs can come from a manually uploaded export off a firewall or secure web gateway, an automated log collector deployed as a Docker container or Windows service that continuously forwards logs, or, if Defender for Endpoint is already deployed, directly from endpoint telemetry with no separate log collector needed at all. That last option changed the calculus for a lot of smaller orgs, because standing up a log collector and getting network gear to forward logs correctly used to be the single biggest blocker to actually turning discovery on.

Once logs are ingesting, Defender for Cloud Apps matches the traffic against its own catalog of roughly 30,000 cataloged cloud apps, scoring each one against more than 90 risk factors: things like whether the app supports multi-factor authentication, its data-at-rest encryption practices, and its terms of service around data ownership. That scoring is what turns a raw list of visited domains into something you can actually act on, since a risk score of 2 out of 10 for an unsanctioned file-sharing app is a very different conversation than the same discovery result for a well-known enterprise SaaS tool that just hasn't been formally onboarded yet.

## From discovery to decision

The output isn't meant to sit as a report. Each discovered app can be tagged sanctioned, unsanctioned, or left unreviewed, and unsanctioned tagging can trigger actual enforcement if you've got the network appliance or Defender for Endpoint integration wired up to block traffic to that app going forward. This is where discovery becomes governance rather than a report that gets generated once for an audit and then sits untouched until the next audit cycle rolls around.

The other decision point is app governance, a separate but related capability that goes past network-level discovery into OAuth app permissions granted inside Microsoft 365 and other supported platforms. That distinction matters because a lot of real shadow IT risk isn't a user visiting an unsanctioned website, it's a user granting a third-party app broad Graph API permissions through an OAuth consent screen, which cloud discovery's traffic-log approach won't catch on its own.

## Where it gets messy

The most common mistake is running discovery once, generating a report, and treating the job as done. Shadow IT usage changes weekly as new SaaS tools launch and teams sign up for free tiers without any procurement involvement, so discovery needs to be a recurring log ingestion, not a one-time audit. The second issue is log quality: if your log collector is only capturing a subset of egress traffic, or a segment of the network routes around the monitored gateway entirely, the discovery results will look clean while missing a real chunk of actual usage.

Discovery is only as good as the traffic it can see, and the gap between what shows up in the dashboard and what's actually happening on your network is usually a rou