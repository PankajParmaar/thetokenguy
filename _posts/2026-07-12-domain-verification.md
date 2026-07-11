---
layout: post
title: "Domain Verification"
date: 2026-07-12
categories: ["Microsoft 365", "Tenant Administration"]
tags: ["microsoft365", "domain", "dns", "tenant"]
author: pankaj
---
Adding a custom domain to a Microsoft 365 tenant looks like a five-minute task on paper: paste a TXT record, wait, click verify. In practice it's where a surprising number of setups stall out, mostly because the DNS side of the house and the Microsoft 365 side are owned by different teams — IT holds the tenant, a web host or registrar admin holds the zone file — and the TXT record request sits in someone's inbox until verification fails for the third time and it finally gets escalated.

## What verification actually proves

When you add a domain in the Microsoft 365 admin center or Entra ID, Microsoft generates a unique TXT record (something like `MS=ms12345678`) and asks you to publish it at your DNS provider under that domain's apex. The whole point of this step is proving domain ownership — that you control DNS for `contoso.com`, not just that you typed the name into a form. Until that TXT record resolves and Microsoft's verification job picks it up, the domain sits in an unverified state and you can't use it for UPNs, mail routing, or anything else tenant-facing.

The friction usually isn't the concept, it's the mechanics. DNS propagation isn't instant, registrar UIs differ wildly in how they let you add a bare TXT record at the root versus a subdomain, and some registrars silently append their own domain suffix to whatever you type, turning your intended record into something Microsoft never sees. If verification keeps failing, checking the actual published record with a tool like `nslookup -type=TXT contoso.com` from outside your network is faster than staring at the admin center retry button.

## The record set you actually need afterward

Verification is just step one. Once the domain is verified, Microsoft 365 wants a specific set of DNS records to make mail, autodiscover, and federation actually work:

- MX record pointing to `contoso-com.mail.protection.outlook.com` for Exchange Online mail flow
- CNAME `autodiscover.contoso.com` pointing to `autodiscover.outlook.com` so Outlook clients can self-configure
- TXT record for SPF (`v=spf1 include:spf.protection.outlook.com -all`) so receiving mail servers trust your outbound mail
- CNAME records for `sip` and `lyncdiscover` if you're doing any Teams/Skype federation with on-prem components
- DKIM CNAME records once you enable DKIM signing in Defender for Office 365, which you should, since SPF alone doesn't survive forwarding

Missing any of these doesn't block domain verification itself, but it produces the classic symptom of "the domain shows as verified, mail is a mess." I've seen setups where MX was fine but SPF was never added, and every outbound message from that domain landed in spam at the receiving end until someone went back and audited the record set against Microsoft's documented requirements.

## Multiple domains and primary UPN suffix

Once you're past one domain, decide which one is the primary UPN suffix for user accounts, because that decision affects sign-in experience and is annoying, though not impossible, to change once thousands of users exist. If you're running a lab or small tenant with several domains for testing SPF/DKIM behavior or multi-brand mail flow, keep a running note of which domain owns which DNS zone, because six months later you will not remember why `sub.contoso.com` has a CNAME pointing somewhere unexpected.

Domain verification is a small technical gate, but it's also the first place where DNS hygiene and tenant ad