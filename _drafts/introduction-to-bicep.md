---
layout: post
title: An introduction to Bicep
categories: iac bicep azure
---

- [Introduction](#introduction)
- [Background](#background)
- [What is Infrastructure as Code (IaC)?](#what-is-infrastructure-as-code-iac)
- [ARM Templates](#arm-templates)
- [What is Bicep?](#what-is-bicep)
  - [Benefits of Bicep](#benefits-of-bicep)
  - [Example of Bicep code](#example-of-bicep-code)
- [Get started](#get-started)
- [Closing notes](#closing-notes)


## Introduction

In this article, we will explain what Bicep is, what benefits it has and why you should use it to deploy your infrastructure in Azure.

![Bicep](../assets/images/post_2023-01_Bicep-logo.png)

## Background

When deploying resources with Azure, you can use the Azure Portal to do so. 
Using the Azure portal to deploy resources is a great way to learn about Azure and getting to know the resource and the configuration options. It also has a small learning curve so you can quickly start with deploying your infrastructure in Azure.

However, deploying manualy via the Azure Portal has several disadvantages:

- Hard to automate
- The Azure Portal interface is subject to changes
- Prone to user errors and typo's
- No consistency in naming

## What is Infrastructure as Code (IaC)?

Infrastructure as Code is - hence the name - a piece of code to deploy your infrastructure. By defining your infrastructure in code, you get the following benefits:

- Reusability
  - You can deploy the exact same infrastructure as many times as you want with a single command
- Consistency
  - Everyone using the code will deploy the exact same resources
- Documentation
  - Your code is key part of your documentation. If you understand the coding language you can understand what will be deployed and in what way
- Version Control
  - It's easy (and recommended) to store your code in a Version Control system like GitHub or Azure DevOps which gives you:
    - Collaboration - multiple people can work on the code simultaniously
    - You can view changes and even revert back to a previous version of the code
- Automation
  - You can automate your deployments using advanced tooling like Ci/Cd
- Modularization
  - You can write your code in modules to easily re-use for other purposes. For example, you can write a module to deploy a Virtual Machine and reuse that module for deploying multiple environments like development, test and production, but each with there environment specific parameters.

## ARM Templates

Before Bicep existed, you could deploy infrastructure as code on Azure using ARM templates. ARM templates use JSON 

## What is Bicep?

Bicep is a domain-specific language (DSL) that uses declarative syntax to deploy Azure resources. In a Bicep file, you define the infrastructure you want to deploy to Azure, and then use that file throughout the development lifecycle to repeatedly deploy your infrastructure. Your resources are deployed in a consistent manner.

Bicep provides concise syntax, reliable type safety, and support for code reuse. Bicep offers a first-class authoring experience for your infrastructure-as-code solutions in Azure.

Bicep is developed by Microsoft and can only be used for deploying resources on Azure (for now).

### Benefits of Bicep

- Simple Syntax - much easier than JSON templates to read and write
- Integration with VSCode
- Preview changes
- No costs and open source

### Example of Bicep code

The following code example is a simple bicep file to deploy a storage account in Azure in an existing resource group.

```powershell
param location string = resourceGroup().location
param storageAccountName string = 'ictstuff${uniqueString(resourceGroup().id)}'

resource storageAccount 'Microsoft.Storage/storageAccounts@2021-06-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
  properties: {
    accessTier: 'Hot'
  }
}
```

Now, look at the same code to deploy this with an ARM template:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    },
    "storageAccountName": {
      "type": "string",
      "defaultValue": "[format('ictstuff{0}', uniqueString(resourceGroup().id))]"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-06-01",
      "name": "[parameters('storageAccountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "StorageV2",
      "properties": {
        "accessTier": "Hot"
      }
    }
  ]
}
```

When you have the Bicep extension installed in VSCode, you get a wonderful authoring experience with type-safety, intellisense, and syntax validation. So this means you can type a couple of letters and intellisense lets you autocomplete and warns you of any missing required parameters and syntax errors.

## Get started

To get started with Bicep quickly, you can do the following:

1. Install the tools. See [Set up Bicep development and deployment environments.](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/install)
2. Complete the [quickstart](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/quickstart-create-bicep-use-visual-studio-code) and the [Learn modules for Bicep.](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/learn-bicep)

## Closing notes

After having used ARM templates and Terraform to deploy Azure resources, I found Bicep to be a nice programming language that was easy to get into. The Microsoft Learn documentation and learning modules are a great way to quickly get familiar with the language and hands-on labs.