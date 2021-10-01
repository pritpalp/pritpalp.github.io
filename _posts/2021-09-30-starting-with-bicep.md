---
title: "Starting with Bicep"
date: 2021-08-30 20:00:00 +0100
toc: true
toc_sticky: true
categories:
  - Azure
tags:
  - Blog
  - ARM
  - Bicep
excerpt: Using your ARM templates to learn Bicep
header: 
  overlay_image: /assets/images/post-header.jpg
  caption: "Photo by [**Irvan Smith**](https://unsplash.com/@mr_vero?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [**Unsplash**](https://unsplash.com)"
  overlay_filter: 0.5
---
[Bicep](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/) is Microsoft's own answer to [Terraform](https://www.terraform.io). ARM templates written in JSON can get lengthy and complex, they can be tricky to reuse without big changes. Terraform allows you to write more reusable code for your resources, but there can be a delay of up weeks/months implementing new features once they are "live" (for example when a new feature becomes publicly available). Bicep should allow users to create reusable code for resources instead of the JSON ARM templates.

As Bicecp allows you to convert files from ARM templates as well as just writing from scratch, that seems like the best way to get started using them.

## Installing Bicep

I installed Bicep via Brew. It also comes as part of the Az CLI.

Brew install:
```bash
brew install bicep
```

Az CLI install:
```bash
az bicep install
```

Due to some SSL issues I was not able to get the Az CLI version installed. To test the commands I ended up using Azure Cloud Shell, where the az cli is already installed and available (just cloning the repo into my directory to access my files).

Az CLI from version 2.20.0 supports deploying Bicep files (current version is 2.28.0).

## Decompiling ARM

The quickest way to generate the bicep files is to run the `bicep decomplile` command. Point it at an ARM template and it produces a bicep file that you can then work with...once you resolve all the errors/warnings.

So we end up with a decompile, check the errors, fix the errors and repeat cycle to get rid of any warnings and errors.

[Migration ARM to Bicep](https://www.rickroche.com/2021/06/migrating-azure-arm-templates-to-bicep/#common-errors-and-how-to-fix-them) page was really helpful at this stage.

## What's with all these errors?

It seems as though ARM templates were quite forgiving in terms of allowing some properties to be in the *wrong* place. Who would have guessed that? I've often had issues with templates that won't *deploy*, but the meaningless feedback leaves you struggling to work out where the issue is. Once you have the bicep file you can open it up and check the contents.

Let's run through an example, Web App.

Here's the original template:

<script src="https://emgithub.com/embed.js?target=https%3A%2F%2Fgithub.com%2Fpritpalp%2Fazure%2Fblob%2Fmaster%2FARM-Templates%2FWebapp-Bicep%2FInitial-Webapp.json&style=github&showBorder=on"></script>

Running `bicep decopmile webapp.json` results in the following:

```bash
WARNING: Decompilation is a best-effort process, as there is no guaranteed mapping from ARM JSON to Bicep.
You may need to fix warnings and errors in the generated bicep file(s), or decompilation may fail entirely if an accurate conversion is not possible.
If you would like to report any issues or inaccurate conversions, please see https://github.com/Azure/bicep/issues.
webapp.bicep(40,5) : Warning BCP037: The property "name" is not allowed on objects of type "SiteProperties". Permissible properties include "clientCertEnabled", "clientCertExclusionPaths", "cloningInfo", "containerSize", "dailyMemoryTimeQuota", "enabled", "geoDistributions", "hostingEnvironmentProfile", "hostNamesDisabled", "hyperV", "isXenon", "redundancyMode", "reserved", "scmSiteAlsoStopped". If this is an inaccuracy in the documentation, please report it to the Bicep Team. [https://aka.ms/bicep-type-issues]
webapp.bicep(48,5) : Warning BCP037: The property "minTlsVersion" is not allowed on objects of type "SiteProperties". Permissible properties include "clientCertEnabled", "clientCertExclusionPaths", "cloningInfo", "containerSize", "dailyMemoryTimeQuota", "enabled", "geoDistributions", "hostingEnvironmentProfile", "hostNamesDisabled", "hyperV", "isXenon", "redundancyMode", "reserved", "scmSiteAlsoStopped". If this is an inaccuracy in the documentation, please report it to the Bicep Team. [https://aka.ms/bicep-type-issues]
webapp.bicep(63,3) : Warning BCP187: The property "location" does not exist in the resource definition, although it might still be valid. If this is an inaccuracy in the documentation, please report it to the Bicep Team. [https://aka.ms/bicep-type-issues]
webapp.bicep(69,5) : Error BCP034: The enclosing array expected an item of type "module[] | (resource | module) | resource[]", but the provided item was of type "string".
```

Looking at the output we have four errors that we'd need to resolve.

The first one is `The property "name" is not allowed on objects of type "SiteProperties"`. So taking a look that the json we should remove "name" from the SiteProperties section and try again.

The second warning is `The property "minTlsVersion" is not allowed on objects of type "SiteProperties".`. Checking the [documentation](https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.management.websites.models.siteconfig.mintlsversion?view=azure-dotnet) we can see that "minTlsVersion" should be part of the SiteConfig section. Moving it resolves this issue.

The third warning is `The property "location" does not exist in the resource definition, although it might still be valid.` This relates to the "location" value in the hotNameBindings section, removing the property resolves this issue.

The remaining error relates to the fact that the bicep files do not need to have a "dependsOn" section. Removing this section resolves this error.

The final run od decompile gives us this:

```bash
WARNING: Decompilation is a best-effort process, as there is no guaranteed mapping from ARM JSON to Bicep.
You may need to fix warnings and errors in the generated bicep file(s), or decompilation may fail entirely if an accurate conversion is not possible.
If you would like to report any issues or inaccurate conversions, please see https://github.com/Azure/bicep/issues.
```

The resulting file is:

<script src="https://emgithub.com/embed.js?target=https%3A%2F%2Fgithub.com%2Fpritpalp%2Fazure%2Fblob%2Fmaster%2FARM-Templates%2FWebapp-Bicep%2FFinal-Webbapp.bicep&style=github&showBorder=on"></script>

Looking at the names that are generated, you'll see some very long names that could be shorter/more descriptive. For example we have "webapps_name_webapps_name_custom_domain" for the section that adds the app url to the AppConfig store, this could be simplified to something like "webapps-appconfig". So it's worth spending a little more time looking through the resulting bicep file and double checking the names and changing where it makes sense.

Also note that the hyphen in variable names have changed to underscores. You'll need to make changes to either the bicep file or for parameter files to resolve this issue where you find it.

## Got my Bicep file, what now?

Now we can deploy...

This is simple, you can just use the same commands to deploy as you did for your ARM template:

```bash
az deployment group create --name MyDeployment --resource-group MyGroup --template-file webapp.bicep --parameters webapp.parameters
```

Need to run an ARM template, you can compile the bicep file into an ARM template:

```bash
bicep build webapp.bicep
```

As the usual deployment commands work, the other options are also available like *what-if* and *validate*.

Now we can convert our ARM templates, we can explore what else we can do with Bicep...
