---
layout: post
title: "App Registrations Explained: Why the Identity Isn't Where You Think It Is"
date: 2026-07-12
categories: ["Microsoft Entra", "Non-Human Identity"]
tags: ["microsoft entra", "app registrations", "service principals", "oauth", "microsoft graph", "workload identity", "powershell"]
author: pankaj
description: "Assuming the app registration that created a resource is the one still authenticating is expensive. Why the identity behind a 403 isn't where you think."
image:
  path: /assets/img/og-default.png
  alt: "App Registrations Explained: Why the Identity Isn't Where You Think It Is"
---

One of the most expensive mistakes during an identity incident is assuming the object that created an application is the same object currently authenticating. A Microsoft Graph integration starts returning `403 Forbidden`, an Azure Automation runbook suddenly loses access to Key Vault, or a deployment pipeline that has worked for months begins failing after a customer onboards a new tenant. The investigation immediately heads to App registrations, because that's the blade the engineer opened the day they set up the integration and clicked "New registration." Meanwhile the object actually making authorization decisions — the tenant's Service Principal — never gets opened, and thirty minutes disappear comparing App IDs, Object IDs, and Enterprise Applications while the outage continues.

The distinction becomes even more painful once applications leave the comfort of a single tenant. A SaaS platform might have one global application registration, but every customer tenant materializes its own Service Principal with its own consent state, RBAC assignments, Conditional Access evaluation, provisioning configuration, and lifecycle. The application definition rarely changes during normal operations; the operational identity changes constantly. Most production outages aren't caused by a broken App Registration — they're caused by tenant-specific state attached to a Service Principal — a revoked consent grant, a changed RBAC assignment, an expired certificate on that tenant's instance specifically — that stays invisible because the App Registration blade still shows the application as perfectly healthy.

## App-only authentication changes the troubleshooting model

The moment an application switches to the client credentials flow, most of the assumptions engineers rely on during user authentication disappear. There is no user object contributing claims, no delegated permission model to inspect, and no human identity providing context for authorization decisions. Every access token is issued directly to the application identity, and every downstream authorization check becomes entirely dependent on the Service Principal — Microsoft Graph evaluates application permissions, Azure RBAC evaluates role assignments, Key Vault evaluates data-plane permissions, and custom APIs evaluate application roles without a single human identity participating anywhere in the transaction.

That's exactly why these outages consume so much time. Engineers instinctively start examining user accounts, Entra group memberships, or Conditional Access reports because that's how they troubleshoot interactive authentication, but none of those objects are even participating in the request anymore. The token contains application claims rather than user claims, downstream resources only recognize the Service Principal, and authorization failures almost always trace back to tenant-specific permissions rather than anything associated with an individual administrator. Until the investigation pivots away from people and toward workload identities, it's remarkably easy to spend an hour proving that every user account is perfectly healthy while the application continues failing exactly as before.

## Multi-tenant applications multiply operational state

Multi-tenant architecture scales application distribution by distributing identity state across every customer directory that consumes it. One application registration can project into thousands of tenants, each creating an independent Service Principal carrying its own admin consent history, user consent configuration, Enterprise Application settings, Conditional Access evaluation, RBAC assignments, and local administrative decisions. That separation is what allows every customer to govern the application independently, but it's also why large SaaS providers spend far more time debugging tenant-specific identity state than application code.

The operational failures are rarely dramatic. One customer disables the Enterprise Application after a security review. Another tightens user consent settings so a previously functional onboarding flow now requires administrator approval. A third removes Microsoft Graph application permissions during an audit without realizing another workload still depends on them. Yet another blocks new service principal creation through custom governance policies. From the application's perspective every one of those failures looks almost identical: authentication succeeds, token acquisition succeeds, authorization fails — meanwhile the global App Registration remains completely unchanged because nothing is actually wrong with it.

Consent becomes part of operational state rather than a deployment checkbox. A successful SaaS rollout isn't defined by publishing an application registration or deploying code into production; it's defined by whether every customer tenant successfully created the Service Principal, granted the expected consent, preserved the required permissions, and continued governing that identity consistently over time. Large-scale identity platforms eventually discover that application availability depends less on software quality than on thousands of independent tenant administrators making thousands of separate identity decisions that all need to remain aligned for the service to function.

## Credentials authenticate globally. Permissions authorize locally.

One architectural boundary quietly explains a surprising number of production incidents: authentication material belongs to the application object, but authorization lives with the tenant's Service Principal. That separation is what allows a single application to authenticate consistently across thousands of organizations while every tenant independently decides what the workload is actually allowed to do. It also means operational changes propagate very differently. Rotate a certificate or replace a client secret on the App Registration and every tenant consuming that application immediately begins relying on the new credential. Remove an Azure RBAC assignment, revoke Microsoft Graph application permissions, or disable the Enterprise Application inside one tenant and the outage remains completely localized — the application itself hasn't changed, and every other customer continues operating normally because their Service Principals still carry the expected authorization state. During a large SaaS incident, understanding which side of that boundary changed usually determines whether engineering spends the next hour fixing one customer or unnecessarily investigating the entire platform.

## Auditing application registrations

Production tenants accumulate application registrations much faster than most identity teams realize. Some belong to active workloads, others survive from migrations, proofs of concept, or third-party integrations that disappeared years ago. Before reviewing permissions or investigating suspicious workload activity, it's worth identifying registrations carrying multiple active credentials, expanding multi-tenant exposure, or simply existing long enough to predate today's governance standards.

```powershell
Connect-MgGraph -Scopes `
"Application.Read.All"

$applications = Get-MgApplication -All

$report = foreach ($app in $applications) {

    [PSCustomObject]@{
        DisplayName      = $app.DisplayName
        AppId            = $app.AppId
        MultiTenant      = ($app.SignInAudience -ne "AzureADMyOrg")
        Secrets          = $app.PasswordCredentials.Count
        Certificates     = $app.KeyCredentials.Count
        Created          = $app.CreatedDateTime
        CredentialExpiry = (
            $app.PasswordCredentials.EndDateTime + $app.KeyCredentials.EndDateTime |
            Sort-Object |
            Select-Object -First 1
        )
    }

}

$report |
Sort-Object Created |
Format-Table -AutoSize

# Triage roadmap:
# - Hunt for multi-credential backdoors.
# - Identify unreviewed multi-tenant footprint expansion.
# - Surface legacy registrations created before governance existed.
# - Flag credentials approaching expiry before they become outages.
# - Prioritize applications whose ownership is no longer obvious.

# Optional
# $report | Export-Csv ".\ApplicationRegistrationReview.csv" -NoTypeInformation
```

The report rarely identifies an active compromise. It usually uncovers something more dangerous: uncertainty. The application has three active credentials with no comment field or owner recorded for any of them, no one on the current team approved its multi-tenant exposure, and there's no answer to whether the workload behind it still exists. That uncertainty is exactly where identity debt accumulates.

## Enterprise Applications are where the runtime actually lives

The inventory naturally leads into Enterprise Applications, because that's where the runtime identity finally becomes visible. Application registrations describe what an application is capable of becoming; Enterprise Applications expose what it has actually become inside a tenant. RBAC assignments, provisioning state, assignment requirements, consent history, sign-in telemetry, workload Conditional Access evaluation, and service principal status all converge there. When a workload suddenly loses authorization, those tenant-specific objects almost always explain the failure long before the application registration does.

Looking only at App Registrations during an outage is roughly equivalent to debugging Active Directory by staring at schema definitions while ignoring users, groups, and domain controllers. The schema matters, but it isn't the object currently failing — the operational surface is somewhere else entirely. Experienced identity engineers eventually stop asking where the application was created and start asking which Service Principal actually processed the request that failed.

## Operational reality

Identity governance debt around SaaS integrations rarely appears overnight. It accumulates one onboarding at a time. A vendor requests Application.ReadWrite.All because it simplifies provisioning. Another asks for Directory.AccessAsUser.All to avoid implementing a narrower permission model before go-live. The project delivers successfully, the team moves on to the next deliverable, and those permissions quietly become permanent infrastructure. Months later, the approval ticket is closed and archived, the requester has moved teams, and there's no record left of whether the grant is still required or which business process depends on it. The application registration remains untouched while the Service Principal continues collecting permissions, tenant-specific exceptions, and operational significance.

That's the shift most organizations eventually have to make. Application registrations are creation templates. Service Principals are living security principals participating in production every minute of every day, and governance has to follow the runtime object, because that's where privilege changes, authorization decisions occur, and attackers establish persistence after the project team has long since disappeared.
                                             