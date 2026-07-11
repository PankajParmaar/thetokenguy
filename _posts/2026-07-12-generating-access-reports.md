---
layout: post
title: "Generating Access Reports"
date: 2026-07-12
categories: ["PowerShell & Graph", "Reporting & Auditing"]
tags: ["powershell", "access-reviews", "entra-id", "reporting"]
author: pankaj
---
Access reports are the artifact an auditor asks for on day one of every review, and building one from scratch under deadline pressure is a bad way to spend that first day. "Show me who has access to the finance SharePoint site" or "list every user with an active role assignment" sounds like a query someone should be able to answer in five minutes, and in a tenant without prior reporting scripts, it usually takes an afternoon of manual cross-referencing instead.

## Role assignment reports

Active directory role assignments are the most commonly requested report, and pulling them cleanly means joining role definitions to role assignments, since Graph returns the assignment with a role definition ID rather than a readable name by default:

```powershell
Connect-MgGraph -Scopes "RoleManagement.Read.Directory"

$roleAssignments = Get-MgRoleManagementDirectoryRoleAssignment -All -ExpandProperty Principal
$roleDefs = Get-MgRoleManagementDirectoryRoleDefinition -All

$roleAssignments | ForEach-Object {
    $roleName = ($roleDefs | Where-Object Id -eq $_.RoleDefinitionId).DisplayName
    [PSCustomObject]@{
        User = $_.Principal.AdditionalProperties.userPrincipalName
        Role = $roleName
    }
} | Export-Csv .\role-assignments.csv -NoTypeInformation
```

That join step is the part most first-attempt scripts miss, and the result without it is a report full of GUIDs that means nothing to whoever's reading it in a compliance review.

## Group-based access reports

For resource access mediated through groups — the far more common pattern than direct role assignment in most tenants — the report needs to trace group membership back to the resource the group grants access to, which Graph doesn't hand you as a single query. I build these in two steps: pull group membership, then separately pull which applications or SharePoint sites reference that group, and join them locally.

```powershell
$group = Get-MgGroup -Filter "displayName eq 'Finance-App-Access'"
$members = Get-MgGroupMember -GroupId $group.Id -All
$members | Select-Object @{N='User';E={$_.AdditionalProperties.userPrincipalName}}, @{N='Group';E={$group.DisplayName}}
```

## Making reports readable for non-technical stakeholders

A CSV of UPNs and role names is useful to another IAM admin and close to meaningless to a compliance reviewer or a business owner doing an access certification. The step that actually makes these reports usable is adding a human-readable business context column — department, manager, last sign-in date — pulled from the user object alongside the access data, so the report answers "does this access still make sense" rather than just "who technically has it."

```powershell
$members | ForEach-Object {
    $u = Get-MgUser -UserId $_.Id -Property Department,JobTitle
    [PSCustomObject]@{
        User = $u.UserPrincipalName
        Department = $u.Department
        JobTitle = $u.JobTitle
    }
}
```

That extra Graph call per user is slower and worth batching carefully in large tenants, but it's the difference between a report that gets rubber-stamped without being read and one that actually surfaces a stale access grant someone recognizes as wrong the moment they see the depar