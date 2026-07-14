---
layout: post
title: "How Mail Flow Works"
date: 2026-07-12
categories: ["Microsoft 365", "Exchange Online"]
tags: ["exchangeonline", "mailflow", "transport", "smtp"]
author: pankaj
description: "Mail flow looks simple from the mailbox side — hit send, message arrives. The pipeline underneath explains most delivery problems that don't add up otherwise."
image:
  path: /assets/img/og-default.png
  alt: "How Mail Flow Works"
---
Mail flow in Exchange Online looks simple from the mailbox side — you hit send and the message appears in someone's inbox a few seconds later — but there's a pipeline underneath that's worth understanding properly, because most of the weird delivery problems I've dug into in my own lab tenant turn out to be a misunderstanding of which stage of that pipeline actually touched the message.

## The path a message actually takes

Inbound mail destined for your tenant hits your MX record first, which points to `contoso-com.mail.protection.outlook.com`. That's Exchange Online Protection, and it's a separate service layer from the mailbox database itself. EOP does the connection-level filtering: reverse DNS checks, IP reputation, SPF/DKIM/DMARC evaluation, and spam/malware scanning through Defender for Office 365 if you're licensed for it. Only after a message clears EOP does it get handed to the transport service that actually routes it toward the recipient's mailbox, and that's the stage where transport rules (mail flow rules) get evaluated.

Outbound mail runs the same pipeline in reverse for anything leaving your tenant to the internet, and internally, mail between two mailboxes in the same tenant never actually leaves Microsoft's network — it goes mailbox-to-mailbox through the transport service without transiting the public internet, which matters if you're trying to reason about latency or about whether a security control that inspects "email" actually sees internal-to-internal mail at all.

## Connectors are where custom routing lives

If your organization has anything other than a straightforward direct-to-internet setup — a hybrid environment with on-prem Exchange, a third-party mail security gateway front-ending inbound mail, or a requirement to route outbound mail through a specific smart host — that routing logic lives in connectors, not in transport rules. Inbound connectors tell Exchange Online how to trust and accept mail from a specific source (say, an on-prem Exchange server in a hybrid deployment), and outbound connectors tell it where to send mail instead of directly to the internet. Getting connector scoping wrong is a classic mistake: a connector configured too broadly can end up trusting spoofed sender IPs as if they were legitimately from your on-prem environment, which quietly defeats your anti-spoofing controls.

## Where message trace actually helps

When someone reports "I never got that email," message trace in the Exchange admin center (or the underlying `Get-MessageTrace` cmdlet) is the tool that tells you which stage the message stopped at, rather than guessing. A message that shows as delivered in trace but never appeared in the inbox usually means a transport rule or a mailbox rule moved it somewhere, not that mail flow itself failed. A message that never shows up in trace at all means it never reached Exchange Online Protection, which points you back toward DNS, sender reputation, or the sending server's own outbound logs instead of anything inside the tenant.

Understanding mail flow as a sequence of distinct stages, rather than one black box, is what turns "email is broken" into a specific, answerable question, and that's really the whole value of learning the pipeline rather than just the admin center buttons that sit on top of it.
