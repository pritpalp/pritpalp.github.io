---
title: "Creating this blog"
date: 2022-07-01 18:00:00 +0100
toc: true
toc_sticky: true
categories:
  - Jekyll
tags:
  - Blog
  - Jekyll
  - GitHub-Pages
excerpt: The process of creating this blog
header: 
  overlay_image: /assets/images/post-header.jpg
  caption: "Photo by [**Irvan Smith**](https://unsplash.com/@mr_vero?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [**Unsplash**](https://unsplash.com)"
  overlay_filter: 0.5
---

## Where to host
One of the first decisions when starting a blog is what your going to use to blog. There are many options including [Wordpress](https://wordpress.com/), [Medium](https://medium.com/), [Gatsby](https://www.gatsbyjs.com/), [Jekyll](https://jekyllrb.com/), [LinkedIn](https://uk.linkedin.com/), etc

The choice of what you use could be made simple by where and how you are going to host your blog. LinkedIn and Medium can be used to write and host your pages with no need to think about hosting costs. Just log in and write.

I wanted something more. I narrowed my choices down to hosting on Azure or AWS - platforms I use day in day out for work. A chance to play about with some interesting technology.

## What to use
Gatsby was the engine that I initially started to look at. Gatsby is a ReactJS framework that you can use to build your blog. It sounded like a good opportunity to learn ReactJS and blog at the same time…

A few weekends later and I gave up. The amount of work that would have been needed to create the initial site and them maintain it just seemed to be excessive. I’d spent hours creating a test site and I was finding the process way too time consuming.

[Azure Static Web Apps](https://azure.microsoft.com/en-us/services/app-service/static/#overview) were just in preview (long since passed into general release now) when I started looking and seemed to be interesting. The free tier looked as though it would be enough for what I wanted...but as the site would be in a GitHub repo anyway...[GitHub Pages](https://pages.github.com/) seemed to be a simpler option. 

Jekyll is the engine behind GitHub Pages, you write in [Markdown](https://www.markdownguide.org/basic-syntax/), same as you do when adding readme’s to repos. A bit of searching around and I found it’s relatively simple to point your own domain at your GitHub Pages site. There are also many [templates](https://jekyllthemes.io/free) freely available to use to format you blog. Hosting is free…

## GitHub Pages
Without using anything special, you can create a very simple static blog:
1. Create a Github account if you don't have one - your username will be in the name of the blog
2. Create a repository with the name username.github.io (replacing username with your username)
3. Create a index.md and add some content...
4. Commit and push the repo back to GitHub
5. Select the publishing source - Go to your GitHub page, select the repo and click on the Settings. From here, click the Pages item under Code and Automation. Select the branch and root that you want to be the source of your site. 

After a few minutes, you should be able to browse to https://username.github.io - replacing username with your username.

If you add more pages, or a directory you'll be able to visit them the following way.
A new file in the root called page2.md would be available via https://username.github.io/page2.html
A new file in a directory called *subdirectory*, page3.md would be https://username.github.io/subdirectory/page3.html

## Jekyll
If your happy with hosting in GitHub Pages, you can easily add some design touches and features with [Jekyll](https://jekyllrb.com/). GitHub has a little sub site on how you can get started using Jekyll [here](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll). I won't go into detail here to reproduce that information, but its pretty straight forward.

Jekyll uses [YAML](https://yaml.org/) config files. To use a theme, you just configure the *remote_theme* setting and anything else that is required. Fill in your details in the *_config.yaml* and your pretty much good to go with writing your articles.

I'm currently using the [MinimalMistakes](https://mmistakes.github.io/minimal-mistakes/) theme. All I need to do to create new articles is drop a new file into the *_pages* directory. Once the site rebuilds after my commit, the new article is available to view.

As Jekyll is for static sites, adding things like comments can be done by using external components. At some point in the future I may look into this...
