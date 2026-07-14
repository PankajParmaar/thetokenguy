---
layout: post
title: "Grant Controls vs Session Controls: Where Most Conditional Access Policies Go Wrong"
date: 2026-07-12
categories: ["Microsoft Entra", "Conditional Access"]
tags: ["microsoft entra", "conditional access", "grant controls", "session controls", "defender for cloud apps", "continuous access evaluation", "powershell"]
author: pankaj
description: "Most Conditional Access designs stop at MFA, as though authentication were the final decision. What happens after sign-in is where policies actually go wrong."
image:
  path: /assets/img/og-default.png
  alt: "Grant Controls vs Session Controls: Where Most Conditional Access Policies Go Wrong"
---

One of the quickest ways to spot a Conditional Access policy that's going to cause operational pain is when the design stops asking what happens after the user successfully signs in. Most discussions end at MFA, compliant devices, or authentication strength, as though authentication were the final security decision — it isn't. Some Conditional Access decisions happen before Microsoft Entra issues an OAuth or OpenID Connect token, while others continue shaping the session long after authentication has completed. Once you separate those two control planes, a lot of confusing policy behavior suddenly becomes predictable.

## Grant controls decide whether a token is issued

Grant controls exist entirely on the authentication side of the transaction. Microsoft Entra evaluates every applicable policy before issuing an access token, combining grant requirements across all matching policies — MFA, authentication strength, compliant device checks, hybrid join requirements, and Terms of Use all participate in that decision, and if any required control isn't satisfied, token issuance simply stops and the application never sees the request. This distinction matters during troubleshooting because applications like SharePoint or Exchange Online never get an opportunity to make their own authorization decisions if Conditional Access doesn't release the token in the first place. When users report that "SharePoint blocked me," the conversation often starts in the wrong place, since more often than not SharePoint never received an authenticated request to evaluate at all.

## Session controls take over after authentication

Session controls operate in a completely different part of the lifecycle. By the time they come into play, authentication has already succeeded and the application has accepted a valid token — the objective is no longer proving identity, it's shaping how that authenticated session behaves over time. That might mean shortening interactive sign-in intervals, preventing browser persistence, handing the session to Microsoft Defender for Cloud Apps for real-time inspection, or restricting what users can do with sensitive data after access has already been granted. None of those controls strengthen authentication, because that decision has already been made; they reduce what can happen next if a valid session gets abused. A lot of policy design mistakes come from expecting session controls to compensate for weak grant controls. They never will — if unmanaged devices shouldn't receive a token, solve that before authentication finishes instead of trying to contain the session afterward.

## Continuous Access Evaluation changes the session lifecycle

Before Continuous Access Evaluation, access tokens were largely governed by time. Once a token was issued, the relying application generally trusted it until expiration, regardless of what happened to the account in the meantime. CAE changes that relationship — instead of treating token lifetime as a fixed countdown, participating Microsoft services can react to critical identity events and immediately re-evaluate whether an existing session should remain valid. The result is a much more dynamic trust model where the effective lifetime of a token depends less on its expiry timestamp and more on whether the underlying identity state has changed. It's easiest to think of CAE as replacing periodic trust validation with event-driven trust validation: the token may still have an expiry time, but it no longer guarantees the session will survive until then.

## Browser session controls deserve more attention

Most Conditional Access deployments stop evolving once MFA is working, which is usually where the more interesting work begins. Browser session controls are one of the few places where identity policy starts influencing data movement rather than authentication — preventing persistent browser sessions on shared kiosks, handing high-risk sessions to Microsoft Defender for Cloud Apps for continuous inspection, or restricting downloads from unmanaged endpoints all happen after the user has successfully authenticated. Those decisions won't stop a sign-in, but they can dramatically reduce what an attacker, or an overly curious insider, can do with an already authenticated session. They're particularly valuable in environments where blocking access outright isn't realistic, since contractors, partners, and BYOD users often still need access to corporate data, just not with the same level of freedom as a managed device sitting on the corporate estate.

## Auditing policies using session controls

As environments grow, it's easy to lose track of which policies actually enforce session restrictions. The following script inventories Conditional Access policies and highlights those configured with session controls.

```powershell
Connect-MgGraph -Scopes "Policy.Read.All"

$policies = Get-MgIdentityConditionalAccessPolicy

$report = foreach ($policy in $policies) {

    [PSCustomObject]@{
        Policy                 = $policy.DisplayName
        State                  = $policy.State
        PersistentBrowser      = $policy.SessionControls.PersistentBrowser.Mode
        SignInFrequency        = $policy.SessionControls.SignInFrequency.Value
        CloudAppSecurity       = $policy.SessionControls.CloudAppSecurity.CloudAppSecurityType
        DisableResilience      = $policy.SessionControls.DisableResilienceDefaults
    }

}

$report |
Where-Object {
   $_.PersistentBrowser -or
   $_.SignInFrequency -or
   $_.CloudAppSecurity
} |
Sort-Object Policy |
Format-Table -AutoSize

# Optional
# $report | Export-Csv ".\CASessionControlAudit.csv" -NoTypeInformation
```

Looking at this output often reveals the opposite problem people expect. Instead of finding too many session controls, many tenants discover they aren't using them at all.

## Sign-in frequency isn't a security blanket

Sign-in Frequency is one of the most misunderstood session controls. Reducing it from 90 days to 4 hours doesn't automatically make the tenant more secure — if the device is compliant, the Primary Refresh Token remains healthy, and Continuous Access Evaluation hasn't triggered, repeatedly forcing users through authentication often creates frustration without materially changing risk. Short token lifetimes also don't stop token theft: if an attacker steals a valid access token, shortening the next interactive sign-in doesn't necessarily invalidate the stolen one. Session lifetime should reflect operational risk, not security anxiety.

## Operational reality

This is where Conditional Access policies quietly become messy. Someone enables Sign-in Frequency because auditors asked about session lifetime without understanding how Primary Refresh Tokens behave. Another team rolls out Defender for Cloud Apps but never links it to Conditional Access, so every policy says "Monitor only" forever. Browser persistence gets disabled for executives because it sounded more secure, then Outlook on the web starts generating support tickets every Monday morning. Six months later the ticket lands on the identity team's queue with a title like "Conditional Access is broken," even though authentication worked exactly as designed — the confusion came from expecting post-authentication controls to solve authentication problems.

Grant controls decide whether a session starts. Session controls decide how much freedom that session gets after it starts. Mixing those two ideas is how perfectly valid Conditional Access policies end up solving the wrong problem.
                                                                                                                                                                                                                                                                                                                                                                                                                                                                