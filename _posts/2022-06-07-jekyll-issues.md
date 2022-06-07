---
title: "Jekyll issues"
date: 2022-06-07 18:00:00 +0100
categories:
  - Jekyll
tags:
  - Blog
  - Jekyll
  - GitHub-Pages
excerpt: Fixing issues with starting Jekyll
header: 
  overlay_image: /assets/images/post-header.jpg
  caption: "Photo by [**Irvan Smith**](https://unsplash.com/@mr_vero?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [**Unsplash**](https://unsplash.com)"
  overlay_filter: 0.5
---

This site uses [Jekyll](https://jekyllrb.com/) on [GitHub Pages](https://pages.github.com/), which works quite well. The only problem I have is *every* time I go to write a new post and test it before pushing it onto the site...I run into errors starting up Jekyll on my localhost.

So, the normal process is to run updates for Ruby and the gems. Then check the versions in the Gemfile.lock file.

Updating Ruby (installed via [Homebrew](https://brew.sh/) on a Mac):

```bash
-> brew update && brew upgrade
...
-> ruby --version
ruby 3.1.2p20 (2022-04-12 revision 4491bb740a) [x86_64-darwin21]
```

Update the Gems and check the versions:

```bash
-> gem update --system
...
-> bundler --version
Bundler version 2.3.15
-> gem --version
3.3.15
-> jekyll --version
jekyll 4.2.2
```

The latest issue I had was:

```bash
-> bundle exec jekyll serve
/usr/local/opt/ruby/bin/bundle:25:in `load': cannot load such file -- /usr/local/lib/ruby/gems/3.1.0/gems/bundler-2.2.25/exe/bundle (LoadError)
    from /usr/local/opt/ruby/bin/bundle:25:in `<main>'
```

And the issue here...the Bundler version in the Gemfile.lock file was not the same as the updated version installed, so they didn't match.

```bash
-> bundler --version
Bundler version 2.3.15
-> tail -n2 Gemfile.lock
BUNDLED WITH
   2.2.25
```

Changing the version to match the latest resolved the issue and allowed me to start up Jekyll.

The nuclear option is to remove the Gemfile.lock and regenerate it...but this could have issues with incompatible versions of gems.

```bash
-> rm Gemfile.lock
-> bundle install
```
