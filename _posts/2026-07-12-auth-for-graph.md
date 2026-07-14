---
layout: post
title: "Auth for Graph"
date: 2026-07-12
categories: ["PowerShell & Graph", "Graph API"]
tags: ["microsoft-graph", "authentication", "app-registration", "oauth"]
author: pankaj
description: "More Graph scripts fail at authentication than at query logic, and the error message rarely says which auth problem it is. How Connect-MgGraph really works."
image:
  path: /assets/img/og-default.png
  alt: "Auth for Graph"
---
More scripts fail at the authentication step than at the actual query logic, and it's rarely obvious from the error message which of the several possible auth problems is the actual cause. Getting comfortable with how Graph authentication actually works — rather than copy-pasting a Connect-MgGraph line from a blog post and hoping — saves a disproportionate amount of debugging time down the line.

## Delegated vs application permissions

This is the distinction that trips up more people than anything else in Graph auth. Delegated permissions mean the script runs in the context of a signed-in user, and it can only do what that user is allowed to do — an interactive `Connect-MgGraph` prompt is delegated auth. Application permissions mean the script runs as itself, as an app registration with its own consented permission set, independent of any signed-in user, which is what an unattended scheduled script actually needs.

```powershell
# Delegated - interactive, ties to signed-in user
Connect-MgGraph -Scopes "User.Read.All"

# Application - unattended, uses app registration credentials
$secureSecret = ConvertTo-SecureString $clientSecret -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential($clientId, $secureSecret)
Connect-MgGraph -TenantId $tenantId -ClientSecretCredential $cred
```

A scheduled task using delegated auth is a script waiting to fail the first time nobody's interactively available to satisfy an MFA prompt, which is exactly the situation where the "why did last night's job just hang" investigation starts.

## Certificate over client secret for anything unattended

Client secrets work but they're a string sitting in a script or a secrets vault with an expiration date someone has to remember to rotate. Certificate-based authentication is the better long-term choice for unattended app registrations, because certificates integrate more cleanly with things like Azure Key Vault and don't have the same "someone forgot to rotate the secret before it expired at 2am on a Sunday" failure mode that plain client secrets do.

```powershell
Connect-MgGraph -TenantId $tenantId -ClientId $clientId -CertificateThumbprint $thumbprint
```

## Consent is a separate problem from permission scope

Adding a permission scope to an app registration's manifest doesn't do anything by itself — an admin still has to grant consent for that permission, and in most tenants that requires a Global Administrator or Privileged Role Administrator to click the consent button (or run the equivalent Graph call) before the permission is actually usable. This is the single most common cause of "my script has the right scope listed but Graph still returns a 403" — the scope exists in the app registration but was never actually consented, and the error message doesn't distinguish between "you don't have this permission" and "this permission exists but nobody approved it yet."

```powershell
# Checking granted (not just requested) permissions
Get-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $spId
```

## The habit worth building

Every auth failure I've debugged eventually came down to one of three things: wrong permission type for the use case, an unconsented scope, or an expired credential nobody was tracking. Building a habit of checking all three explicitly — rather than assuming the error message points at the actual cause — turns Graph auth debugging from guesswork into a five-minute checklist.
