---
layout: post
title: "Understanding Tenants, Subscriptions, and Management Groups"
date: 2026-07-12 00:03:00 +0530
categories: ["Microsoft Entra", "Core Identity"]
tags: ["microsoft entra", "azure", "tenant", "subscription", "management groups", "azure architecture", "powershell"]
author: pankaj
description: "Microsoft runs two hierarchies in parallel: identity in Microsoft Entra, resources in Azure Resource Manager. Where each begins is what makes Azure sensible."
image:
  path: /assets/img/og-default.png
  alt: "Understanding Tenants, Subscriptions, and Management Groups"
---

If you've spent time in both the Microsoft Entra admin center and the Azure portal, you've probably noticed something confusing: sometimes you're managing users, sometimes you're managing virtual machines, sometimes you're assigning RBAC roles, sometimes you're creating Conditional Access policies, and yet everything appears to belong to the same organization. The reason is that Microsoft is really running two separate hierarchies in parallel — an identity hierarchy in Microsoft Entra and a resource hierarchy in Azure Resource Manager. Once you understand where each one begins and ends, a lot of Azure concepts stop feeling disconnected from each other.

## Start with the tenant

A Microsoft Entra tenant is the identity boundary for your organization — think of it as the home for everything related to identity. Inside a tenant you'll find users, groups, devices, applications, service principals, administrative roles, Conditional Access policies, Identity Protection, Privileged Identity Management, and authentication methods. Anything that answers the question "who are you?" belongs in the tenant:

```text
Contoso Tenant
│
├── Users
├── Groups
├── Devices
├── Enterprise Applications
├── App Registrations
├── Conditional Access
└── Administrative Roles
```

A company typically runs one primary production tenant, though mergers, acquisitions, or large enterprises may end up operating several.

## A subscription is a billing and resource boundary

A subscription has nothing to do with users signing in — it's where Azure resources actually live, things like virtual machines, storage accounts, virtual networks, key vaults, Azure SQL, App Services, and AKS clusters. A subscription answers a different set of questions: who pays for this resource, who can deploy resources, what spending limits apply, and which Azure RBAC roles are assigned. A tenant can contain many subscriptions:

```text
Tenant
│
├── Subscription A (Production)
├── Subscription B (Development)
├── Subscription C (Sandbox)
└── Subscription D (Shared Services)
```

The tenant owns identities; the subscriptions own infrastructure, and keeping that distinction straight resolves most of the confusion people run into early on.

## Why multiple subscriptions?

Plenty of newcomers create everything inside a single subscription, and that works fine for labs, but larger environments usually split subscriptions apart for isolation, cost management, access control, and governance. Isolation keeps production and development workloads separate from each other. Cost management gives each subscription its own billing information so finance teams can allocate spend cleanly. Access control lets developers hold Contributor rights in Dev while only getting Reader rights in Production, and governance lets different Azure Policies apply to different subscriptions depending on their risk profile.

## Management groups sit above subscriptions

As environments grow, managing dozens or hundreds of subscriptions individually gets painful fast, which is where Management Groups come in — think of them as folders for subscriptions:

```text
Tenant
│
└── Root Management Group
     │
     ├── Production
     │      ├── Finance Subscription
     │      └── HR Subscription
     │
     ├── Development
     │      ├── Dev Subscription
     │      └── Test Subscription
     │
     └── Sandbox
```

Policies and permissions assigned at the Management Group level flow down to child subscriptions automatically, which makes them extremely useful for enforcing organization-wide standards without touching every subscription by hand.

## Where Azure RBAC fits

A common misconception is that Microsoft Entra roles and Azure RBAC roles are the same thing — they aren't. Microsoft Entra roles like Global Administrator, User Administrator, Authentication Administrator, and Conditional Access Administrator manage identities and grant permissions inside the tenant. Azure RBAC roles like Owner, Contributor, Reader, Virtual Machine Contributor, and Network Contributor manage Azure resources and grant permissions inside subscriptions or resource groups. Someone can be a Global Administrator with only Reader access on Azure subscriptions, or conversely hold no Entra administrative role at all while owning several Azure subscriptions outright — these are genuinely independent permission models, not two views of the same thing.

## A simple way to remember it

Ask two questions. "Who is this?" puts you in tenant territory. "What resource are they managing?" puts you in subscription territory. That simple distinction clears up most Azure architecture discussions before they even get complicated.

## Useful PowerShell

Connect to Microsoft Graph for tenant information:

```powershell
Connect-MgGraph
```

View your organization (tenant):

```powershell
Get-MgOrganization
```

List verified domains:

```powershell
Get-MgDomain
```

Connect to Azure:

```powershell
Connect-AzAccount
```

View available subscriptions:

```powershell
Get-AzSubscription
```

Switch subscriptions:

```powershell
Set-AzContext -Subscription "Production"
```

Display the current Azure context:

```powershell
Get-AzContext
```

List Management Groups:

```powershell
Get-AzManagementGroup
```

View subscriptions under a Management Group:

```powershell
Get-AzManagementGroupSubscription `
   -GroupName "Production"
```

## Visualizing the hierarchy

```text
Microsoft Entra Tenant
│
├── Users
├── Groups
├── Devices
├── Applications
│
└── Root Management Group
     │
     ├── Production
     │      ├── Subscription A
     │      └── Subscription B
     │
     ├── Development
     │      └── Subscription C
     │
     └── Sandbox
            └── Subscription D
```

Identity flows from the tenant, governance flows through Management Groups, and resources live inside subscriptions. Keeping those three layers separate in your mind makes Azure much easier to reason about, especially once RBAC, Azure Policy, and enterprise-scale landing zones enter the picture.
