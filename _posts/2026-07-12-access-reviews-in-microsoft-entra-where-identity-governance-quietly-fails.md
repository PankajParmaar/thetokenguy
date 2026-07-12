---
layout: post
title: "Access Reviews in Microsoft Entra: Where Identity Governance Quietly Fails"
date: 2026-07-12
categories: ["Microsoft Entra", "Governance"]
tags: ["microsoft entra", "identity governance", "access reviews", "microsoft graph", "zero trust"]
author: pankaj
---

Large-scale identity environments frequently expose a structural reality: Access Reviews lose their efficacy not because reviewers are careless, but because the review process scales past the limit of human cognitive capacity. Monday morning arrives, Outlook fills with campaign notifications, and department managers face the task of validating hundreds of identities mapped to security groups they did not create, Microsoft 365 Groups they do not own, application roles they have never encountered, and nested Active Directory groups synchronized years earlier through Entra Connect. The path of least operational resistance becomes the blanket approval. After several cycles, the campaign shifts from an active security control into administrative theatre — the directory records a completed review, compliance dashboards reflect a green status, and the underlying permissions remain completely unvalidated against actual business intent.

The root of this friction does not lie within the review workflow itself. It stems from the fact that Microsoft Entra evaluates what currently exists in the directory, completely divorced from the historical chain of decisions that produced the assignment. By the time a reviewer receives a line item, the architectural context has vanished. A synchronized security group might nest another synchronized group originating from an on-premises organizational unit that no cloud administrator manages. That nested group might contain dynamic memberships driven by legacy HR attributes, while an application team has layered direct app-only assignments on top to resolve a past migration outage. The resulting access path is technically valid but functionally opaque. Access Reviews enumerate effective membership flawlessly, yet they cannot reconstruct the architectural workarounds, temporary exceptions, or migration shortcuts that accumulate inside a tenant over time.

This historical intent gap becomes highly visible as hybrid architectures age. Shadow groups created for synchronization survive long after the legacy applications requiring them have been decommissioned. Emergency privileges granted during tenant mergers become permanent because the original engineering context lives exclusively in a closed ticketing system. Direct resource assignments appear after administrators bypass group-based authorization to restore production access quickly, but those assignments never integrate into a defined identity lifecycle. From the reviewer's perspective, every entry looks identical because Entra surfaces the final assignment state rather than the operational timeline — the console displays a user inside a group, but the directory cannot explain whether that membership originated from a standard business request, an automated pipeline, an inherited nested relationship, or a one-off troubleshooting session that was never cleaned up.

This is precisely where operational models expect Access Reviews to solve a governance problem they were never architected to handle. Identity governance requires that entitlement context survives alongside the entitlement itself. Entra Access Packages mitigate this by ensuring approvals, request justifications, assignment policies, sponsors, and explicit expiration rules remain tightly coupled to the lifecycle state. Months later, when a review kicks off, reviewers are no longer analyzing anonymous group memberships — they are inspecting access that carries its own technical lineage. Without this structural metadata, group ownership decays rapidly. Managers transition roles, application owners leave the organization, shared mailboxes wind up configured as the primary owner of critical security groups, and synchronized objects continue granting access even when no accountable decision maker exists in the tenant.

A similar pattern degrades privileged access governance, though the associated blast radius is significantly wider. Permanent role assignments created outside Privileged Identity Management gradually blend into the background noise of the directory because they bypass activation workflows, approval chains, and time-bound enforcement mechanisms. Even within tenants running regular review cycles, audits frequently uncover highly privileged identities assigned years prior through direct directory role membership, emergency elevation during critical outages, or legacy migration scripts that predated PIM adoption. These assignments issue highly privileged tokens continuously without generating anomalous operational signals because nothing links them back to an active, bounded business requirement. Access Reviews confirm that the assignment is active; they cannot determine whether the privilege should have expired multiple projects ago.

## Operational drift occurs at the tenant edge

Operational drift usually begins outside the review campaign itself. External collaboration is a common example. Guest accounts created for vendor onboarding, consulting engagements, or joint development projects frequently survive well beyond the contractual relationship because the project lifecycle and the identity lifecycle are managed independently. The guest object remains valid, its group memberships remain intact, and any downstream application authorization continues to function exactly as designed. An Access Review surfaces the guest account months later, but without project metadata, contract dates, or sponsorship history, the reviewer has little evidence to determine whether the access represents an active business requirement or administrative residue.

Hybrid environments introduce another layer of structural friction. During Active Directory modernization, synchronized OUs and security groups are often retained to avoid disrupting dependent workloads while cloud-native authorization models are introduced incrementally. The migration completes, applications move into Microsoft Entra, and administrators gradually shift toward cloud-managed identities, yet the synchronization boundary remains unchanged because removing it risks affecting unknown dependencies. Those synchronized groups continue flowing into Entra ID, where they remain eligible for application assignments, Conditional Access targeting, and downstream authorization decisions. The directory faithfully evaluates their current membership, but the operational context explaining why the synchronization relationship still exists has usually disappeared with the migration project itself. Access Reviews expose the resulting memberships without exposing the architectural dependency that continues generating them.

The same governance debt appears across Microsoft 365 workloads. Teams created for temporary initiatives often leave behind Microsoft 365 Groups that quietly survive long after the collaboration space has been abandoned. Even after conversations stop and ownership changes, the group continues acting as the authorization boundary for SharePoint sites, Planner plans, Loop workspaces, and other connected workloads. Removing a seemingly inactive group can unexpectedly revoke data-plane access across multiple services, while leaving it untouched preserves permissions that no longer map to an active business function. The review campaign identifies the membership but provides almost no visibility into the downstream resources still trusting that identity object.

These scenarios illustrate why governance quality is determined long before an Access Review begins. Once ownership metadata, sponsorship records, and entitlement lineage have been lost, the review becomes an exercise in interpreting directory state rather than validating business intent. The directory can explain who currently has access; it cannot explain which infrastructure decision created the authorization path or whether the dependency remains operationally relevant.

## Auditing governance debt

```powershell
# Triage roadmap:
# - Locate orphan security objects
# - Map zero-owner governance risk
# - Identify groups likely to escape meaningful review
# Requires: Group.Read.All, Directory.Read.All

Connect-MgGraph -Scopes "Group.Read.All","Directory.Read.All"

$groups = Get-MgGroup -All `
    -Property Id,DisplayName,GroupTypes,SecurityEnabled

$report = foreach ($group in $groups) {

    $owners = Get-MgGroupOwner -GroupId $group.Id -ErrorAction SilentlyContinue

    if (($owners | Measure-Object).Count -eq 0) {
        [PSCustomObject]@{
            GroupName       = $group.DisplayName
            SecurityGroup   = $group.SecurityEnabled
            GroupType       = ($group.GroupTypes -join ",")
            OwnerCount      = 0
            GovernanceRisk  = "No Assigned Owner"
        }
    }
}

$report |
Sort-Object GroupName |
Export-Csv ".\GroupsWithoutOwners.csv" -NoTypeInformation

$report
```

The report generated by this script should not become another governance dashboard that attempts to measure every object equally. A tenant with tens of thousands of directory objects cannot sustain that approach, and the operational return is negligible. Investigation should begin where identity failures produce the largest authorization blast radius: Microsoft Entra administrative role assignments, app-only service principals holding application permissions, security groups protecting business-critical workloads, and Microsoft 365 Groups acting as the identity boundary for collaborative data. Each orphaned owner or undocumented assignment within these objects represents an authorization dependency that can silently outlive the workload, the project, or the administrators who originally implemented it.

Distribution lists, low-impact collaboration groups, and short-lived departmental objects certainly benefit from periodic review, but they should never consume the same operational budget as identities capable of issuing privileged tokens, granting application permissions, or controlling access to production data. Governance efforts become far more predictable when engineering teams prioritize authorization boundaries instead of object counts. A review campaign that validates one hundred high-impact identities with complete lifecycle evidence delivers substantially more value than reviewing ten thousand low-risk memberships with no surviving context.

Access Reviews are most effective when they verify decisions already enforced by lifecycle policies, entitlement workflows, and time-bound privilege assignments. Once governance depends on reviewers reconstructing forgotten infrastructure history from current directory state, the platform is no longer validating identity design — it is compensating for its absence.
                                                   