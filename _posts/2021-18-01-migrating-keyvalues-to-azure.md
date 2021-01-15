---
layout: post
title: "Azure Key Values"
date: 2021-01-18 12:00:00 -0000
categories: azure app-config key0vault
---
# Migrating Key Values to Azure

When migrating to the cloud, one of the major concerns for many clients is security. 
When it comes to storing confidential information, azure offers scalable, fully managed and secure services. 
This article will demonstrate, using Azure Key Vault and Azure App Configuration, a commmon migration solutions for storing secrets.

## Key vault
Azure Key Vault is a cloud service for securely storing and accessing secrets.
A secret is anything that you want to tightly control access to, such as API keys, passwords, certificates, or cryptographic keys.
Access to key vault can be controlled using [Managed Identities](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview),
and access policies with the common RBAC used across Azure for indiviudal permission levels. 

> Our recommendation is to use a vault per application per environment (Development, Pre-Production and Production). 
This helps you not share secrets across environments and also reduces the threat in case of a breach.

Key vault alone is a good solution for managing secret values in the cloud, it has a REST API for getting values and versioning available. However, there are a few things missing, such as real time updates, feature management and tagging, which is where App configuration comes in..

## Azure App Configuration
Azure App Configuration provides a service to centrally manage application settings and feature flags. Modern programs, especially programs running in a cloud, generally have many components that are distributed in nature. 
Spreading configuration settings across these components can lead to hard-to-troubleshoot errors during an application deployment.
 
 
zepiUse App Configuration to store all the settings for your application and secure their accesses in one place. 

Naming conventions should be drawn out before implementation to ensure consistency going forward, our recommended approach is to label them by domain, for example see below.
> common/apiKey             
> apis/basket-service/retryAttempts         
> external/twitter/apiKey

## Tieing it together
App Configuration complements Azure Key Vault, which is used to store application secrets. App Configuration makes it easier to implement the following scenarios:

* Centralize management and distribution of hierarchical configuration data for different environments and geographies
* Dynamically change application settings without the need to redeploy or restart an application,
* Control feature availability in real-time
* No need to expose key vault values


There are other methods of storing sensitive information in azure, such as Azure Table Storage or Cosmos DB Table, but these don't provide the same features and will generally cost alot more than the two above.