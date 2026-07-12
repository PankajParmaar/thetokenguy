---
layout: post
title: "Iron Bar — The Access Token"
hero_title: "Iron Bar &mdash; The <em>Access Token</em>"
date: 2026-07-12 00:10:00 +0530
categories: [featured]
sitemap: false
hidden: true
tags: [identity, access-tokens, history, jwt, zero-trust]
author: pankaj
description: "Face. Name. Wax seal. Passport. Smart card. JWT. Six hundred years of credential evolution — and someone is still finding a way through."

---

Sometime in the ninth century, if you stood accused and the evidence ran out, they handed you a bar of iron pulled from a fire.

You'd carry it nine steps. Then they'd bandage your hand and check back in three days. Healed cleanly? God had verified your innocence. Still a mess? Also God, but with a different verdict. The Church administered the test, interpreted the result, and recorded the outcome. It was the identity provider. The priest was just the admin.

This was called Trial by Ordeal. It was the official criminal justice system of medieval Europe for about four hundred years. It mostly worked. The priests who administered it would cool the bar for people they already trusted. Fire had nothing to do with it. Conditional access, ninth century edition. The system had a backdoor. The backdoor was called judgment. The role existed before the current team joined. It was never revoked.

A few centuries later, in a different part of Europe, a different problem. Two people, same name, same town. Same UPN, different objects — and no authoritative directory to tell them apart. One of them has debts. One of them doesn't. One is in the tenant. The other one probably should be disabled by now. Neither has a token that proves anything. The one who argues better — who knows something, has something, or is something the magistrate believes — walks away with the other person's life. The loser inherits the accumulated disasters.

The token is the iron bar. A JWT is a signed assertion — an issuer telling a resource that a subject has been verified, with a set of claims about what it's allowed to do. Three parts, base64 encoded, dot separated. The iss says who vouched for you. The aud says who should accept it. The exp says when it stops being valid.

The exp claim is where the design tension lives. An access token in Entra ID is typically valid for around sixty minutes. Sixty minutes is long enough that authentication doesn't become friction. A stolen token, replayed from a different continent, works perfectly for the duration. The token doesn't know it was stolen. The resource doesn't either. It checks the signature. It checks the expiry. Both pass.

This is where AiTM sits. The attack doesn't crack credentials. It proxies the entire authentication flow — the user authenticates through what appears to be a legitimate login page, MFA completes, the ESTSAUTH session cookie issues, and the attacker captures it in transit. MFA was satisfied. Conditional Access was satisfied. The amr claim in the token shows mfa. The sign-in logs show a successful authentication. The only signal is a SessionId that later appears from a different IP — if anyone is looking.

The industry's answer was Continuous Access Evaluation. CAE inverts the model. Instead of trusting the token until exp, the resource can reject it mid-session when something changes — account disabled, password reset, a location shift that violates policy. In CAE-capable clients, token lifetimes extend to twenty-four or twenty-eight hours precisely because revocation becomes near real-time. If you've seen AADSTS50173 — the provided grant has expired due to it being revoked — that's CAE doing its job. The token was valid. Something changed. The session ended without waiting for exp.

CAE works when both the client and the resource support it. A large portion of the application estate — third-party integrations, older line-of-business apps, anything that doesn't speak the CAE claim challenge protocol — sits outside it. When a CAE-incapable app issues an access token, the exp is the only enforcement boundary left. A stolen token with fifty minutes remaining is fifty minutes of uncontested access.

The legacy MFA stack and the Authentication Methods Policy run in parallel in most tenants that have been live for more than four years. Per-user MFA, ADAL-based authentication, the old Authenticator app registration flow — none of it is automatically replaced when the Authentication Methods Policy is enabled. Two systems, same tenant, different controls, different audit surfaces. A user registered for MFA under the legacy stack doesn't appear correctly in the Authentication Methods Activity report. A Conditional Access policy requiring a specific authentication strength evaluates against the Authentication Methods Policy. If the user's registration lives in the legacy system, the evaluation produces AADSTS50076 or AADSTS50097 depending on which app they're hitting and which authentication library it's using. The migration path is documented. Running both systems simultaneously is the default state for tenants that enabled MFA before 2022 and haven't completed the migration.

The product direction is readable. FIDO2 and passkeys replace the transmitted secret with a private key that never leaves the secure enclave. An AiTM proxy sitting between the user and login.microsoftonline.com captures nothing — the challenge-response is local, the signature is bound to the real domain, the ceremony cannot be relayed without the device that holds the key.

Passkeys work cleanly for browser-based flows against modern IdPs. A line-of-business application authenticating over LDAP against a directory that hasn't moved since 2014 is a different conversation. Most enterprise application estates span both. NIST SP 800-63-4, finalised in July 2025, mandates phishing-resistant authentication at AAL2. A significant portion of the installed base doesn't meet it. The migration runs through a hybrid state with no defined end date, and during that transition the weakest authentication method in the chain sets the actual exposure.

From an administrator's perspective, CAE, token protection in Conditional Access, sign-in risk policies, phishing-resistant authentication strengths — all of it is available. Which applications in the estate are CAE-capable. Which service principals last rotated their secrets before the current team joined. Which Conditional Access policies have exception groups added during a 2am incident and never revisited. The EU Digital Identity Wallet rollout, the UAE's March 2026 OTP elimination deadline, India's April 2026 directive — regulators are moving faster than most estate audits.

Post-quantum cryptography is a separate thread. The FIDO Alliance demoed post-quantum passkey signing at Authenticate 2025. YubiKey previewed hardware-backed post-quantum credentials. The standards are stabilising. The OAuth application grants registered in 2019 are the more immediate problem for most tenants.

Shadow admins accumulate through nested group membership. Three hops through security groups created before the current team joined, and an account ends up with Global Administrator equivalent permissions. The role assignment report shows nothing. PIM shows nothing. A recursive group membership query against the directory shows everything — permissions inherited through groups that inherited from other groups, in a chain that has compounded across several years of org changes and project assignments.

Service principals outnumber human users in most tenants that have been running for more than three years. A significant portion carry client secrets rather than certificates. A portion of those secrets haven't rotated since the application was registered. The application owner left. The team that inherited it changed twice. The secret is still valid. The permissions were scoped to Contributor on a production subscription because that is what the developer needed to get the integration working on a Friday afternoon.

The client secret problem has a cleaner answer. Workload identity federation lets a GitHub Actions workflow or an Azure DevOps pipeline authenticate to Entra without a secret — the external token issued by the CI/CD platform is exchanged directly for an Entra access token through a configured federated credential. No secret stored in a pipeline variable. No rotation schedule. No expiry that someone has to track. A misconfigured federated credential produces AADSTS70021 — no matching federated identity record found — and the pipeline fails immediately. Adoption is patchy. Teams that haven't encountered it default to client secrets because the setup is unfamiliar and the pipeline works on the first try.

Break-glass accounts sit outside PIM and Conditional Access by design — that is the point of them. They also sit outside the monitoring that would flag an impossible travel event on a standard account. The breach investigation starts there.

HRIS drift is the quiet one. A contractor offboards. The termination ticket is raised. The handoff between HR and IT breaks somewhere in the process. The Entra account stays active. The account authenticates. AADSTS50076 doesn't fire — that requires an interactive prompt, it doesn't block. The token issues. The exp ticks down. Nothing surfaces until someone runs `Get-MgUser -Filter "userType eq 'Guest'" | Where {$_.AccountEnabled -eq $true}` and finds accounts with no sign-in activity in eleven months, still sitting in the directory.

A user connects Canva to their work account in 2021. The consent prompt asks for Mail.Read and Files.ReadWrite.All. The user clicks Accept because the app needs it to function. The grant sits in the tenant under Enterprise Applications. Three years later the user is on a different team, using a different set of tools, and the OAuth grant is still active, still scoped, still presenting valid tokens to Canva's backend on every request.

Default Entra ID allows users to consent to apps requesting low-privilege permissions without admin involvement. Mail.Read across the entire mailbox qualifies as low-privilege by that definition. An attacker registers an OAuth app, makes it look credible, a user grants consent, and the app has background access to email and calendar for as long as the grant exists. The grant doesn't expire. Revocation requires the user to remember granting it.

`Get-MgOAuth2PermissionGrant -All | Where {$_.Scope -like "*Mail*"}` run against a tenant live for three or more years. Schedule an afternoon.

An organisation federates with a partner tenant for a project. The default inbound cross-tenant access settings accept MFA claims from the external tenant. The partner tenant's MFA posture is unknown — what authentication methods they have enabled, whether their Conditional Access policies are enforced, whether they completed the Authentication Methods migration. The amr claim in the inbound token says mfa. Entra accepts it. The session proceeds. Cross-tenant access settings allow inbound trust to be scoped — accepting MFA claims only from specific tenants, or requiring re-evaluation against the local tenant's policies. Most tenants that set up B2B federation for a project deadline haven't revisited the trust settings since.

Trace identity back far enough and you arrive at a lookup table. X.500, the standard that eventually became the conceptual backbone of Active Directory, was modelled on directory services. A name, some attributes, a way to find things. The Yellow Pages had entries. X.500 had entries. Active Directory has entries. Entra ID has entries. The technology reinvents itself every decade and the underlying idea sits there unchanged, patient as a monk with a hot iron bar, waiting for someone to rediscover it and call it innovation.

I've spent fifteen years inside this. I once watched a very confident administrator run a full sync on a live production tenant instead of a delta. Not because they didn't know the difference. What followed was the kind of meeting where a senior stakeholder says *"I want to step back a bit"* — which is corporate language for *"I need a moment to not be visibly associated with what is currently happening."*

We automated the ordeal. We forgot the judgment.
