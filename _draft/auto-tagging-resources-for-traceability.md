---
layout: post
title: Auto-tagging resources for traceability
tags: azure tagging frameworks
image: 2024-05-30-auto-tagging-resources-for-traceability.png
---
Ever looked at a bunch of resources in a resource group in the Azure Portal and wondered, "What code repository do I need to modify to fix the issue with those resources?!". Well, there are some tagging frameworks for Terraform that will help you to answer those questions, albeit requiring some work to make answering those questions. So, let's take a look at the options and how you can use them.

## What tagging framework options are there for Terraform?

There are two main frameworks that I've found that you can use:

- [Yor](){:target="_blank"} by [env0](https://www.env0.com/){:target="_blank"}
- [Terragrunt](https://www.terratag.io/){:target="_blank"} by [bridgecrew](https://bridgecrew.io/){:target="_blank"}

Both frameworks will scan Terraform code and add tags to resources that it finds, but they provide different capabilities. So, let's dive into what each one can do.
