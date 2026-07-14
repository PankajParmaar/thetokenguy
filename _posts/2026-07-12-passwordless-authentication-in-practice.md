---
layout: post
title: "Passwordless Authentication in Practice"
date: 2026-07-12
categories: ["Microsoft Entra", "Authentication"]
tags: ["microsoft entra", "passwordless", "windows hello for business", "fido2", "microsoft authenticator", "passkeys", "authentication", "powershell"]
author: pankaj
description: "Let's go passwordless means something different in every mixed environment. One strategy built from several technologies, not a single standalone feature."
image:
  path: /assets/img/og-default.png
  alt: "Passwordless Authentication in Practice"
---

"Let's go passwordless" sounds straightforward until someone asks what that actually means in a mixed environment. For some users it means signing in with Microsoft Authenticator, for others it's Windows Hello for Business, privileged administrators might use FIDO2 security keys, and frontline workers often end up relying on passkeys stored on their phones. Passwordless isn't a single feature in Microsoft Entra — it's an authentication strategy built from several technologies that all remove the password from the normal sign-in flow, and each one carries its own deployment considerations.

## Passwordless isn't password-free

This catches a lot of people the first time they deploy it: removing passwords from the sign-in experience doesn't necessarily remove passwords from the identity itself. Every Entra user account still has a password unless you've built a completely different identity system around it — the difference is that users don't type it during normal authentication. That distinction matters because password hash synchronization, account recovery, legacy applications, and some identity lifecycle processes may still depend on passwords behind the scenes even after the front-door experience changes. Passwordless changes how users authenticate; it doesn't magically erase every password dependency sitting quietly in the rest of your environment.

## The three methods you'll encounter most often

Microsoft Entra supports several passwordless options, but most deployments end up revolving around three. With Microsoft Authenticator, the user signs in with their username and confirms the request using Number Matching inside the app — the approval itself isn't really the interesting part, since Number Matching, application context, and location details are what turned Authenticator push notifications from something users blindly approved into something they can actually verify before tapping. For most organizations this is the easiest place to begin, since it doesn't require new hardware or major changes to endpoint management.

Windows Hello for Business replaces passwords with device-bound credentials protected by the TPM, and users authenticate with face recognition, fingerprint, or PIN. One detail that's often misunderstood is the PIN itself — it isn't your credential, it never leaves the device, isn't transmitted to Microsoft Entra, and isn't validated by a server. Its only job is to unlock the private key that's securely stored in the TPM, which means a stolen PIN without the device is useless, and a stolen device without the PIN or biometric is equally useless. Once that distinction clicks, Windows Hello starts making a lot more sense as a design rather than just another sign-in option.

A FIDO2 security key stores a cryptographic key pair inside dedicated hardware, and the private key never leaves the device. Even if a user lands on a convincing phishing page, the authentication request fails because the security key validates the website before responding to the cryptographic challenge — if the origin doesn't match, nothing gets signed. That's why FIDO2 is classified as phishing-resistant authentication rather than simply another MFA method sitting on the same spectrum as SMS or OTP.

## What actually changes

The biggest shift isn't that the password disappears from the sign-in screen — it's that the password stops being the credential the service actually relies on. Password-based authentication depends on a shared secret, where both the user and the identity provider ultimately depend on the same piece of information staying secret indefinitely. Passwordless methods replace that model with asymmetric cryptography: the user's device or security key holds the private key, Microsoft Entra stores the corresponding public key, and every sign-in becomes a challenge-response operation. Because the private key never leaves the device, there's nothing equivalent to a reusable password for an attacker to capture in the first place.

## Auditing passwordless registrations

One of the first questions during a rollout is usually who actually has a passwordless method registered. The script below connects to Microsoft Graph, inspects every user, and produces a simple inventory of registered authentication methods.

```powershell
Connect-MgGraph -Scopes `
"User.Read.All","UserAuthenticationMethod.Read.All"

$report = foreach ($user in Get-MgUser -All) {

    $methods = Get-MgUserAuthenticationMethod -UserId $user.Id

    [PSCustomObject]@{
        DisplayName          = $user.DisplayName
        UserPrincipalName    = $user.UserPrincipalName
        Authenticator        = ($methods.AdditionalProperties.'@odata.type' -contains "#microsoft.graph.microsoftAuthenticatorAuthenticationMethod")
        WindowsHello         = ($methods.AdditionalProperties.'@odata.type' -contains "#microsoft.graph.windowsHelloForBusinessAuthenticationMethod")
        FIDO2Key             = ($methods.AdditionalProperties.'@odata.type' -contains "#microsoft.graph.fido2AuthenticationMethod")
        Phone                = ($methods.AdditionalProperties.'@odata.type' -contains "#microsoft.graph.phoneAuthenticationMethod")
        TemporaryAccessPass  = ($methods.AdditionalProperties.'@odata.type' -contains "#microsoft.graph.temporaryAccessPassAuthenticationMethod")
    }
}

$report |
Sort-Object DisplayName |
Format-Table -AutoSize

# Optional
# $report | Export-Csv .\PasswordlessInventory.csv -NoTypeInformation
```

This report won't tell you whether a rollout is ready, but it immediately highlights users who are still relying on phone-based methods or haven't enrolled anything beyond a password.

## Operational dependencies

Most passwordless rollouts don't fail because of cryptography — they fail because of configuration drift. Before enabling anything, verify that Combined Security Information Registration is already enabled, since mixing the legacy MFA registration experience with the newer registration flow almost always creates unnecessary confusion for end users. If you're deploying FIDO2 security keys, decide up front whether you're allowing every certified key or enforcing an AAGUID allow list, because it's much easier to make that call before different teams start buying different hardware independently. Temporary Access Pass should also be part of the rollout plan — without it, recovering users who lose both their phone and security key quickly turns into a helpdesk exercise instead of a controlled recovery process. Finally, identify applications that still expect passwords or Basic Authentication, since those dependencies rarely appear in architecture diagrams and usually surface instead on the first Monday after the rollout goes live.

## Common traps

Watch out for broken hybrid device registration silently preventing Windows Hello provisioning, users falling back to SMS because the old method was never removed, FIDO2 deployments where nobody agreed on an AAGUID policy until after hundreds of keys had already been purchased, or helpdesk calls that spiral because Temporary Access Pass was never planned for. Don't forget the break-glass accounts either — passwordless policies are easy to write and surprisingly easy to apply by accident to the very accounts you'll need most when everything else goes sideways.

Attackers have spent decades building tools to steal passwords because passwords are predictable targets, and every successful passwordless deployment removes one more credential from that equation. The interesting engineering work starts when you discover which systems in your environment still quietly assume a password will always exist.
                                                                                                                                                                                                                                                                             