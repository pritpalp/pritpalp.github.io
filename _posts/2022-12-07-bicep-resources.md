---
title: "Existing or New Resources in a Bicep Template"
date: 2022-12-07 20:00:00 +0100
toc: false
toc_sticky: false
categories:
  - Azure
tags:
  - Blog
  - Bicep
excerpt: How to reference existing resources or create them in Bicep
header: 
  overlay_image: /assets/images/post-header.jpg
  caption: "Photo by [**Irvan Smith**](https://unsplash.com/@mr_vero?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [**Unsplash**](https://unsplash.com)"
  overlay_filter: 0.5
---
So, you've started to work with Bicep rather than ARM...now lets say you want to deploy a Web App.

That's not too bad.

```bicep
param webappName string
param location string = resourceGroup().location
param hostingPlanName string

resource serverFarm 'Microsoft.Web/serverfarms@2020-06-01' = {
  name: hostingPlanName
  location: location
  sku: {
    name: 'S1'
    tier: 'Standard'
  }
}

resource webapp 'Microsoft.Web/sites@2020-06-01' = {
  name: webappName
  location: location
  kind: 'app'
  properties: {
    serverFarmId: serverFarm.id
    siteConfig: {
      alwaysOn: true
    }
  }
}
```

Now, say we need to link Application Insights to the app when we deploy it. You'd just need to add in the `InstrumentationKey` for Application Insights to a property of the webapp:

```bicep
resource webapp 'Microsoft.Web/sites@2020-06-01' = {
  name: webappName
  location: location
  kind: 'app'
  properties: {
    serverFarmId: serverFarm.id
    siteConfig: {
      alwaysOn: true
      appSettings: [
        {
          name: 'APPINSIGHTS_INSTRUMENTATIONKEY'
          value: InstrumentationKey
        }
      ]
    }
  }
}
```

So where does the value for that come from? In certain environments we already have Application Insights deployed (for example in production). In this case we can just add a few lines to reference the existing resource and pull out the value in our template.

```bicep
resource existingAppInsights 'Microsoft.Insights/components@2020-02-02' existing = {
  name: appInsightsName
  scope: resourceGroup(appInsightsRG)
}

resource webapp 'Microsoft.Web/sites@2020-06-01' = {
  name: webappName
  location: location
  kind: 'app'
  properties: {
    serverFarmId: serverFarm.id
    siteConfig: {
      alwaysOn: true
      appSettings: [
        {
          name: 'APPINSIGHTS_INSTRUMENTATIONKEY'
          value: existingAppInsights.properties.InstrumentationKey
        }
      ]
    }
  }
}
```

Note that we declare the resource and use the `existing` keyword. The scope is used to search the correct Azure resource group (in this case) for an Application Insights deployment. We can then use this reference to get the value of the `InstrumentationKey` and set it to a value in `appSettings`.

Now, that's great for an environment where a resource already exists, but let's say Dev (for reasons) doesn't have a permanent resource for the above code to find. Running the template would fail when the reference to the *existing* Application Insights fails to find an instance.

In this case, you'd want your template to create an instance and use that to get a value for `InstrumentationKey`. So the template would look something like this:

```bicep
module appInsights '/modules/appinsights.bicep' = {
  name: appInsightsName
  params: {
    applicationInsightsName: appInsightsName
    location: location
    logAnalyticsNamespaceName: logAnalyticsNamespaceName
    workspaceSKU: workspaceSKU
  }
}

resource webapp 'Microsoft.Web/sites@2020-06-01' = {
  name: webappName
  location: location
  kind: 'app'
  properties: {
    serverFarmId: serverFarm.id
    siteConfig: {
      alwaysOn: true
      appSettings: [
        {
          name: 'APPINSIGHTS_INSTRUMENTATIONKEY'
          value: appInsights.outputs.InstrumentationKey
        }
      ]
    }
  }
}
```

Here you use a module to create an Application Insights instance and pull the `InstrumentationKey` from the module outputs.

So the aim was to combine both these into a single bicep file that would enable the code to either create an instance or use the existing instance to grab the value that we need. Sounds simple, but after many attempts the only way to do this was by introducing another variable. This was used to determine if the environment that we were using need to create or reference an existing instance.

So the template became:

```bicep
param webappName string
param location string = resourceGroup().location
param hostingPlanName string
param appInsightsName string
param appInsightsRG string
param logAnalyticsNamespaceName string
param workspaceSKU string
param aiExists bool

resource existingAppInsights 'Microsoft.Insights/components@2020-02-02' existing = if (aiExists) {
  name: appInsightsName
  scope: resourceGroup(appInsightsRG)
}

module appInsights '/modules/appinsights.bicep' = if (!aiExists){
  name: appInsightsName
  params: {
    applicationInsightsName: appInsightsName
    location: location
    logAnalyticsNamespaceName: logAnalyticsNamespaceName
    workspaceSKU: workspaceSKU
  }
}

var InstrumentationKey = (aiExists) ? existingAppInsights.properties.InstrumentationKey : appInsights.outputs.InstrumentationKey

resource webapp 'Microsoft.Web/sites@2020-06-01' = {
  name: webappName
  location: location
  kind: 'app'
  properties: {
    serverFarmId: serverFarm.id
    siteConfig: {
      alwaysOn: true
      appSettings: [
        {
          name: 'APPINSIGHTS_INSTRUMENTATIONKEY'
          value: InstrumentationKey
        }
      ]
    }
  }
}
```

Note the `var InstrumentationKey` value. We use a conditional with the `aiExists` to determine which value of the InstrumentationKey we use. All the parameters need to have values supplied. For the ones that don't matter for that particular run, we can just supply dummy values as they won't do anything.

If it's not straightforward to get decide if the Application Insights exists before hand, you could run a simple Azure CLI/Powershell script to search and then return the correct boolean value before running the template in your pipeline.
