---
layout: post
title: "Tenant Setup Checklist"
date: 2026-07-12
categories: ["Microsoft 365", "Tenant Administration"]
tags: ["microsoft365", "tenant", "administration", "identity"]
author: pankaj
description: "Every tenant setup starts the same way: sign up, add a domain, create a few accounts, call it done. Decisions skipped early become load-bearing much later."
image:
  path: /assets/img/og-default.png
  alt: "Tenant Setup Checklist"
---
Every Microsoft 365 tenant I've poked around in my lab started the same way: someone signed up, added a domain, created a few accounts, and called it done. That's fine for a trial, but if you're setting one up to actually study how the platform behaves, or standing one up for a small org, skipping the early decisions costs you later. Tenant-level settings have a habit of becoming load-bearing once users, mail, and data pile on top of them, and unwinding a bad default six months in is a different kind of work than getting it right on day one.

## Naming and directory basics

The initial tenant name (the `.onmicrosoft.com` value) sticks around as a fallback UPN suffix even after you add a custom domain, so treat it as permanent rather than cosmetic. Pick something that won't be confusing to see in mail headers or sign-in logs later. Right after that, decide your directory's default location and language settings, because they drive default DLP sensitive information types, compliance region defaults, and which admin center you land on. If you're doing this for study purposes, set the country/region to match wherever you're actually testing from, since some compliance and Purview features behave differently depending on data residency assumptions.

## Licensing before users

It's tempting to create users first and assign licenses as an afterthought. Don't. Group-based licensing (through Entra ID dynamic or assigned groups) is far easier to set up when no users exist yet than to retrofit onto a directory full of manually-licensed accounts with drifted assignments. Decide early whether you're going to license by group or license individually, because mixing the two approaches across a tenant creates the kind of inconsistency that makes troubleshooting "why doesn't this person have Teams" take twenty minutes instead of two.

## Security defaults and break-glass accounts

New tenants ship with Security Defaults enabled, which forces MFA registration and blocks legacy authentication. If you plan to use Conditional Access instead (which you should, once you're past a handful of users), you have to explicitly turn Security Defaults off, and Entra ID will nag you about it in the recommendations blade until you do. Before you touch any of that, create at least two break-glass accounts: cloud-only, excluded from every Conditional Access policy, with long random passwords stored somewhere outside the tenant itself. I've locked myself out of a lab tenant exactly once by being clever with Conditional Access policy targeting, and it was enough to make break-glass accounts a non-negotiable first step every time since.

## Global admin count

Set a hard rule for yourself on how many Global Administrator accounts exist and stick to it. Two is usually the right number for a small setup — enough for redundancy, few enough that you actually know who has the keys. Everything else should run through more scoped Entra roles, which is really its own topic (see admin roles below), but the discipline starts at tenant creation, not after you've handed out Global Admin to make an integration work faster.

## Audit logging and diagnostic settings

Unified audit log and mailbox audit logging are on by default in modern tenants, but it's worth confirming rather than assuming, especially if the tenant was created a while back under older defaults. If you want log retention beyond the default window, or you want sign-in and audit data flowing into a SIEM or a Log Analytics workspace, wire up diagnostic settings in Entra ID at setup time. Retrofitting log pipelines after an incident, when you actually need the history, is the wrong time to discover you never turned it on.

Getting a tenant's bones right at creation doesn't take long, but it's the kind of work that only shows up on an audit checklist when it's missing — a diagnostic setting that was never wired up, a break-glass account that doesn't exist, a G