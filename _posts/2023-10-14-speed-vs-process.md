---
title: "How processes can reduce development and deployment speed"
date: 2023-10-14 09:00:00 +0100
toc: true
toc_sticky: true
categories:
  - 
tags:
  - Agile
  - CI/CD
excerpt: Process Vs Speed
header: 
  overlay_image: /assets/images/post-header.jpg
  caption: "Photo by [**Irvan Smith**](https://unsplash.com/@mr_vero?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [**Unsplash**](https://unsplash.com)"
  overlay_filter: 0.5
---
I want to share with you some thoughts on how processes can sometimes get in the way of delivering value to our customers. 

I've been working on a project that made a huge improvement in reducing the time to deploy our code to production. We used to work in two-week sprints and release every six weeks. That meant our code was sitting idle for four weeks before it reached the users. We decided to change that and release every sprint, so our code was only one sprint behind. 

Sounds great, right? Well, there was a catch...we had to decide what stories we were going to release *before* the sprint was over. 

Why? Because of the Change Advisory Board (CAB). The CAB is a group of people from different departments who review and approve the changes that each project wants to make in production. They need 10 day's notice to do that. No exceptions (unless it's an emergency fix, but then you have to fill out more forms). So, we had to guess what we would finish in the sprint, and hope nothing went wrong. 

But things always go wrong. Sometimes we had stories that we couldn't finish in time. Then we had to remove them from the release (and update the paperwork). Sometimes we finished faster and had extra stories. Then we had to wait until the next release to merge them (and risk conflicts). Sometimes we finished testing the release after the sprint was over. That had a knock on effect on the other activities related to the release. 

What else goes into a release you ask? Well, we have pipelines to create to do the deployment. These need to be added to a runbook for the release with any other steps that need to be executed (one off steps etc.). Then we need a meeting with interested parties to talk through the runbook and agree that everything was covered in enough detail (rollback steps etc.). A dress rehearsal in the PreProduction environment. Testing in the PreProduction environment. Appending any changes needed afterwards, and another meeting before Production release. Before finally the Production release. 

You see the problem? Our process was slowing us down instead of helping us. One person could spend most of a sprint dealing with the release rather than working on stories especially if you consider other factors like holidays/sickness...And let's not forget the other Scum ceremonies that will take up time each sprint as well, sprint planning, sprint review, sprint retrospectives, backlog refinement and daily scrum as well as the ad-hoc meetings that require your input... 

So, how can we get these processes changed? Well, that's the big question. My current situation doesn't have a simple solution. There is reluctance to change the tried and tested existing process. The fact that we have managed to fit our increased release cycle into the process works against us - why change the process if it works? But we've only made it work by making it the number 1 priority to the detriment of other tasks. 

Continuous Delivery would never be possible in the current situation. We are limited to dev/test environments being deployed automatically because those are internal test environments. The fact that there just isn't trust in the team's ability to not break production makes it impossible to automatically progress to pre-production or production. Organisations love processes and meetings. Everything is geared towards lowering risk. Nobody wants to be held responsible for causing an incident. 

When adopting strategies like agile/scrum/kanban/lean/.../ [the next big thing] the key thing is that the organisation as a whole needs to buy into it. Having a fast moving dev team limited by historic approvals processes can only work up to a certain point - eventually you will reach frustration points and limits. 

Let's take blameless postmortems for example. They are great when they work. You can gather all relevant people together and drill down into an incident and walk away with actions to mitigate re-occurrence and a thorough understanding of what happened. But that only works if it is truly blameless. Management needs to trust the process and the people. 

At a former company we were having a sprint retrospective and had been joined by *someone with power* (referred to as X). The retro was a simple "what went well, what didn't, what can we improve" style event. We'd barely started the process when X decided to give their input. When I say input...I mean call out individuals for their perceived faults during the sprint. The scrum master lost control of the meeting and let X lead, as X was higher in the organisational chain than them. The tech leads kept quiet too, so as not to shoulder any blame or draw focus on themselves. The rest of us were left to defend our actions in the sprint. And what were the complaints? A few stories slipped into the next sprint as more time had to be spent on other stories. There were a few technical issues as well. Pretty much standard stuff. That one meeting showed that management had no trust in the processes or the team. Rather than being a productive tool, this one meeting destroyed team morale. 

So, to get things changed there needs to be trust, in people and processes. 