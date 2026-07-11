---
layout: post
title: "Microsegmentation Basics"
date: 2026-07-12
categories: ["Zero Trust", "Network Access Control"]
tags: ["microsegmentation", "network-security", "zero-trust"]
author: pankaj
---
Microsegmentation gets mentioned constantly in Zero Trust literature as the network-layer counterpart to least privilege, but the actual mechanics of how it works, and why it's harder to retrofit than most people expect, don't get discussed nearly as often as the concept itself.

## What it actually means

Traditional network segmentation splits a network into broad zones — a DMZ, an internal zone, maybe a separate zone for servers versus workstations — and trusts everything inside a given zone to talk to everything else in that same zone freely. Microsegmentation shrinks the unit of trust down much further, in principle to the level of individual workloads or even individual processes, so that a web server can talk to its specific database on its specific port, and nothing else on that same subnet gets any assumed reachability at all. The goal is that if any single workload is compromised, the attacker's lateral movement options are limited to the exact, narrow set of connections that workload was explicitly permitted to make, rather than the entire subnet or VLAN it happens to sit on.

## How it's actually implemented

In practice, microsegmentation shows up as software-defined policy enforced close to the workload rather than purely at a perimeter firewall — host-based firewall rules, hypervisor-level network policy, or a dedicated segmentation platform that tags workloads and enforces communication rules based on those tags rather than IP ranges. I tested a simplified version of this in a lab using host firewall rules between a small set of VMs representing a web tier, an app tier, and a database tier, explicitly denying all traffic by default and only opening the specific ports each tier needed to reach the next one. The immediate practical effect was that a test VM I intentionally left in the "web tier" group could reach the app tier on its expected port and nothing else, including other web-tier VMs sitting on the same subnet, which is the entire point — the subnet stopped being the unit of trust.

## Where it gets hard

The difficulty with microsegmentation isn't the concept, it's the discovery work that has to happen before you can write sane policy. You cannot segment traffic you don't understand, and most existing environments have years of accumulated, undocumented east-west traffic — a reporting job that pulls from a database three subnets over, a legacy print server that a finance app still calls at month-end close, a monitoring agent that reaches every host on the VLAN because someone opened it wide during a troubleshooting session two years ago and never closed it back down. Writing a default-deny microsegmentation policy without first mapping actual traffic flows is how you break production, because a legacy batch job that talks to three other systems overnight gets cut off the moment the new policy goes live, and the dependency only surfaces when the batch job's failure alert fires the next morning.

This is why real microsegmentation projects tend to start with a monitoring-only phase, observing actual traffic patterns for weeks before writing a single enforcement rule, and even then the policy gets rolled out tier by tier rather than tenant-wide in one shot. The organizations that skip this discovery step and try to jump straight to enforcement are the ones who end up rolling the whole thing back after the first outage, which then makes the next microsegmentation attempt politically much harder to greenlight.

## Where it fits with everything else

Microsegmentation is the network-layer expression of the same least-privilege principle that PIM and entitlement management apply to identity — narrow the blast radius to only what's actually needed, rather than trusting a broad zone by default. It doesn't replace identity-based controls; a compromised, over-privileged identity can sti