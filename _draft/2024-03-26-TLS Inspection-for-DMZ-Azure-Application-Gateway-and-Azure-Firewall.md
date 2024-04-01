---
layout: post
title: TLS Inspection for DMZ Azure Application Gateway & Azure Firewall - certificate options, issues and limitations
---

If the title of this article isn't long enough then the rest of the blog post will make up for it, because there's a lot to cover.

I recently worked with a customer that wanted to deploy a conventional DMZ environment in Azure using Azure Application Gateways and Azure Firewall, leveraging the various security features of Azure Firewall Premium SKU but specifically TLS Inspection for in-board HTTP(S) traffic.

This article describes how TLS Inspection can be configured and my challenges with its implementation in an Enterprise environment.

## Architecture

We'll be using a specific architectural scenario, which is the [Zero-trust network for web applications with Azure Firewall and Application Gateway](https://learn.microsoft.com/en-us/azure/architecture/example-scenario/gateway/application-gateway-before-azure-firewall), where the Azure Application Gateway sits in front of the Azure Firewall. The HTTP(S) traffic passes through those devices before getting to the backend services (API Management, App Service etc).

![Architecture diagram showing the packet flow in a web app network that uses Application Gateway in front of Azure Firewall Premium.](/images/2024-03-26-TLS-Inspection.png)

When looking at the end-to-end TLS connection between the user and the backend service, the following occurs:

- The user's browser establishes a TLS connection to the Azure Application Gateway, which has a certificate configured as part of its HTTP(S) Listener. Termination of the TLS connection on the Azure Application Gateway allows it to perform Web Application Firewall (WAF) traffic inspection.
- The Azure Application Gateway creates a new TLS connection to the "backend" via the Azure Firewall. Termination of the TLS connection on the Azure Firewall allows it to perform TLS Inspection.
- The Azure Firewall creates a new TLS connection to the backend service.

To facilitate this, the Azure Application Gateway and Azure Firewall have to establish a trusted TLS connection, where the Azure Firewall

The certificate used by Azure Firewall for TLS Inspection has [very specific requirements](https://learn.microsoft.com/en-us/azure/firewall/premium-certificates#intermediate-ca-certificate-requirements). More on this later...

## Options

There are 3 options when configuring a certificate for TLS Inspection, according to the official Microsoft documentation [Azure Firewall Premium Certificates](https://learn.microsoft.com/en-us/azure/firewall/premium-certificates#configure-a-certificate-in-your-policy):

1. Create your own self-signed certificate
1. Use your corporate PKI to create an Intermediate CA certificate
1. Certificate auto-generation

I'll go through each option, discuss some Pros and Cons, identify some limitations, and talk about some of the issues I encountered.

### Option 1 - Create your own self-signed certificate

For Dev/Test environments, Microsoft suggest [creating a self-signed sertificate](https://learn.microsoft.com/en-us/azure/firewall/premium-certificates#create-your-own-self-signed-ca-certificate) using OpenSSL and the provided config file with either the Bash or PowerShell script to generate root CA and intermediate CA certificates that you can then use on the Azure Application Gateway and AzureFirewall resources (respectively).

#### OpenSSL command-line?! What about Terraform?

I'm glad you asked, because I spent a fair amount of time on this.

First of all, I tried using the [azurerm_key_vault_certificate](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/key_vault_certificate) but eventually discovered that the azurerm provider - or rather the Azure SDK - does not support the *Basic_Constraints* attribute required for the certificate.

"Don't worry!", I hear you cry, "You can use the Hashicorp TLS provider instead."

Yes and no... The [self_signed_cert](https://registry.terraform.io/providers/hashicorp/tls/latest/docs/resources/self_signed_cert) and [locally_signed_cert]() resources do have a *is_ca_certificate* attribute, BUT they do not also support the required *pathlength* attribute. In fact, there is an outstanding, and slightly confusing, enhancement request to [add max_path_length in tls_locally_signed_cert](https://github.com/hashicorp/terraform-provider-tls/issues/296).

**Summary:**
PROs:

- Good for Dev/Test environments
- Good if you're happy managing private keys and certificate lifecycle from the command-line using OpenSSL

CONS:

- Not a fully integrated IaC experience

### Option 2 - Use your corporate PKI to create an Intermediate CA certificate

For Production environments, Microsoft recommends [deploying a certificate issued from your corporate PKI](https://learn.microsoft.com/en-us/azure/firewall/premium-deploy-certificates-enterprise-ca). That is fine, so long as it complies with [Microsoft's certificate requirements](https://learn.microsoft.com/en-us/azure/firewall/premium-certificates#intermediate-ca-certificate-requirements) and your corporate security standards.

The latter was an issue for the customer I was working with because they DO NOT issue Intermediate CA certificates directly from their roor CA. In fact, this was the certificate chain that was initially issued to the DMZ Azure Firewall;

(TODO Image of 4-level certificate  chain)

**Summary:**

### Option 3 - Certificate auto-generation
