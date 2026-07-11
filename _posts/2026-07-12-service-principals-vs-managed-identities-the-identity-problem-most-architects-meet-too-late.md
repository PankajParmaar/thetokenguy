---
layout: post
title: "Service Principals vs Managed Identities: The Identity Problem Most Architects Meet Too Late"
date: 2026-07-12
categories: ["Microsoft Entra", "Non-Human Identity"]
tags: ["microsoft entra", "managed identity", "service principal", "workload identity", "azure", "powershell"]
author: pankaj
---
# Service Principals vs Managed Identities: The Identity Problem Most Architects Meet Too Late

Most identity teams spend years tightening MFA policies, Conditional Access rules, and privileged access workflows for people while an entirely different identity estate grows almost unnoticed. Azure Functions deploy infrastructure. AKS workloads authenticate to Azure resources. Logic Apps call Microsoft Graph. Automation accounts modify subscriptions overnight. GitHub Actions publish production code. Terraform pipelines build and destroy environments without a human touching the keyboard. None of those workloads care whether it's three in the afternoon or three in the morning — they authenticate, request tokens, and make privileged API calls exactly as they were designed to.

Workload authentication is already a solved problem. The operational risk sits somewhere else entirely: credential ownership, secret distribution, forgotten service principals, undocumented application objects, and deployment pipelines that quietly assume an identity created years ago will always exist. By the time someone asks who owns a particular application credential, the original engineer has usually changed teams, the proof-of-concept has become production, and multiple automation platforms have quietly inherited the same trust relationship. That is where Service Principals and Managed Identities stop being Azure terminology and start becoming infrastructure design decisions.

## Every application already has two identities

One of the easiest ways to waste valuable incident response time is by investigating the wrong object. Engineers routinely open an App Registration because that's where the application was originally created, while every authorization failure they're chasing is happening against the tenant's Service Principal. Those two objects serve very different operational purposes, and confusing them becomes expensive under pressure.

The friction becomes obvious in multi-tenant deployments. A single application registration can project itself into dozens or hundreds of customer tenants, each represented by its own Service Principal carrying independent RBAC assignments, Graph permissions, and audit history. When an application suddenly loses access, starts receiving `403 Forbidden` responses from Microsoft Graph, or fails authorization against Azure resources, the investigation almost always belongs on the Service Principal inside the affected tenant rather than the application definition sitting somewhere else.

The same distinction matters when authentication switches from delegated to app-only flows. Delegated tokens inherit a user's security context and many authorization decisions become easier to reason about because there's a human identity attached to the request. Client credential flows remove that safety net completely — every permission evaluation lands directly on the Service Principal, making it the object that accumulates Graph application permissions, Azure RBAC assignments, and application roles over time. Miss that distinction and it's remarkably easy to spend half an hour following the wrong Object ID through Entra sign-in logs while the failing identity is sitting in a different part of the directory.

## Managed Identity removes credential ownership

A Service Principal gives an application an identity, but it doesn't remove the operational burden that comes with managing credentials. Someone still generates client secrets or certificates, stores them, rotates them, replaces them before expiry, and eventually discovers every workload still depending on them.

Managed Identity changes that relationship by moving credential ownership into the platform itself. Rather than loading a certificate into memory or retrieving a client secret from Key Vault, the workload requests a token directly from the Azure Instance Metadata Service (IMDS) endpoint. Azure validates the workload's identity, issues a short-lived OAuth token for the requested resource, and never exposes a long-lived credential to the application in the first place.

The interesting engineering work starts after the architecture diagram ends. Busy Azure Functions, containerized microservices, and AKS workloads can all generate enormous numbers of token requests if every process independently calls IMDS without understanding token lifetime. Authentication libraries cache tokens aggressively for exactly that reason, but poorly implemented retry logic, aggressively scaled container platforms, or custom authentication code can still overwhelm the metadata endpoint. Once that happens, the conversation shifts away from secret rotation and toward cache behavior, token lifetime management, retry strategies, and IMDS throttling. The credential problem disappears, but authentication itself remains another distributed dependency that deserves the same engineering discipline as storage, networking, or messaging.

## System-assigned and user-assigned aren't interchangeable

The difference between system-assigned and user-assigned Managed Identities only looks like a lifecycle decision until infrastructure starts being rebuilt automatically. A system-assigned identity lives and dies with its resource, which sounds perfectly reasonable until somebody tears down an environment during an ARM or Bicep redeployment expecting a clean rebuild. The resource disappears, the identity disappears with it, and every RBAC assignment attached to that identity disappears as well. When the deployment recreates the resource, Azure creates an entirely new identity with a different object ID — the workload starts successfully, but Key Vault, Storage, Event Hubs, or SQL immediately begin rejecting requests because every permission assignment belonged to the identity that no longer exists.

User-assigned identities solve that particular problem because the identity survives independently of the workload, allowing infrastructure to be rebuilt without recreating every RBAC assignment. That operational convenience introduces a different risk: the same identity quickly becomes attached to multiple Functions, Container Apps, virtual machines, or AKS workloads simply because reusing one identity feels easier than maintaining several. Over time, identity boundaries begin following convenience rather than workload isolation, and in Kubernetes environments lacking disciplined namespace separation and tightly scoped RBAC, compromising a single pod can expose permissions intended for entirely different applications because they all authenticate using the same shared identity. At that point, identity reuse stops being an operational shortcut and becomes a lateral movement path designed directly into the platform.

## Secrets have operational gravity

The operational life of a client secret is almost never as tidy as the architecture diagram suggests. It starts in a secure location, perhaps an Azure Key Vault or a pipeline variable, then slowly migrates into places nobody intended. Someone enables verbose logging during a failed deployment and inadvertently writes authentication details into Azure DevOps or GitHub Actions runner logs. A developer injects a secret into an application setting for a temporary test and forgets to remove it before merging the branch. Another application references Key Vault correctly but bypasses the organization's broader Zero Trust controls because the workload itself has unrestricted access to retrieve the secret from anywhere. Then comes the inevitable incident: a `.env` file lands in a public GitHub repository, automated scanners find it within minutes, and the security team begins emergency credential rotation while engineering races to identify every application, automation account, deployment pipeline, and forgotten script still depending on the compromised credential. The rotation itself is usually the easy part — the difficult part is discovering how many undocumented dependencies quietly accumulated around that secret over the previous two years. This is why Managed Identity changes the operational equation so dramatically: it doesn't just improve secret governance, it removes one of the largest sources of governance debt before it has a chance to spread across the estate.

## Workload identity is still identity

One misconception appears regularly during architecture reviews — because there isn't a human user involved, traditional identity governance somehow becomes less important. In practice the opposite happens. Service Principals, Managed Identities, and application identities continuously accumulate permissions while rarely attracting the same governance attention as user accounts. They don't change departments, submit access requests, or trigger routine access reviews, which means privilege creep quietly accumulates over months or years until workload identities become some of the most privileged security principals in the tenant without anyone intentionally designing them that way.

Conditional Access coverage for workload identities has improved, but in many environments policy coverage remains inconsistent because licensing, implementation maturity, and governance priorities are still overwhelmingly focused on human identities. Attackers understand that imbalance. A service principal holding `Application.ReadWrite.All` can create new credentials on existing applications without ever stealing a user's password. One holding `RoleManagement.ReadWrite.Directory` can manipulate privileged role assignments and establish directory persistence that survives password resets, MFA rollouts, and many traditional identity response actions. Human identities usually receive continuous scrutiny; workload identities often receive trust once during deployment and disappear into the background until an incident response investigation rediscovers them.

## Auditing workload identities

This is the report you run when an audit finding asks who owns a given application registration and the honest answer is that the original requester left the company two years ago. It surfaces applications carrying active credentials, highlights multi-tenant registrations that deserve another review, and exposes older application objects that were created long before today's governance standards existed. During incident response it also helps narrow the search space for service principals that have quietly accumulated enough privilege to become persistence mechanisms rather than application identities.

```powershell
Connect-MgGraph -Scopes `
"Application.Read.All","Directory.Read.All"

$applications = Get-MgApplication -All
$servicePrincipals = Get-MgServicePrincipal -All

$report = foreach ($app in $applications) {

    $sp = $servicePrincipals | Where-Object {
        $_.AppId -eq $app.AppId
    }

    [PSCustomObject]@{
        DisplayName           = $app.DisplayName
        AppId                 = $app.AppId
        MultiTenant           = ($app.SignInAudience -ne "AzureADMyOrg")
        CredentialCount       = ($app.PasswordCredentials.Count + $app.KeyCredentials.Count)
        OldestCredential      = ($app.PasswordCredentials.StartDateTime + $app.KeyCredentials.StartDateTime |
                                 Sort-Object |
                                 Select-Object -First 1)
        NewestExpiry          = ($app.PasswordCredentials.EndDateTime + $app.KeyCredentials.EndDateTime |
                                 Sort-Object -Descending |
                                 Select-Object -First 1)
        ServicePrincipalCount = ($servicePrincipals | Where-Object {$_.AppId -eq $app.AppId}).Count
        Created               = $app.CreatedDateTime
    }

}

$report |
Where-Object {
   $_.CredentialCount -gt 0
} |
Sort-Object Created |
Format-Table -AutoSize

# Incident response triage:
# - Hunt for applications carrying multiple active credentials.
# - Flag multi-tenant registrations that never received governance review.
# - Surface legacy application objects created before current security standards.
# - Prioritize service principals that could provide persistence after credential theft.

# Optional
# $report | Export-Csv ".\WorkloadIdentityTriage.csv" -NoTypeInformation
```

The report is only the starting point. Multiple active credentials often exist because somebody was afraid to remove the old one after a rotation. Multi-tenant application registrations deserve immediate scrutiny because they're designed to establish trust beyond a single tenant, and forgotten application objects frequently survive infrastructure migrations long after the workloads they originally supported disappeared. Those identities are attractive persistence targets because they tend to receive far less attention than privileged user accounts while often holding equally powerful Microsoft Graph permissions.

## Workload identity federation changes the architecture

Managed Identity solves credential ownership inside Azure, but most engineering organizations don't build exclusively inside Azure anymore. GitHub Actions deploys infrastructure, AWS-hosted services call Microsoft Graph, GCP workloads interact with Azure resources, and self-hosted runners span multiple cloud providers. Falling back to client secrets or certificates for those external platforms simply recreates the operational problems Managed Identity eliminated in the first place. The immediate concern isn't an expired credential interrupting a deployment — the more damaging scenario is a developer cloning a repository containing hardcoded authentication material, exposing repository secrets through an overly permissive CI/CD workflow, or unintentionally leaking credentials into build artifacts that become available far beyond the original engineering team. Once a long-lived secret escapes, it stops being an authentication mechanism and becomes a lateral movement opportunity.

Workload Identity Federation replaces that entire model with trust established through OpenID Connect. GitHub Actions, AWS IAM, and GCP workload identities present short-lived OIDC tokens that Microsoft Entra validates against a configured federated credential by checking issuer, audience, and subject claims before issuing its own access token. Nothing permanent is copied into repository secrets, deployment variables, or external secret stores because there is no client secret or certificate to distribute. The pipeline authenticates by proving where it came from rather than by presenting a credential that somebody generated months or years earlier.

That architectural shift matters because external automation has become part of the identity perimeter whether organizations acknowledge it or not. The strongest workload identity design isn't the one with the longest client secret rotation policy or the most carefully protected certificate — it's the one that removes long-lived credentials from external pipelines entirely. For GitHub Actions, AWS, and GCP integrations, Workload Identity Federation has become less of an optional feature and more of the operational baseline for eliminating one of the most persistent credential exposure paths in modern cloud infrastructure.
                                                                                                                                                                                                                                                                                                                                                                                        