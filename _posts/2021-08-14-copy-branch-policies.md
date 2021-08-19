---
title: "Copying Branch Policies in Azure Devops Repos!"
date: 2021-08-14 15:00:00 +0100
toc: true
toc_sticky: true
categories:
  - Azure
tags:
  - Blog
  - Azure CLI
  - Azure Devops Repos
  - Azure Devops CLI
  - Shell script
excerpt: How to copy branch policies between branches in Azure Devops
header: 
  overlay_image: /assets/images/post-header.jpg
  caption: "Photo by [**Irvan Smith**](https://unsplash.com/@mr_vero?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [**Unsplash**](https://unsplash.com)"
  overlay_filter: 0.5
---

Your repo is in Azure Devops and you have a number of Branch Policies active that you want to carry over to your new branches...

It's a simple scenario, you create a sprint branch at the end of each sprint and want to maintain the build validation steps for the code. As your code base increases, you add more and more validation steps. At the start of a project, this is manageable - you have maybe secret validation and a code coverage check. A year latter, your two build validation steps have become twenty all with individual criteria.

Creating a new sprint branch has gone from taking five minutes to thirty minutes or more - mostly copying and pasting between two windows...this isn't what your job should be.

## The azure cli azure-devops extension

There is an azure-devops extension for the az cli. Available from <https://github.com/Azure/azure-devops-cli-extension>

To install the extension you need to have [az-cli](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) version 2.0.69 as of writing.

{% highlight bash %}
az extension add --name azure-devops
{% endhighlight %}

Once installed just log in and you now have access to some more cli commands for Azure Devops!

{% highlight bash %}
➜ az devops -h

Group
    az devops : Manage Azure DevOps organization level operations.
        Related Groups
        az pipelines: Manage Azure Pipelines
        az boards: Manage Azure Boards
        az repos: Manage Azure Repos
        az artifacts: Manage Azure Artifacts.

Subgroups:
    admin            : Manage administration operations.
    extension        : Manage extensions.
    project          : Manage team projects.
    security         : Manage security related operations.
    service-endpoint : Manage service endpoints/connections.
    team             : Manage teams.
    user             : Manage users.
    wiki             : Manage wikis.

Commands:
    configure        : Configure the Azure DevOps CLI or view your configuration.
    invoke           : This command will invoke request for any DevOps area and resource. Please use
                       only json output as the response of this command is not fixed. Helpful docs -
                       https://docs.microsoft.com/en-us/rest/api/azure/devops/.
    login            : Set the credential (PAT) to use for a particular organization.
    logout           : Clear the credential for all or a particular organization.

For more specific examples, use: az find "az devops"

Please let us know how we are doing: https://aka.ms/azureclihats
{% endhighlight %}

## Using the azure-devops cli

Before you start, if you run these commands from within a checked out Git repo you don't have to specify your Azure Devops organization (via the --organization *url*) and your project (--project *project name or id*) which is what I'll be assuming.

Now you can explore your repo settings...So lets start with pulling back your repos

(id's and other sensitive things have been sanitized with hashes)

{% highlight bash %}
➜  az repos list
[
  {
    "defaultBranch": "refs/heads/sprint-12",
    "id": "#########",
    "isDisabled": false,
    "isFork": null,
    "name": "MyTestRepo",
    "parentRepository": null,
    "project": {
      "abbreviation": null,
      "defaultTeamImageUrl": null,
      "description": null,
      "id": "#########",
      "lastUpdateTime": "2020-07-06T11:24:23.807Z",
      "name": "Test",
      "revision": 47,
      "state": "wellFormed",
      "url": "https://dev.azure.com/###/_apis/projects/######",
      "visibility": "private"
    },
    "remoteUrl": "https://###@dev.azure.com/###/Test/_git/MyTestRepo",
    "size": 42832884,
    "sshUrl": "git@ssh.dev.azure.com:v3/###/Test/MyTestRepo",
    "url": "https://dev.azure.com/###/#########/_apis/git/repositories/#########",
    "validRemoteUrls": null,
    "webUrl": "https://dev.azure.com/###/Test/_git/MyTestRepo"
  },
  {
    "defaultBranch": "refs/heads/master",
    "id": "######",
    "isDisabled": false,
    "isFork": null,
    "name": "AnOtherRepo",
    "parentRepository": null,
    "project": {
      "abbreviation": null,
      "defaultTeamImageUrl": null,
      "description": null,
      "id": "######",
      "lastUpdateTime": "2020-07-06T11:24:23.807Z",
      "name": "Test",
      "revision": 47,
      "state": "wellFormed",
      "url": "https://dev.azure.com/###/_apis/projects/######",
      "visibility": "private"
    },
    "remoteUrl": "https://###@dev.azure.com/###/Test/_git/AnOtherRepo",
    "size": 130577,
    "sshUrl": "git@ssh.dev.azure.com:v3/###/Test/AnOtherRepo",
    "url": "https://dev.azure.com/###/######/_apis/git/repositories/######",
    "validRemoteUrls": null,
    "webUrl": "https://dev.azure.com/###/Test/_git/AnOtherRepo"
  }
]
{% endhighlight %}

So in the example above we have two repositories, MyTestRepo & AnOtherRepo, within the project Test. We can see a fair bit of information for the repos. Note the top level *id*, we will need this to read the repos policy and write the policy.

## JMESPath queries to limit results

If you run the **az repos policy list** command, you will get back the policies on all branches of all repos in your project. Not very readable, so you need to use the use a [JMESPath](https://jmespath.org) query to limit your results. Now, this is a step that took the most time - figuring out the correct query was not easy using something that I wasn't familiar with. The cli does provide some help and examples:

```bash
➜  az repos policy list --query-examples
Query string                                                                         Help
-----------------------------------------------------------------------------------  ----------------------------------------------------
[].isDeleted                                                                         Show the value of isDeleted field.
[].isEnabled                                                                         Show the value of isEnabled field.
[?isEnabled=='True']                                                                 Show the resources that satisfy the condition.
[?contains(@.isEnabled, 'something')==\`true\`].isEnabled                            Show the isEnabled field that contains given string.
[].revision                                                                          Show the value of revision field.
[?revision=='1']                                                                     Show the resources that satisfy the condition.
[?contains(@.revision, 'something')==\`true\`].revision                              Show the revision field that contains given string.
[].isEnterpriseManaged                                                               Show the value of isEnterpriseManaged field.
[].url                                                                               Show the value of url field.
```

Let's grab the id for our MyTestRepo by querying by name to pull back only the id.

```bash
➜  az repos list --query "[?name=='MyTestRepo'].id"
[
 "#####"
]
```

If we use this to id we can then get all the policies on the repo - but again this will bring back all the policies on all the branches. A little playing with the query and we can limit this to a particular branch. Note the two query clauses joined by a **|** pipe. The result of the left side is passed to the right side and filtered again to return only the values with the matching repository and branch name.

```bash
➜  az repos policy list --query "[?contains(settings.scope[].repositoryId,'#####')] | [?contains(settings.scope[].refName, 'refs/heads/sprint-12')]"
[
  {
    "createdBy": {
      "descriptor": "###",
      "directoryAlias": null,
      "displayName": "Pritpal",
      "id": "###",
      "imageUrl": "https://dev.azure.com/###/_api/_common/identityImage?id=####",
      "inactive": null,
      "isAadIdentity": null,
      "isContainer": null,
      "isDeletedInOrigin": null,
      "profileUrl": null,
      "uniqueName": "pritpalp@kainos.com",
      "url": "https://spsproduks1.vssps.visualstudio.com/###/_apis/Identities/###"
    },
    "createdDate": "2021-06-23T11:30:50.424761+00:00",
    "id": 247,
    "isBlocking": true,
    "isDeleted": false,
    "isEnabled": true,
    "isEnterpriseManaged": false,
    "revision": 1,
    "settings": {
      "allowSquash": true,
      "scope": [
        {
          "matchKind": "Exact",
          "refName": "refs/heads/sprint-12",
          "repositoryId": "###"
        }
      ]
    },
    "type": {
      "displayName": "Require a merge strategy",
      "id": "###",
      "url": "https://dev.azure.com/###/###/_apis/policy/types/###"
    },
    "url": "https://dev.azure.com/###/###/_apis/policy/configurations/247"
  },
  {
    "createdBy": {
      "descriptor": "###",
      "directoryAlias": null,
      "displayName": "Pritpal",
      "id": "###",
      "imageUrl": "https://dev.azure.com/###/_api/_common/identityImage?id=###",
      "inactive": null,
      "isAadIdentity": null,
      "isContainer": null,
      "isDeletedInOrigin": null,
      "profileUrl": null,
      "uniqueName": "pritpalp@kainos.com",
      "url": "https://spsproduks1.vssps.visualstudio.com/###/_apis/Identities/###"
    },
    "createdDate": "2021-06-23T11:30:55.534151+00:00",
    "id": 248,
    "isBlocking": true,
    "isDeleted": false,
    "isEnabled": true,
    "isEnterpriseManaged": false,
    "revision": 1,
    "settings": {
      "buildDefinitionId": 34,
      "displayName": "Secret Scanning",
      "manualQueueOnly": false,
      "queueOnSourceUpdateOnly": true,
      "scope": [
        {
          "matchKind": "Exact",
          "refName": "refs/heads/sprint-12",
          "repositoryId": "###"
        }
      ],
      "validDuration": 720.0
    },
    "type": {
      "displayName": "Build",
      "id": "###",
      "url": "https://dev.azure.com/###/###/_apis/policy/types/###"
    },
    "url": "https://dev.azure.com/###/###/_apis/policy/configurations/248"
  },
  {
    "createdBy": {
      "descriptor": "###",
      "directoryAlias": null,
      "displayName": "Pritpal",
      "id": "###",
      "imageUrl": "https://dev.azure.com/###/_api/_common/identityImage?id=###",
      "inactive": null,
      "isAadIdentity": null,
      "isContainer": null,
      "isDeletedInOrigin": null,
      "profileUrl": null,
      "uniqueName": "pritpalp@kainos.com",
      "url": "https://spsproduks1.vssps.visualstudio.com/###/_apis/Identities/###"
    },
    "createdDate": "2021-06-23T11:30:58.081030+00:00",
    "id": 249,
    "isBlocking": true,
    "isDeleted": false,
    "isEnabled": true,
    "isEnterpriseManaged": false,
    "revision": 1,
    "settings": {
      "buildDefinitionId": 61,
      "displayName": "Repositories Code Coverage",
      "filenamePatterns": [
        "/COW.Repositories/*"
      ],
      "manualQueueOnly": false,
      "queueOnSourceUpdateOnly": true,
      "scope": [
        {
          "matchKind": "Exact",
          "refName": "refs/heads/sprint-12",
          "repositoryId": "###"
        }
      ],
      "validDuration": 720.0
    },
    "type": {
      "displayName": "Build",
      "id": "###",
      "url": "https://dev.azure.com/###/###/_apis/policy/types/###"
    },
    "url": "https://dev.azure.com/###/###/_apis/policy/configurations/249"
  },
  {
    "createdBy": {
      "descriptor": "###",
      "directoryAlias": null,
      "displayName": "Pritpal",
      "id": "###",
      "imageUrl": "https://dev.azure.com/###/_api/_common/identityImage?id=###",
      "inactive": null,
      "isAadIdentity": null,
      "isContainer": null,
      "isDeletedInOrigin": null,
      "profileUrl": null,
      "uniqueName": "pritpalp@kainos.com",
      "url": "https://spsproduks1.vssps.visualstudio.com/###/_apis/Identities/###"
    },
    "createdDate": "2021-06-25T13:28:53.358440+00:00",
    "id": 256,
    "isBlocking": true,
    "isDeleted": false,
    "isEnabled": true,
    "isEnterpriseManaged": false,
    "revision": 4,
    "settings": {
      "allowDownvotes": false,
      "blockLastPusherVote": false,
      "creatorVoteCounts": false,
      "minimumApproverCount": 1,
      "requireVoteOnLastIteration": false,
      "resetOnSourcePush": false,
      "resetRejectionsOnSourcePush": false,
      "scope": [
        {
          "matchKind": "Exact",
          "refName": "refs/heads/sprint-12",
          "repositoryId": "###"
        }
      ]
    },
    "type": {
      "displayName": "Minimum number of reviewers",
      "id": "###",
      "url": "https://dev.azure.com/###/###/_apis/policy/types/###"
    },
    "url": "https://dev.azure.com/###/###/_apis/policy/configurations/256"
  }
]
```

Now, lets say we only want the display name of a policy and its id again applying a pipe to then list only those values ([].[*the values we want to return*]):

```bash
➜  branch-policies git:(master) ✗ az repos policy list --query "[?contains(settings.scope[].repositoryId,'###')] | [?contains(settings.scope[].refName, 'refs/heads/sprint-16')] | [].[settings.buildDefinitionId, settings.displayName]"
[
  [
    null,
    null
  ],
  [
    null,
    null
  ],
  [
    34,
    "Secret Scanning"
  ],
  [
    61,
    "Code Coverage"
  ]
]
```

The default output is in JSON, so we can change that to table to make it easier to read:

```bash
➜  branch-policies git:(master) ✗ az repos policy list --query "[?contains(settings.scope[].repositoryId,'###')] | [?contains(settings.scope[].refName, 'refs/heads/sprint-16')] | [].[settings.buildDefinitionId, settings.displayName]" --output table
Column1    Column2
---------  -------------------------------------------------------


34         Secret Scanning
61         Repositories Code Coverage
```

Note the two empty results, these are for policies that are not build policies. There are policies (for example the merge strategy or minimum number of reviewers) and build policies (code you want to run before building pull requests and pre-merging).

## Copying policies

We can use the cli to pull back the values for a branch in your repo...now its just a case of copying them to your new branch. You know the id or your repo, and now you have all the extra info you need to copy policies to another branch.

You can either use the [cli](https://docs.microsoft.com/en-us/cli/azure/repos/policy?view=azure-cli-latest) or a [policy file](https://docs.microsoft.com/en-gb/azure/devops/cli/policy-configuration-file?view=azure-devops) to create the policy.

Here's a bit of bash to demo copying the build policies from *branch a* to *branch b* via the cli:

```bash
#!/bin/bash -e
repo_name=$1
copy_from=$2
copy_to=$3
repo_id=$(az repos list --query "[?name=='$repo_name'].id" -o tsv)

# list the policies in the source branch
policy_list=$(az repos policy list --query "[?contains(settings.scope[].repositoryId,'$repo_id')] | [?contains(settings.scope[].refName, 'refs/heads/$copy_from')] | [].[settings.buildDefinitionId, settings.displayName, settings.filenamePatterns[0]]" --output tsv | tr '\t' ',')
# results in a tsv format, so we remove the tabs and replace with a comma
IFS=$'\n' 
array=($policy_list)
# we put the list into an array split by the new line char, so we have an array of values - one for each policy

for i in "${array[@]}"
do
    # some policies are not "build", so we want to ignore those
    if [[ "$i" != "None,None,None" ]]; then
        # split the element with the comma seperator, ready to use to create the policy
        IFS=$','
        vals=($i)
        if [[ "${vals[2]}" == "None" ]]; then
            echo "Create the policy for ${vals[1]}"
            create_policy=$(az repos policy build create --branch $copy_to --enabled true --blocking true --queue-on-source-update-only true --manual-queue-only false --valid-duration 720 --repository-id $repo_id --build-definition-id ${vals[0]} --display-name "${vals[1]}")
        else
            echo "Create the policy for ${vals[1]}"
            create_policy=$(az repos policy build create --branch $copy_to --enabled true --blocking true --queue-on-source-update-only true --manual-queue-only false --valid-duration 720 --repository-id $repo_id --build-definition-id ${vals[0]} --display-name "${vals[1]}" --path-filter "${vals[2]}")
        fi
    fi
done
```

To do this via a series of policy files, you'd have to write out a JSON file for each policy you wanted to apply. 

And that's were I'll leave it.