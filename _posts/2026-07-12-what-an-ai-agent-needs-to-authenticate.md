---
layout: post
title: "What an AI Agent Needs to Authenticate"
date: 2026-07-12
categories: ["AI & Identity", "AI Agents & Access"]
tags: ["ai-agents", "authentication", "identity", "oauth"]
author: pankaj
description: "Wiring an autonomous agent into internal APIs exposes how much of OAuth, SAML, and OIDC assumed a human sits at a browser proving who they actually are."
image:
  path: /assets/img/og-default.png
  alt: "What an AI Agent Needs to Authenticate"
---
I spent an afternoon in my lab last week trying to wire an autonomous agent into a set of internal APIs, and the exercise reminded me how much of our authentication tooling was never designed with this pattern in mind. We built OAuth, SAML, and OIDC around a fairly clean split: a human sits at a browser, proves who they are with a password or a passkey, and gets a token. An AI agent doesn't sit anywhere. It runs as a process, sometimes spawning sub-processes, sometimes calling out to other agents, and it needs to authenticate without a human present to click "allow" every single time.

## The credential problem

The first thing you notice is that an agent needs a credential that survives longer than a login session but shorter than a service account password left with a five-year expiry because whoever set it up moved teams before the rotation ticket ever got filed. Client credentials grants work for this at a basic level - the agent has a client ID and secret, it requests a token, it gets one scoped to whatever the authorization server allows. But that model assumes a static, known caller. An agent that spins up dynamically inside a container, maybe as part of a CI pipeline or an orchestration framework like a multi-agent system, doesn't always have a stable identity to hang a client secret off of. This is where workload identity federation earns its keep - letting the agent prove its identity through a platform-native attestation (a Kubernetes service account token, an Azure managed identity, an AWS instance profile) rather than a secret sitting in an environment variable.

## Delegation is the hard part

The harder question isn't how the agent proves it's the agent - it's how the agent proves it's acting on behalf of a specific user, and only within that user's actual permissions. If I ask an agent to summarize my inbox, the agent needs a token that reflects my mailbox access, not the service's full application-level access to every mailbox in the tenant. OAuth's token exchange grant (RFC 8693) exists for exactly this, letting a service swap its own token for one scoped down to a specific user context. In practice I see very few implementations actually doing this correctly - most agent frameworks I've tested in my lab default to using a single service principal with broad application permissions, then relying on application-layer logic to filter what the user is allowed to see. That's not delegation, that's the agent deciding on its own what the boundary should be, and it's the same trust-the-app-layer mistake we made with early SharePoint and Exchange integrations.

## Where this breaks in practice

When I traced a sample agent flow with a proxy sitting between the agent and the identity provider, I noticed the token it received had `openid profile email` scopes and nothing else - meaning the downstream API had no idea what the agent was actually authorized to touch. It fell back to an internal allowlist. That's a common pattern and it's fragile: the allowlist lives in application code, not in the identity layer, so an audit of "what can this agent do" requires reading source code instead of reading a token or a policy document. Multi-agent setups make this worse, because agent A might call agent B, and if agent B doesn't validate that the incoming call actually carries a legitimate, scoped credential (rather than just trusting network position inside the same VPC), you've built a system where compromising any one agent gives you the reach of all of them.

The practical takeaway from a few weekends of poking at this is that agent authentication isn't a new protocol problem, it's an old delegation problem wearing a new coat, and the frameworks that get it right are the ones borrowing directly from OAuth token exchange and workload identity rather than inventing bespo