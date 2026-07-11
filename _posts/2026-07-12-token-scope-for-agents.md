---
layout: post
title: "Token Scope for Agents"
date: 2026-07-12
categories: ["AI & Identity", "AI Agents & Access"]
tags: ["ai-agents", "oauth-scopes", "token-design", "access-control"]
author: pankaj
---
The first token I ever pulled out of an agent's request headers in my home lab had a scope string that read `https://graph.microsoft.com/.default`. That single scope grants whatever the app registration itself was consented for, which in the test tenant I'd built happened to include Mail.Read, Files.ReadWrite.All, and User.Read.All. The agent's actual job was to read three specific calendar events. That gap between what a token allows and what a task requires is the entire scope problem in one screenshot, and it's the reason token scoping for AI agents deserves more attention than it usually gets.

## Scopes were built for apps, not tasks

OAuth scopes were designed around the idea of a relatively fixed application requesting a relatively fixed set of permissions once, at consent time, and then using those permissions repeatedly for the life of the grant. An agent doesn't work that way. It might need Files.Read for one step of a task and nothing at all for the next nine steps, and then suddenly need Mail.Send because the task turned into "draft this email for approval." Static scope grants force a choice between requesting broad scopes up front (so nothing breaks mid-task) or requesting narrow scopes and having the agent silently fail or, worse, prompt for re-consent in a loop the user doesn't understand. Most implementations I've looked at choose broad, because narrow is annoying to build and support.

## Down-scoping and token exchange

The better pattern, and one that's finally showing up in production tooling, is requesting a broad-but-bounded token at the session level and then down-scoping it per call using RFC 8693 token exchange or a similar narrowing mechanism, so the credential that actually reaches a downstream API carries only the scope that specific call needs. This matters because scope is often the only signal a downstream resource server has for deciding what a caller can do - if the token says Files.ReadWrite.All, a resource server has no way to know the agent's current task only justified Files.Read on a single folder, unless someone built a separate authorization layer to enforce that on top of OAuth, which most teams haven't.

## What actually goes wrong

When I logged the scopes across a chain of three agents (an orchestrator, a retrieval agent, and a summarizer) I found all three carrying an identical token, minted once and passed down unchanged. That means a bug or a compromise anywhere in that chain has the reach of the orchestrator's full scope set, not the narrower scope the summarizer actually needed. This is the classic "confused deputy" pattern showing up again, except now it's confused deputies calling confused deputies. It also creates an audit problem: if every hop uses the same token, your access logs can't distinguish which agent in the chain actually made a given call, because the bearer credential looks identical at every hop.

Scoping tokens tightly per agent and per task adds real engineering overhead - it means minting more tokens, managing more token lifecycles, and building resource servers that actually enforce scope rather than treating any valid token as sufficient. But the alternative is a system where the blast radius of one compromised agent step is defined by the broadest scope anyone ever requested for convenience, which is not a design decision anyone would choose if they saw it written down plainly instead of buried in an app registration screen.
