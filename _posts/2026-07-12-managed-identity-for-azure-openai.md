---
layout: post
title: "Managed Identity for Azure OpenAI"
date: 2026-07-12
categories: ["AI & Identity", "Identity in AI Pipelines"]
tags: ["managed-identity", "azure-openai", "azure", "identity"]
author: pankaj
---
I rebuilt a small Azure OpenAI test setup in my lab twice - once with an API key sitting in an environment variable, and once with a system-assigned managed identity handling authentication instead - specifically to see how different the operational experience actually was, beyond the security talking points. The difference was smaller in terms of code changes than I expected and larger in terms of what I stopped having to worry about, which is roughly the pattern I'd expect managed identity to follow for any Azure-native service.

## What changes when you switch

With an API key, the credential lives wherever the application config puts it - an environment variable, a Key Vault reference the app has to fetch and cache, sometimes (against better judgment) a hardcoded string during early prototyping that someone forgets to remove before the first commit. That key is a bearer credential in the fullest sense: whoever has it can call the Azure OpenAI resource, and Azure has no way to distinguish a legitimate call to the API from a copy-pasted key that leaked in a support ticket six months ago. With a system-assigned managed identity, the calling application's identity is tied directly to the Azure resource it runs on - a function app, a VM, an App Service instance - and it requests a token from Azure AD using a platform-issued attestation that never leaves the Azure control plane in a form a human or an external process could intercept. In my rebuilt test setup, removing the API key entirely and granting the function app's managed identity the Cognitive Services OpenAI User role was enough to keep everything working, and it meant there was no secret left in the deployment for me to accidentally leak.

## Where the role assignment still needs care

Azure's built-in roles for OpenAI access - Cognitive Services OpenAI User for inference calls, Cognitive Services OpenAI Contributor for management operations - are reasonably well scoped compared to some of Azure's broader built-in roles, but I still ran into the familiar trap of assigning at the subscription or resource group level because it was faster during setup than scoping the role assignment to the specific Azure OpenAI resource the workload actually needed. That's the same mistake as any over-broad role assignment, just wearing an AI-shaped label - the managed identity solved the credential-storage problem cleanly, but scoping the actual role assignment tightly is still a separate task that has to be done deliberately, and it's easy to skip when everything's already working at the broader scope.

## Multi-resource pipelines complicate the picture

A pipeline that only talks to Azure OpenAI is the easy case. Most real AI pipelines also need to reach a vector store, a storage account for source documents, and sometimes an external API the managed identity model doesn't cover at all, since managed identity only works cleanly within Azure's own ecosystem. I hit this directly building a retrieval pipeline that needed both Azure OpenAI and Azure AI Search - the managed identity approach worked cleanly for both once I set up the correct role assignments on each resource separately, but it took deliberately checking Azure AI Search's specific data plane RBAC roles rather than assuming the same role names or a shared identity setup would just carry over, because the two services don't share a permission model even though they're both frequently used together in the same pipeline.

Switching to managed identity for Azure OpenAI workloads removes an entire category of credential-leak risk almost for free once it's configured, and the actual engineering lift is smaller than it looks from the documentation - but it doesn't replace the separate discipline of scoping each role assignment tightly to the specific resource a given workload needs, which is where most of the real permission sprawl in Azure AI pipelines still tends to hide.
