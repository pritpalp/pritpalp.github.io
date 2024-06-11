---
title: "How to call one pipeline from another in ADO"
date: 2024-06-07 15:00:00 +0100
toc: false
toc_sticky: false
categories:
  - Azure
tags:
  - Azure
  - Azure Devops
  - Azure Devops CLI
  - Shell script
excerpt: Ever wanted to call multiple pipelines from a single pipeline?
header: 
  overlay_image: /assets/images/post-header.jpg
  caption: "Photo by [**Irvan Smith**](https://unsplash.com/@mr_vero?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [**Unsplash**](https://unsplash.com)"
  overlay_filter: 0.5
---
Sometimes you write a pipeline to do a specific task. And it works great for what it is. Then, someone wants to maybe trigger one of your other pipelines every single time as well.

So now you have two pipelines that you have created and used, but now want to run them both every time, but you don't want to have to trigger each pipeline individually...oh, and the pipelines are in YAML.

We can do this by using [az pipelines run](https://learn.microsoft.com/en-us/cli/azure/pipelines?view=azure-cli-latest#az-pipelines-run) to call the pipeline that we want to run, passing in the ID of the pipeline that we want to call and the parameters that are required.

```
RUN_PIPELINE=`az pipelines run --id $(yourOtherPipelinesId) --org https://[your ADO url]/ --project [your ADO project] --branch $(Build.SourceBranch) --parameters buildBranch=[your branch to build from]`
```
This returns us a JSON object containing useful information. So using [jq](https://jqlang.github.io/jq/) we can pull information that we need, like the id of the pipeline run: 

```
PIPELINE_ID=$(jq -r '.id' <<<"$RUN_PIPELINE")
```
Using this id, we can then query the pipeline using [az pipelines build show](https://learn.microsoft.com/en-us/cli/azure/pipelines/build?view=azure-cli-latest#az-pipelines-build-list) to see the status of the run:

```
MY_RUN=`az pipelines build show --id $PIPELINE_ID --org https://[your ADO url]/ --project [your ADO project]`
```

Now we can put this together into a step for our main pipeline, but when you run it, you'll see that it will trigger the other pipeline and continue. So the triggered pipeline executes outside of this one and you don't know if its worked or not.

But remember that we can query the status of the pipeline that we have just triggered? We can use this to find out when the pipeline completes. We just need to put in a loop and pause before checking the pipeline status again.

This keeps the script active until the pipeline we called finishes.

To authorize the script we pass in the System.AccessToken:

```
env:
  AZURE_DEVOPS_EXT_PAT: $(System.AccessToken)
```

For each of the pipelines that we call, we need to ensure that the permissions are correctly set.

Selecting the pipeline to be called, select manage security:

![ADO Pipeline Security](/assets/images/ado-pipleine-security.jpg)

You need to find the user that ADO will use to run the pipelines. I found the user, but trying to run the pipeline and pulling the name of the user from the error.

The permissions to add are:
 * Edit queue build configuration
 * Manage build qualities
 * Queue builds

![ADO Pipeline Security](/assets/images/ado-pipeline-permissions.jpg)

These permissions allow one pipeline to call another and set parameters for that pipeline.

The job would then look something like this:

```bash
- job: Trigger_Pipeline
  steps:
  - script: |
      RUN_PIPELINE=`az pipelines run --id $(pipelineBId) --org https://dev.azure.com// --project Pipeline_Test_project --branch $(Build.SourceBranch) --parameters buildBranch=$(pipelineBBranch)`
      PIPELINE_ID=$(jq -r '.id' <<<"$RUN_PIPELINE")
      sleep 5
      MY_RUN=`az pipelines build show --id $PIPELINE_ID --org https://dev.azure.com/pipelinetesting/ --project Pipeline_Test_project`
      STATE=$(jq -r '.status' <<<"$MY_RUN")
      while [ $STATE != "completed" ]
      do
        sleep 10
        MY_RUN=`az pipelines build show --id $PIPELINE_ID --org https://dev.azure.com/pipelinetesting/ --project Pipeline_Test_project`
        STATE=$(jq -r '.status' <<<"$MY_RUN")
        RESULT=$(jq -r '.result' <<<"$MY_RUN")
        if [ $RESULT == 'failed' ] || [ $RESULT == 'error' ]
        then
          exit 1
        fi
      done
      echo "Done!"
    env:
      AZURE_DEVOPS_EXT_PAT: $(System.AccessToken)
    failOnStderr: True
```

If you need to run multiple pipelines, you should be able to put this inline script into an actual script and then call it. But some things may then need to be modified (most probably how you authorize the script).