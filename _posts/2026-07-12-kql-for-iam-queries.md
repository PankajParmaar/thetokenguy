---
layout: post
title: "KQL for IAM Queries"
date: 2026-07-12
categories: ["Microsoft Defender", "SIEM & Sentinel"]
tags: ["kql", "identity queries", "azure ad logs"]
author: pankaj
description: "KQL is what everything in Sentinel funnels through eventually. Coming from IAM rather than general SOC work, the tables worth learning first are much narrower."
image:
  path: /assets/img/og-default.png
  alt: "KQL for IAM Queries"
---
KQL is the language everything in Sentinel eventually funnels through, and if you're coming from an IAM background rather than a general SOC background, the tables and query patterns worth knowing first are narrower than the full breadth of what Sentinel supports, so it's worth being deliberate about which parts to learn first.

## The core identity tables

`SigninLogs` is where interactive Azure AD sign-ins land, and it's usually the first table anyone reaches for when investigating a specific user or a suspicious authentication event. `AADNonInteractiveUserSignInLogs` covers the token refresh and background authentication traffic that doesn't involve a user actively typing credentials, which is a huge volume of real traffic that gets missed entirely if you only ever query the interactive log. `AuditLogs` captures the administrative actions, role assignments, app registration changes, group membership edits, and it's the table that answers "who changed this" rather than "who logged in." For anyone running Defender for Identity, `IdentityLogonEvents` and `IdentityDirectoryEvents` bring on-prem AD authentication and directory change activity into the same query surface as the cloud-native tables.

## Patterns worth knowing by heart

A basic risky sign-in query, filtering for failures with a specific error code and grouping by user, looks like this:

```kql
SigninLogs
| where ResultType == "50126"
| summarize FailureCount = count() by UserPrincipalName, bin(TimeGenerated, 1h)
| where FailureCount > 10
```

Impossible travel detection, comparing consecutive sign-ins for the same user across geographically distant locations within an implausible time window, is one of the more valuable patterns to have ready, usually built with `prev()` or a self-join against `SigninLogs` ordered by `TimeGenerated` per user, comparing `Location` fields between consecutive rows.

Privileged role assignment monitoring against `AuditLogs`, filtering `OperationName` for `"Add member to role"` and joining against a watchlist of roles you consider sensitive, like Global Administrator or Privileged Role Administrator, catches escalation activity that a raw sign-in query would never surface, since privilege escalation often doesn't require a fresh authentication event at all.

## Where IAM-specific queries get harder than they look

Time zone handling trips people up constantly, since `TimeGenerated` is UTC by default and a query built to flag off-hours activity needs an explicit conversion to the relevant local time zone or it'll flag perfectly normal daytime activity in a region eight hours off from whoever wrote the query. Guest and B2B accounts are another recurring gap, since their `UserPrincipalName` format and home tenant fields behave differently from a native tenant member, and a query written and tested only against native accounts will often silently under-match or over-match once guest activity is in the mix. Correlating `SigninLogs` with `AuditLogs` by user identity also needs care, because the identifier fields aren't always named or formatted consistently across tables, `UserId` versus `InitiatedBy` versus `UserPrincipalName`, and joining on the wrong field produces a query that runs without error but silently returns nothing or returns the wrong rows.

## A habit worth building

Saving working queries as Sentinel functions rather than copy-pasting raw KQL between analysts is a small habit that pays off fast, since IAM-specific logic like risky role definitions or your org's specific off-hours window is exactly the kind of thing that should live in one place and get referenced everywhere, not get slightly reinvented by each person who touches the workspace.

KQL for identity work rewards depth in a small number of tables over broad shallow familiarity with everything Sentinel ingests, and `SigninLogs` plus `AuditLogs` plus whatever Defender for Identity tables are relevant will cover the large majority of real IAM investigation work.
