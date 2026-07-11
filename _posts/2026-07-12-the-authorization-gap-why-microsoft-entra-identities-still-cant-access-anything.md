---
layout: post
title: "The Authorization Gap: Why Microsoft Entra Identities Still Can't Access Anything"
date: 2026-07-12
categories: ["Microsoft Entra", "Non-Human Identity"]
tags: ["microsoft entra", "authorization", "service principals", "managed identity", "microsoft graph", "azure rbac", "app roles", "zero trust"]
author: pankaj
---
# The Authorization Gap: Why Microsoft Entra Identities Still Can't Access Anything

The authorization gap usually surfaces during deployments rather than design reviews. A workload authenticates successfully against the Microsoft Entra token endpoint, receives an access token with the expected audience and claims, and proceeds exactly as the authentication flow intended. The first request to the target service immediately returns HTTP 403. Authentication has completed successfully, yet the workload remains functionally unusable because nothing in the token issuance process establishes whether the target resource is willing to authorize the requested operation. Engineering teams frequently spend hours decoding JWT claims, validating certificate thumbprints, reviewing OAuth scopes, and comparing token contents, even though the failure sits entirely outside the authentication pipeline. The identity platform has already completed its responsibility; the authorization decision belongs to the resource receiving the request.

This separation becomes increasingly difficult to visualize as cloud workloads span multiple Microsoft control planes. Microsoft Entra establishes identity and issues tokens, but virtually every workload enforces authorization independently. Azure Resource Manager evaluates Azure RBAC assignments before allowing management operations. SharePoint Online builds authorization from site collections, inherited permission structures, and Microsoft 365 Group ownership rather than directory role assignments. Exchange Online applies its own RBAC model together with Application Access Policies to determine which mailboxes an application may access, regardless of what Microsoft Graph permissions appear in the token. Federated Kubernetes clusters accept Microsoft Entra as an identity provider but immediately transition into Kubernetes RBAC for every authorization decision inside the cluster. Each platform consumes the authenticated identity differently, maintains its own permission model, and exposes its own operational tooling. During production incidents, this fragmented architecture often drives investigations toward Microsoft Entra because successful authentication is visible in sign-in logs, while the actual authorization engine rejecting the request remains hidden inside the downstream service.

The resulting troubleshooting pattern is remarkably consistent. Sign-in telemetry confirms successful authentication, Conditional Access evaluates as expected, token inspection shows the correct issuer, audience, and claims, and Enterprise Applications display no apparent configuration issues. Despite every authentication indicator appearing healthy, the workload remains blocked because the resource provider evaluates authorization using configuration that exists entirely outside the identity tenant. SharePoint administrators may have broken permission inheritance on a site months earlier. An Azure subscription owner may have removed an inherited RBAC assignment during subscription restructuring. Exchange administrators may have introduced a restrictive Application Access Policy limiting mailbox scope. Kubernetes namespace administrators may have modified a ClusterRoleBinding that no longer references the federated workload identity. None of these changes alter authentication behavior, yet every one of them is capable of producing an immediate service outage.

The complexity increases further when a single non-human identity spans several authorization boundaries simultaneously. A migration automation platform, for example, may authenticate once through a Managed Identity while depending on Microsoft Graph application permissions to enumerate directory objects, Azure RBAC to provision infrastructure, Key Vault data-plane permissions to retrieve certificates, SharePoint permissions to migrate document libraries, and workload-specific authorization models exposed by internal APIs. Every authorization dependency is administered independently, reviewed independently, and revoked independently. There is no central authority correlating these relationships into a single operational view. The identity object appears healthy, token acquisition continues succeeding, and service health dashboards remain green, yet removing a single Graph app role, revoking Key Vault data-plane access, or deleting a localized resource assignment is sufficient to halt the entire workload. From the perspective of Microsoft Entra, nothing has failed. From the perspective of the application, the deployment is completely offline because one authorization dependency somewhere beyond the identity boundary quietly disappeared.

## Accumulated authorization state measures actual exposure

Identity governance often stops at the directory object because identities are easy to enumerate, categorize, and review. Permissions are considerably harder because they are distributed across multiple resource providers, evaluated through different authorization engines, and rarely exposed through a single governance workflow. The result is a misleading security posture where review campaigns confirm that every Service Principal has an owner while never examining the permissions that actually determine its operational capability. An Enterprise Application with no effective permissions contributes almost nothing to the attack surface regardless of how many instances exist in the tenant. Conversely, a small collection of automation identities holding tenant-wide Microsoft Graph application permissions, privileged Azure RBAC assignments, and unrestricted access to production APIs represents a far larger operational risk despite appearing insignificant in inventory reports. Identity count measures administrative scale; accumulated authorization state measures actual exposure.

Permission accumulation rarely occurs through a single design decision. Broad Graph application permissions granted during proof-of-concept testing survive into production because removing them risks disrupting undocumented integrations. Azure RBAC assignments created during subscription migrations remain attached to automation identities long after deployment pipelines have stabilized. Application teams expose custom application roles to satisfy one integration, only for additional workloads to adopt the same roles until any change ticket to narrow them gets rejected in review because three unrelated services would break. Each change is individually rational, yet together they produce an authorization model that gradually loses any relationship to the original architectural intent. Periodic identity reviews continue reporting healthy ownership while the permission landscape expands quietly beneath them.

## Auditing Graph permissions

```powershell
# Triage roadmap:
# - Inventory active AppRole assignments
# - Surface broad Microsoft Graph application permissions
# - Isolate high-privilege authorization drift

Connect-MgGraph -Scopes "Application.Read.All","AppRoleAssignment.Read.All"

$graphSp = Get-MgServicePrincipal `
    -Filter "appId eq '00000003-0000-0000-c000-000000000000'"

$assignments = Get-MgServicePrincipalAppRoleAssignedTo `
    -ServicePrincipalId $graphSp.Id -All

$report = foreach ($assignment in $assignments) {

    $principal = Get-MgServicePrincipal `
        -ServicePrincipalId $assignment.PrincipalId `
        -ErrorAction SilentlyContinue

    $role = $graphSp.AppRoles |
        Where-Object Id -eq $assignment.AppRoleId

    [PSCustomObject]@{
        EnterpriseApplication = $principal.DisplayName
        GraphPermission        = $role.Value
        PermissionType         = "Application"
    }
}

$report |
Sort-Object EnterpriseApplication,GraphPermission |
Export-Csv ".\GraphApplicationPermissions.csv" -NoTypeInformation

$report
```

The exported inventory is most valuable when treated as the starting point of an authorization investigation rather than a compliance report. Begin with the resource that rejected the request, identify which authorization engine governs that operation, locate the assignment authorizing access to that resource, and then trace the dependency back toward the Service Principal or Managed Identity consuming it. This direction of analysis exposes broken authorization chains far more quickly than repeatedly inspecting tokens or Enterprise Application configuration. A Graph application permission may ultimately depend on administrator consent granted years earlier. A failed Key Vault request may trace back to a missing data-plane role assignment rather than an Azure RBAC issue. An internal API rejecting a request may depend on a custom application role whose assignment was removed during a routine cleanup. Following the authorization path from the resource back to the directory narrows the investigation to the configuration boundary that actually failed instead of the authentication system that merely issued the token.

Long-lived application permissions deserve the same scrutiny as privileged human identities because they frequently outlive the engineering work that justified them. Permanent Application.ReadWrite.All grants issued during migration projects, custom application roles created for temporary integrations, and service principals retaining tenant-wide permissions after vendor offboarding all become authorization artifacts with no surviving design ownership. They continue functioning because the underlying identity still authenticates successfully, not because anyone intentionally validated that the authorization remains appropriate. Governance improves when authorization boundaries are treated as versioned architectural components with explicit ownership, lifecycle controls, and periodic redesign. Once permission models begin reflecting accumulated operational history instead of deliberate architecture, the directory becomes a catalogue of identities while the real security posture is determined somewhere else.
                                      