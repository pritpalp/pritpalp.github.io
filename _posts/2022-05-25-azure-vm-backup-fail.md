---
title: "Azure VM Backup Job Fails"
date: 2022-05-25 14:00:00 +0100
categories:
  - Azure
  - Azure VM Backup
tags:
  - Blog
  - Azure
excerpt: Azure VM Backup Job Fails
header: 
  overlay_image: /assets/images/post-header.jpg
  caption: "Photo by [**Irvan Smith**](https://unsplash.com/@mr_vero?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [**Unsplash**](https://unsplash.com)"
  overlay_filter: 0.5
---

So, you have an Azure VM and setup backup. You've configured a backup policy, got a recovery vault created and setup.

You log in the next day to see how your backup when and see that it failed.

The error I could see from the VM blade was very generic and didn't tell me anything. I spent some time checking the actual VM to ensure that it was all correct (checking the Guest Agent was running, checking through the Event Log for errors relating to Backup), but didn't find any answers as to why the job failed - everything seemed to be correct.

Navigating to the *Backup Jobs* blade of the Recovery Services vault, I was able to get a much more meaningful error message.

![Failure Error Message](/assets/images/azure-vm-backup-error.png)

Looking up the error here, [UserErrorRequestDisallowedByPolicy](https://docs.microsoft.com/en-gb/azure/backup/backup-azure-vms-troubleshoot?WT.mc_id=Portal-Microsoft_Azure_Support#usererrorrequestdisallowedbypolicy---an-invalid-policy-is-configured-on-the-vm-which-is-preventing-snapshot-operation), lead me to the solution. The subscription has a policy enabled that will not create a resource unless it is correctly tagged.

The Backup process creates a resource group to store the recovery points and uses a default naming convention. The way to resolve this is to manually create your resource group with the tags you need and then change your backup policy to use this resource group.

![Backup Policy Screen](/assets/images/azure-vm-backup-policy.png)

So we can give the name of a resource group we have created here, but there is something to pay attention to.

You need to follow the naming convention. Let's say you want a group called "myrestorepointcollection-uksouth". Reading the documentation and checking the field in the backup policy form, you **need** a number at the end of that name, otherwise the backup process will attempt to create one with the name "myrestorepointcollection-uksouth1". But remember to omit the number when you enter the name in the backup policy form.

* Resource Group created - *myrestorepointcollection-uksouth1*
* Value in backup policy - *myrestorepointcollection-uksouth*

Each one of these resource groups can collect up to 600 restore points. When you reach that limit, the backup service will attempt to create another resource group. So from our example, the new resource group will be "myrestorepointcollection-uksouth2".
