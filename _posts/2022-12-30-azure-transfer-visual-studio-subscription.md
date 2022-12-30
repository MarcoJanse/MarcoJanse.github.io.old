---
layout: post
title: Transfer your Visual Studio Enterprise subscription to another Azure tenant
categories: azure
---

- [Transfer your Visual Studio Enterprise subscripion to another Azure tenant](#transfer-your-visual-studio-enterprise-subscripion-to-another-azure-tenant)
  - [Introduction](#introduction)
  - [Best practices](#best-practices)
  - [Prerequisites](#prerequisites)
  - [Steps](#steps)
    - [Invite/create guest account in source tenant](#invitecreate-guest-account-in-source-tenant)
    - [Accept the invitation](#accept-the-invitation)
    - [Validate Guest user and assign subscription permissions](#validate-guest-user-and-assign-subscription-permissions)
    - [Move the subscription to the target tenant](#move-the-subscription-to-the-target-tenant)
    - [Remove Guest account from the source tenant](#remove-guest-account-from-the-source-tenant)
  - [Closing notes](#closing-notes)


## Introduction

Chances are that you are reading this because your company gave you a Visual Studio subscription and you have activated your free monthly Azure credits while still being logged in with your company's AzureAD account, resulting in associating your Visual Studio Enterprise MPN subscription to your company's Azure AD tenant, which usually has a couple of drawbacks:

- You probably won't have owner/administrator permissions in the AzureAD tenant
- There are a lot of existing resources in your company's tenant, which makes it far from the ideal Azure playground

So for example, creating your own Azure Landing zone will not be possible as you will need full access to the tenant and all subscriptions. 

## Best practices

To use your monthly Azure credits to the fullest, you should link this subscription to a new Azure AD tenant, or greenfield tenant for which you have full administrative access and can build and test from scratch without interfering with any existing resources or copy with any limitations.

Fortunately, there is a way to move your Azure subscription to another Azure AD tenant.

## Prerequisites

- First of all, you need to have an AzurAD account in your company's tenant. This is the source tenant.
- Secondly, you need to have a second AzureAD tenant/Azure environment that you have full control over (Global Administrator role). If you don't have one, I suggest you get yourself a [Microsoft 365 Business Premium Trial for one month](https://go.microsoft.com/fwlink/p/?LinkID=2102309&clcid=0x409&culture=en-us&country=US) or an [Azure free trial](https://azure.microsoft.com/en-us/offers/ms-azr-0044p/) to get yourself a nice clean tenant and some additional Azure credits and/or some Microsoft 365 functionality to test.
- You need to identify which subscription is yours. If there are multiple MPN subscriptions, you should be able to find yours via [Portal settings \| Directory + subscriptions](https://portal.azure.com/#settings/directory)
- Make sure there are no resources linked to this subscription. (you can move some resources, with your subscription, but it's generally better for these kind of subscriptions to just stat over with deploying resources in the new tenant)
- You will need account operator or owner permissions on the Visual Studio Enterprise subscription in the source tenant.
- You need to be able to invite guests in your company's AzureAD tenant. You can find this under `User settings` - `External collaboration settings`

<br>

![External collaboration settings](../assets/images/post_2022-12_azure-azuread-externalcollaboration-settings.png)

<br>

  - In my company's tenant, I had these permissions as I was a member of an administrative role and the Guest invite settings were set to `Member users and users assigned to specific admin roles can invite guest users including guests with member permissions` 

With all these requirements covered, lets get started!

## Steps

### Invite/create guest account in source tenant

- In your source tenant go to the [Azure AD portal](azad.cmd.ms) or the new [Entra portal](ad.cmd.ms)
- Under Users - New Users, select the `Invite External User` option
  - Depending on the template you will only need to enter the Email address of the administrator of the target tenant or fill in some more details.
- After this, an Email should be sent to the target tenants administrator account.

### Accept the invitation

- Open the Email of the user/administrator from the other tenant and you should have an Email from [invites@microsoft.com](invites@microsoft.com).
- Using the link from the Email accept the invitation

### Validate Guest user and assign subscription permissions

- Back in the source tenant, check the Azure AD users. There should now be a new guest user from your target tenant
- In the Azure portal, go to [Subscriptions blade](https://portal.azure.com/#view/Microsoft_Azure_Billing/SubscriptionsBlade) and select the Visual Studio Enterprise subscription you want to move.
- Go to Access Control (IAM)
  - Select Role Assignments and click **+Add** and `Add Role Assignment`
  - Select `Owner` as the role
  - Under, members click the `Select Members` and lookup up the guest account you invited in the previous step
  - Click Next and Finish
  - Your Guest user should now be able to manage this subscription

<br>

![Add role assignment](../assets/images/post_2022-12_azure-IAM-add-role-assignment.png)

<br>

### Move the subscription to the target tenant

- Login to Azure with the guest user account on the source tenant
  - You might need to do this by logging in to your own tenant and switch directory by using the [Portal settings \| Directory + subscriptions](https://portal.azure.com/#settings/directory)
  - There, switch to the source tenant
- Go to [Subscriptions blade](https://portal.azure.com/#view/Microsoft_Azure_Billing/SubscriptionsBlade) and select the Visual Studio Enterprise subscription you want to move.
- This time, choose `Change directory`

<br> 

![Subscription options](../assets/images/post_2022-12_azure-subsciption-options-change-directory.png)

<br>

- You should be able to select the source and target directory, as you have access to boh and subscription owner permissions
- Initiate the move after reading and selecting the disclaimer and click on `Change`

The move will take some time to refelect on both subscriptions. Give it about 20 minutes and click refresh and repeat until you no longer see the subscription in the source tenant and are able to see the subscription in the target tenant. Do not proceed to the last step until this is ok.

### Remove Guest account from the source tenant

Finally, we can clean up the guest account in the source tenant. 

-The easiest way to do this is by using your own tenant account and going to [https://myaccount.microsoft.com/organizations](https://myaccount.microsoft.com/organizations)
- Here you should be able to select the source tenant and leave.

## Closing notes

I hope this article will help you to migrate your Visual Studio Enterprise (or any) Azure subscription. 
If you want to migrate a subscription with resources to a different tenant, have a look at this article: [Transfer an Azure subscription to a different Azure AD directory \| Microsoft Learn](https://learn.microsoft.com/en-us/azure/role-based-access-control/transfer-subscription)