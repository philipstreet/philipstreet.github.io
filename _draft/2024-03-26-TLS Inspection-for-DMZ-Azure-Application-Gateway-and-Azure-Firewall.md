---
layout: post
title: TLS Inspection for DMZ Azure Application Gateway & Azure Firewall - certificate options, issues and limitations
---

If the title of this article isn't long enough then the rest of the blog post will make up for it, because there's a lot to cover.

I recently

## Architecture

This article covers a specific architectural scenario, which is the [Zero-trust network for web applications with Azure Firewall and Application Gateway](https://learn.microsoft.com/en-us/azure/architecture/example-scenario/gateway/application-gateway-before-azure-firewall).

![alt text](/images/2024-03-26-TLS-Inspection.png)

Architecture diagram showing the packet flow in a web app network that uses Application Gateway in front of Azure Firewall Premium.

Azure Application Gateway in front of Azure Firewall.

## Options

There are X options, according to the official Microsoft documentation:

1. Create your own self-signed certificate
1. Use the enterprise PKI to create an Intermediate CA certificate
1. Certificate auto-generation

I'll go through each option, discuss some Pros and Cons, identify some limitations, and talk about some of the issues I encountered.

### Option 1 - Create your own self-signed certificate

### Option 2 - Use the enterprise PKI to create an Intermediate CA certificate

### Option 3 - Certificate auto-generation