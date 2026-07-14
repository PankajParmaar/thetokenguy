---
layout: post
title: "Pass-the-Hash"
date: 2026-07-12
categories: ["Microsoft Defender", "Identity Defence"]
tags: ["pass the hash", "ntlm", "credential theft"]
author: pankaj
description: "Pass-the-hash has been known for over a decade and still works in plenty of environments — proof of how hard NTLM is to retire, not just carelessness."
image:
  path: /assets/img/og-default.png
  alt: "Pass-the-Hash"
---
Pass-the-hash has been a known technique for well over a decade at this point, and it still works in a lot of environments today, which tells you more about how hard NTLM is to fully retire from a real enterprise than it does about anyone being careless.

## The mechanics, briefly

NTLM authentication doesn't require the plaintext password, it authenticates based on a hash derived from the password. That design choice, made for reasons that made sense decades ago, means that if an attacker can extract the NTLM hash from a compromised machine's memory, typically from the LSASS process, they can present that hash directly to authenticate to other systems without ever needing to know or crack the actual password. The hash functions as the credential itself. That's the whole technique in one sentence, and it's exactly why it's remained viable for so long: it doesn't exploit a bug, it exploits how NTLM was designed to work in the first place.

Kerberos, by contrast, isn't inherently immune to the same class of problem, it has its own equivalent in pass-the-ticket, where a stolen Kerberos ticket gets reused rather than a password hash, but the mechanics and detection signatures differ enough that they're usually discussed separately.

## Why it's still around

The technical fix for pass-the-hash has existed for years: disable NTLM entirely and force Kerberos-only authentication, or at minimum enforce Credential Guard to prevent LSASS from ever holding extractable hash material in the first place. In practice, full NTLM removal breaks legitimate things in most mature environments, older applications with hardcoded NTLM dependencies, certain printer and scanner integrations, some legacy line-of-business software that was never updated to support Kerberos cleanly. That's why most orgs run a mitigation posture rather than a full removal: Credential Guard where hardware supports it, restricting local admin account reuse across machines through something like LAPS, and NTLM auditing turned on to at least know how much NTLM traffic is still happening before trying to kill it.

## What detection actually looks like

Defender for Identity flags pass-the-hash through the mismatch between how the authentication looks and what a genuine interactive logon should look like, an NTLM authentication event that doesn't correspond to any preceding interactive or network logon consistent with normal use for that account. Defender for Endpoint contributes visibility into the credential theft side directly, since ASR rules that block credential stealing from LSASS memory target exactly the extraction step that makes the whole technique possible in the first place, which is why that particular ASR rule tends to get prioritized early in a hardening rollout.

## The practical takeaway

The single highest-leverage mitigation, more than any detection rule, is breaking local admin password reuse across machines. Pass-the-hash is devastating specifically because a stolen hash for a local admin account often works identically on dozens of other machines that share the same local admin credential, turning one compromised workstation into lateral access across an entire fleet. Tools like LAPS that randomize and rotate local admin passwords per machine don't stop hash theft, but they collapse the blast radius down to the single machine the hash was actually stolen from, which in a real incident is the difference that matters most.
