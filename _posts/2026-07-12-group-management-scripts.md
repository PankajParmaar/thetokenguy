---
layout: post
title: "Group Management Scripts"
date: 2026-07-12
categories: ["PowerShell & Graph", "Identity Automation"]
tags: ["powershell", "groups", "entra-id", "access-management"]
author: pankaj
description: "Groups are the quiet backbone of nearly every access decision in Microsoft 365 — exactly why stale membership is such a persistent, compounding problem."
image:
  path: /assets/img/og-default.png
  alt: "Group Management Scripts"
---
Groups are the quiet backbone of almost every access decision in a Microsoft 365 tenant — conditional access policies, license assignment, SharePoint permissions, application role assignments, most of it eventually traces back to group membership. Which is exactly why group sprawl and stale membership are such a persistent problem: group ownership rarely gets assigned when the group is created, so there's no name attached when it comes time to review who's still in it. I've hit this directly during an access review — asked who owned a group granting access to a sensitive finance SharePoint site, and the owner field in Entra was blank, with the last membership change dated over two years earlier.

## Membership audits

The starting point for any group hygiene work is just seeing what's actually there:

```powershell
$group = Get-MgGroup -Filter "displayName eq 'Finance-Confidential-Access'"
Get-MgGroupMember -GroupId $group.Id -All |
    Select-Object Id, @{N='Name';E={$_.AdditionalProperties.displayName}}
```

I run this against sensitive-access groups specifically, cross-referenced against an HR export, to catch members who've left the department or the company but never got removed. The AdditionalProperties quirk above — Graph returns group members as generic directory objects, so display name and other properties live in a nested hashtable rather than as top-level fields — trips people up the first time they try to filter or sort on name directly.

## Bulk membership changes

Adding or removing members in bulk is one of the more common asks after a reorg:

```powershell
foreach ($userId in $userIdsToAdd) {
    New-MgGroupMember -GroupId $group.Id -DirectoryObjectId $userId
}
```

Removal follows the same shape with `Remove-MgGroupMemberDirectoryObjectByRef`, and the detail worth flagging is that removing someone from a group that feeds conditional access or licensing takes effect on next token refresh, not instantly — a user might retain access for up to an hour depending on token lifetime, which matters if you're doing an urgent offboarding and expecting immediate effect.

## Dynamic groups as the better long-term answer

For anything rule-based — department, job title, license type — dynamic membership rules beat manual scripts entirely, because the group maintains itself:

```
(user.department -eq "Finance") and (user.accountEnabled -eq true)
```

The catch with dynamic groups is that rule syntax errors fail silently in the sense that the group just doesn't populate as expected, and Graph's error messages for malformed rules aren't always specific about which clause is wrong. I test new rules against a small pilot group first rather than pointing them directly at a group tied to production access.

## The unglamorous payoff

None of this is complicated logic, but group management is one of those areas where a small scheduled script — checking stale membership monthly, flagging groups with no owner, comparing dynamic rule output against expectations — quietly prevents the kind of access sp