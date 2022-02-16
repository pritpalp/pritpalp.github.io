---
title: "Azure Devops Repos and Secret Scanning"
date: 2022-02-16 14:00:00 +0100
toc: true
toc_sticky: true
categories:
  - Azure
  - Azure Devops
  - Azure Repo
tags:
  - Blog
  - Azure
  - Security
excerpt: Secret Scanning your Devops Repos
header: 
  overlay_image: /assets/images/post-header.jpg
  caption: "Photo by [**Irvan Smith**](https://unsplash.com/@mr_vero?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [**Unsplash**](https://unsplash.com)"
  overlay_filter: 0.5
---
## Why?

When your team are committing code to shared repo's (and public sometimes), you really don't want them to commit sensitive data like API keys or passwords. If they do, it can sometimes be possible to [purge](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository) the offending data, but you never know if it's been captured by someone. Azure Devops Repos are a little different from Github, you can't purge something from history.

## Requirements

So, on my last project we needed to introduce something to stop the dev team from accidentally committing "secrets" to the repo. The repo was in Azure Devops, so the solution needed to work with that. You could use a tool that sits on the dev's machine and works with pre-commit hooks to scan before code is committed into the repo. This is probably the ideal solution, but does mean that the tool needs to work over all variations of dev machines and be maintainable (your config for what constitutes a secret may change over time).

## How

[Gitleaks](https://github.com/zricethezav/gitleaks) was the solution that we decided to use. A command line utility that I can run from the pipeline as part of the pre-merge process (build validation policy). Gitleaks is relatively simple, run the command within your repo and it will let you know if anything like a secret is present. There's a default config file in the repo that contains a large number of values that you may want to stop being committed to a repo like AWS access tokens, GitHub PAT, etc. It's being updated all the time - pipelines on my project are still using an older version without issue (there's been a big version change since I last updated and I've not spent any time evaluating the new version yet[^1]).

The pipeline to run the task is very simple, the GitLeaks executable is located on the build agent (we run this on a self hosted buildagent - as we need to deploy into an ASE). A command line script calling the executable with the relevant command options is all it takes. The pipeline pulls the code to check and the command line script runs GitLeaks. Any secrets detected cause the pipeline to fail and the output of the command can be seen.

```bash
"c:\utils\gitleacks\gitleaks.exe" --config-path=.gitleaks.toml --path=$(Build.Repository.LocalPath) --verbose --redact --no-git
```

Notable options here are the *config-path* and *no-git* options. The config path points to the Gitleaks config file containing the rules on what to search for. This covers a vast majority of secrets and some generic rules. There's an example config in the [repo](https://github.com/zricethezav/gitleaks/blob/master/config/gitleaks.toml). 

The *no-git* option ensures that only the specified directory is scanned. When you only have a few branches, the scan is reasonably quick. But once you've got 20+ branches...it can take a really long time to run - and do you want to scan all those branches every time you run this as part of a PR? Remember, GitLeaks will flag up stuff that you have resolved (unless you add those particular commits as exceptions). So running this *every time* you create a PR and having to check the output for *new* leaks would be very time consuming.

For our use, we also had to add in some exceptions - for example our unit test code directories. This was added in under the `[[allowlist]]` section as shown below (this also excludes node_modules, tests and bin directories).

```toml
  paths = [
    '''(.*?)(UnitTests|node_modules|Tests|bin)'''
  ]
```

You may also find the need to exclude specific values. For example, just the other day a dev committed a value similar to `ApiKey = "mySecretString"` in a config file, where mySecretString would be substituted for an *actual* value when the code ran. Now, this was being flagged up by GitLeaks as a *Generic API Key*.

```json
{
   "Description": "Generic API Key",
   "StartLine": 14,
   "EndLine": 14,
   "StartColumn": 5,
   "EndColumn": 28,
   "Match": "\"apikey\":  \"mySecretString\"",
   "Secret": "\"apikey\":  \"mySecretString\"",
   "File": "******",
   "Commit": "",
   "Entropy": 0,
   "Author": "",
   "Email": "",
   "Date": "0001-01-01T00:00:00Z",
   "Message": "",
   "Tags": [
      "key",
      "API",
      "Generic"
   ],
   "RuleID": ""
}
4:01PM WRN leaks found: 1
```

What I would have liked to do, would have been to write an exception under this rule that would ignore that string...but this proved to be very difficult to do - I just couldn't get the value to be excluded no matter what regex I used. I didn't what to ignore the whole file, so I ended up adding in another generic whitelist for the value *mySecretString*.

```toml
[[allowList]]
  paths = [
    '''(.*?)(UnitTests|node_modules|Tests|bin)'''
  ]
  regexes = [
    '''mySecretString'''
  ]
```

Not an ideal solution, but it worked. As always, getting the regex correct and in the correct place is a pain...

[^1]: A quick look at the instructions and the new version of GitLeaks introduces two options *detect* and *protect*. So my next update is not going to be as straight forward as my last!
