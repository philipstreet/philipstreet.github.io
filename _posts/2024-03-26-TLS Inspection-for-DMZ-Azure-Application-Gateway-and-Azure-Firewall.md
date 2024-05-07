---
layout: post
title: TLS Inspection for DMZ Azure Application Gateway & Azure Firewall - certificate options, issues and limitations
tags: Azure, Azure Firewall, Application Gateway, certificates, TLS Inspection
image: 2024-03-26-cert-chain.png
---
If the title of this article isn't long enough then the rest of the blog post will make up for it, because there's a lot to cover.

I recently worked with a customer that wanted to deploy a conventional DMZ environment architecture in Azure using Azure Application Gateways and Azure Firewall, leveraging the various security features of Azure Firewall Premium SKU but specifically TLS Inspection for in-bound HTTP(S) traffic.

This article specifically describes my challenges with configuring TLS Inspection in an enterprise environment.

## Architecture

We'll be using a specific architectural scenario, which is the [Zero-trust network for web applications with Azure Firewall and Application Gateway](https://learn.microsoft.com/en-us/azure/architecture/example-scenario/gateway/application-gateway-before-azure-firewall), where the Azure Application Gateway sits in front of the Azure Firewall. The HTTP(S) traffic passes through those devices before getting to the backend services (API Management, App Service etc).

![Architecture diagram showing the packet flow in a web app network that uses Application Gateway in front of Azure Firewall Premium.](/images/2024-03-26-TLS-Inspection.png){:.centered}

When looking at the end-to-end TLS connection between the user and the backend service, the following occurs:

- The user's browser establishes a TLS connection to the Azure Application Gateway, which has a certificate configured as part of its HTTP(S) Listener. Termination of the TLS connection on the Azure Application Gateway allows it to perform Web Application Firewall (WAF) traffic inspection.
- The Azure Application Gateway creates a new TLS connection to the "backend" via the Azure Firewall. Termination of the TLS connection on the Azure Firewall allows it to perform TLS Inspection.
- The Azure Firewall creates a new TLS connection to the backend service.

To decrypt and inspect TLS traffic, Azure Firewall Premium dynamically generates certificates and presents itself to the Application Gateway as the web server. The Azure Firewall has to use an CA certificate to sign the certificates that it generates.

The certificate used by Azure Firewall for TLS Inspection has [very specific requirements](https://learn.microsoft.com/en-us/azure/firewall/premium-certificates#intermediate-ca-certificate-requirements).

## Options

There are 3 options when configuring Azure Firewall with a CA certificate to be used for TLS Inspection, according to the official Microsoft documentation [Azure Firewall Premium Certificates](https://learn.microsoft.com/en-us/azure/firewall/premium-certificates#configure-a-certificate-in-your-policy):

1. Create your own self-signed certificate
2. Use your enterprise PKI to create an Intermediate CA certificate
3. Certificate auto-generation (via the Azure Portal)

I'll go through each option, discuss some Pros and Cons, identify some limitations, and talk about some of the issues I encountered.

### Option 1 - Create your own self-signed certificate

For Dev/Test environments, Microsoft suggest [creating a self-signed sertificate](https://learn.microsoft.com/en-us/azure/firewall/premium-certificates#create-your-own-self-signed-ca-certificate) using OpenSSL and their provided config file with either the Bash or PowerShell script to generate root CA and intermediate CA certificates that you can then use on the Azure Application Gateway and AzureFirewall resources (respectively).

#### OpenSSL command-line?! What about Terraform?

I'm glad you asked, because I spent a fair amount of time on this.

First of all, I tried using the [azurerm_key_vault_certificate](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/key_vault_certificate) resource but eventually discovered that the azurerm provider - or rather the Azure SDK - does not support the *Basic_Constraints* attribute required for the certificate, which is used to set the *is_ca_certitifcate* and *pathlength* properties. These certificate attribute properties are required to allow the Azure Firewall to issue temporary certificates for the TLS connection created by the Application Gateway.

**After engaging Microsoft Support, they provided confirmation from the Azure Key Vault product group that there are *currently* no plans to support these features in the Azure SDK.**

"Don't worry!", I hear you cry, "You can use the Hashicorp TLS provider instead."

Yes and no... The [self_signed_cert](https://registry.terraform.io/providers/hashicorp/tls/latest/docs/resources/self_signed_cert) and [locally_signed_cert](https://registry.terraform.io/providers/hashicorp/tls/latest/docs/resources/locally_signed_cert) resources do have a *is_ca_certificate* property, BUT they do not support the required *pathlength* property. In fact, there is an outstanding, and slightly confusing, enhancement request to [add max_path_length in tls_locally_signed_cert](https://github.com/hashicorp/terraform-provider-tls/issues/296) to Hashicorp's TLS provider.

**Summary:**

PROs:

- Good for Dev/Test environments
- Good if you're happy managing private keys and certificate lifecycle from the command-line using OpenSSL

CONS:

- Not a fully integrated IaC experience
- Requires additional automation to manage the certificate lifecycle
- Not recommended for Production environments

Issues:

- Neither the [azurerm_key_vault_certificate](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/key_vault_certificate), [self_signed_cert](https://registry.terraform.io/providers/hashicorp/tls/latest/docs/resources/self_signed_cert) or [locally_signed_cert](https://registry.terraform.io/providers/hashicorp/tls/latest/docs/resources/locally_signed_cert) resources *currently* support creating the required intermediate CA certificate

### Option 2 - Use your enterprise PKI to create an Intermediate CA certificate

For Production environments, Microsoft recommends [deploying a certificate issued from your enterprise PKI](https://learn.microsoft.com/en-us/azure/firewall/premium-deploy-certificates-enterprise-ca). That is fine, so long as it complies with [Microsoft's certificate requirements](https://learn.microsoft.com/en-us/azure/firewall/premium-certificates#intermediate-ca-certificate-requirements) and your enterprise security standards.

The latter was an issue for the customer I was working with because they DO NOT issue Intermediate CA certificates directly from their root CA. In fact, this was the certificate chain that was initially issued to the DMZ Azure Firewall;

![Certificate Chain](/images/2024-03-26-cert-chain.png){:.centered}

Initial suggestions by Microsoft Support were to bundle the certificate chain to uploaded to the Application Gateway, but this did not work. They later confirmed with the Azure Firewall and Application Gateway product teams that you can only upload a single certificate, meaning the certificate chain has to be no more than one level, i.e. a single root CA certificate or a root & intermediate CA certificate.

**Summary:**

PROS:

- Good for Production environments
- Certificate is managed by existing operational and security processes

CONS:

- Only works if your enterprise security standards permit the issuing Intermediate CA certificates directly from your root CA

Issues:

- You can only use your enterprise PKI if the Intermediate CA certificate is issued directly from your root CA, or you use the root CA.

### Option 3 - Certificate auto-generation

The Azure Portal allows you to [auto-generate](https://learn.microsoft.com/en-us/azure/firewall/premium-certificates#certificate-auto-generation) the required Managed Identity, Key Vault and Self-Signed Certificate on your Azure Firewall. This also includes a certificate issuance policy that will auto-renew the certificate, minimising certificate management for your security operations team.

The obvious drawbacks of this are that "vanilla" resources are created, without any of the standard operational and security configuration that your organisation may normally apply when deploying these resources, such as naming convention, Service / Private Endpoints, Certificate Contacts, Diagnostic Settings etc. So, you have to retrospectively add these modifications afterwards.

**Summary:**

PROS:

- Correct certificate is generated for use with Application Gateway and Azure Firewall
- Certificate will auto-renew, minimising management by security operations team

CONS:

- Resources are created with names that may not comply with your enterprise resource naming standard
- Resources are created with configuration that may not comply with your enterprise configuration and security standards
- Certificate cannot be exported for re-use elsewhere

Issues:

- The valid certificate created via the Azure Portal cannot be exported, or replicated using automation as it is not supported by the Azure SDK.

## Wrap-up

Unfortunately, I don't have a recommended one-size-fits all solution to this. All I can give you is some general guidance, and hope that some of what I've learned above will save you wasted effort in trying to implement a solution that is not supported or will not technically work.

My guidance is that you generate the Intermediate CA certificate using one of the following techniques, in order of preference:

1. Use your enterprise PKI if:
   1. It conforms to your enterprise security standards, and
   2. The resulting certificate adheres to [Microsoft's certificate requirements](https://learn.microsoft.com/en-us/azure/firewall/premium-certificates#intermediate-ca-certificate-requirements).
2. Use Microsoft's config & script(s) for [creating a self-signed sertificate](https://learn.microsoft.com/en-us/azure/firewall/premium-certificates#create-your-own-self-signed-ca-certificate) if:
   1. You need to automate the creation of the certificate
   2. Your enterprise security team sign-off on the risks of this approach
3. Use the Azure Portal if:
   1. You're not bothered about automating the certificate and the associated resources
   2. Your enterprise security team sign-off on the risks of this approach

## Conclusion

I hope this will be of help to you. Let me know if you have any suggestions for improvements in the comments.
