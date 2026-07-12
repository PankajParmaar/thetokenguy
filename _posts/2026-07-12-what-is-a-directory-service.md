---
layout: post
title: "What Is a Directory Service?"
date: 2026-07-12 00:01:00 +0530
categories: ["Microsoft Entra", "Core Identity"]
tags: ["microsoft entra", "identity", "active directory", "directory service", "fundamentals"]
author: pankaj
---

Identity conversations tend to start with authentication, MFA, or Conditional Access, and that's usually backwards. Before you can authenticate anyone, something has to know who they are in the first place. It needs a place to store identities, organize them, retrieve them quickly, and answer basic questions like whether a user exists, what groups they belong to, whether their account is disabled, and which applications they should see. That place is a directory service, and if you're learning Microsoft Entra, understanding the directory is more valuable than memorizing individual features, because most identity services are simply different ways of reading from or writing to it.

## Think of it as an identity database

At its simplest, a directory service is a specialized database optimized for identity information. Unlike a relational database that stores orders or invoices, a directory stores objects such as users, groups, devices, applications, service principals, administrative roles, and contacts. Every object carries attributes — for a user that might mean display name, UPN, email, department, manager, object ID, and account status — and applications query these attributes constantly. Authentication is just one consumer of that data; everything from license assignment to conditional access evaluation is really a read against the same underlying store.

## Directories are optimized for reading

One thing that separates directories from traditional transactional databases is the shape of their workload. A directory gets read thousands or millions of times a day — a user signing into Microsoft 365, Outlook searching the Global Address List, Teams checking group membership, SharePoint evaluating permissions, an application validating a token — while writes are comparatively rare. Most users change departments once a year but authenticate dozens of times a day, and the underlying design reflects that imbalance heavily toward read performance.

## Objects and relationships matter

A directory isn't just a flat list of users; it's a graph of relationships. A user belongs to groups, groups receive permissions, applications trust users, devices belong to users, managers relate to employees, and dynamic groups depend on user attributes updating correctly. That graph is what makes identity platforms powerful in practice — changing one group membership can immediately affect access across hundreds of applications without anyone touching each application individually, which is also why a stale or incorrectly scoped group is such a common source of access sprawl.

## LDAP is not the directory

People often use LDAP and directory service interchangeably, but they're different layers of the same problem. LDAP is a protocol for talking to a directory, while the directory service itself is the system that actually holds and organizes the identity data — the same way HTTP is how clients communicate but the web server is what stores and serves the content. Modern cloud directories increasingly expose REST APIs like Microsoft Graph instead of LDAP, but the underlying idea hasn't changed: something has to store the objects, and something has to define how you ask questions about them.

## Why cloud directories look different

Traditional directories were built around corporate networks, where users sat inside offices, servers lived in data centers, and the domain was effectively the security boundary. Cloud identity broke those assumptions: identities now authenticate from anywhere, applications live across multiple clouds, and devices may never touch a corporate network at all. Microsoft Entra ID is Microsoft's cloud directory built around that shift — it still stores identities, groups, devices, and applications, but the directory itself became internet-facing rather than network-facing, which changes where those identities get used and how they get authenticated.

## Where everything else fits

Once the directory exists, the rest of the identity stack gets easier to reason about. Authentication is answering "is this really the user?" Conditional Access is asking "should they be allowed right now?" PIM is asking "can they temporarily become an administrator?" and Lifecycle Workflows are asking "what should happen when someone joins or leaves?" Every one of those services ultimately relies on information stored in the directory — remove the directory, and the rest of the stack has nothing left to evaluate.

## Final thought

The directory is the foundation of every identity platform. Features like MFA, Conditional Access, and Privileged Identity Management get most of the attention in architecture reviews, but they're all making decisions based on the same underlying identity data. Understanding the directory first is what makes the rest of Microsoft Entra feel far less mysterious.
