---
title: "Configure .Net Core code coverage"
date: 2021-08-19 20:00:00 +0100
categories:
  - Azure
tags:
  - Blog
  - Azure Devops Pipelines
excerpt: How to get code coverage working for .Net Core with Azure Devops
header: 
  overlay_image: /assets/images/post-header.jpg
  caption: "Photo by [**Irvan Smith**](https://unsplash.com/@mr_vero?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [**Unsplash**](https://unsplash.com)"
  overlay_filter: 0.5
---

Your dev's say they want to see the code coverage reports in Azure Devops...and it must be easy because Visual Studio "just does it"...

Sounds simple...everything is an MS product - Azure Devops, Visual Studio, .Net Core, what could possibly cause issues...

So you want to be able to see the results when your pipeline runs, Note the two tabs next to **Summary**, *Tests* and *Code Coverage*:

![Pipeline Run Page](/assets/images/cc-run-summary.jpg)

Is there a built in task that will do it for you? Yes and no.

I found plenty of examples just adding `/p:CollectCoverage=true /p:CoverletOutputFormat=cobertura` to the build options. But, trying that out I found that there were no reports created to upload. A little digging around and its related to .Net Core. Your project uses .Net Core so you need to do things a little differently.

First off you need to install the [coverlet collector](https://github.com/coverlet-coverage/coverlet) yourself so you can collect the required data.

This allows you to run the `dotnet test --collect:"XPlat Code Coverage"` command. Now you can collect the results in a similar way to adding `/p:CollectCoverage=true`, but you still need to create a report that you can publish.

To do this you'll need to install the [Report Generator](https://danielpalme.github.io/ReportGenerator/usage.html). This takes the collected data and can transform it into a variety of formats (see the link above for more information).

Now as we want to read them within Azure Devops, we what the report to be in cobertura format. Running *reportgenerator* to convert the report into format that Azure Devops likes, we can use the usual [PublishCodeCoverageResults@1](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/test/publish-code-coverage-results?view=azure-devops) task to get it into our pipeline run page.

The code coverage results in Azure Devops:
![Code Coverage example](/assets/images/cc-coverage-page.jpg)

The test results in Azure Devops:
![Tests Page example](/assets/images/cc-tests-page.jpg)

Here's the YAML pipeline section for this:

<script src="https://gist.github.com/pritpalp/ccdb68a940701ef4e0e7f2831e2aeb14.js"></script>
