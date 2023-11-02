---
title: "Homebrew on the Mac"
date: 2023-03-17 09:00:00 +0100
toc: false
toc_sticky: false
categories:
  - Mac
tags:
  - Blog
  - Mac Config
excerpt: Homebrew and sudo
header: 
  overlay_image: /assets/images/post-header.jpg
  caption: "Photo by [**Irvan Smith**](https://unsplash.com/@mr_vero?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [**Unsplash**](https://unsplash.com)"
  overlay_filter: 0.5
---
This week I had to set up a new machine for work. My old one was barely limping along having to be constantly tethered to a wall socket these past three years. I always find it interesting setting up something new. It's amazing how many apps and files can gather over time. I had apps that I'd used for one specific thing years ago, just sitting there taking up space. Visual Studio Code was filled with plugins/extensions that I don't remember ever installing or using.

But the most important thing was that, initially, I'd had admin/sudo access on my old Mac. I'd installed various apps through [Homebrew](https://brew.sh/) over time. Don't know what Homebrew is? Well, it's simply a package manager for MacOS (and Linux if you want). Need to run [Go](https://go.dev/) on your Mac, you can go to the Go homepage, find the correct download and install it yourself...or just install Homebrew and type `brew install go`. When you need to update it, you simply `brew update && brew upgrade`. No need to find installation packages yourself if you can use Homebrew to do it instead. And that one update and upgrade command will update *everything* Homebrew has installed, no more having to individually update apps.

Once admin access had been removed, I found that I was unable to update some Homebrew apps without having to raise tickets with IT. This left me with a year old install of Powershell, a two year old install of Dotnet that had broken at some point, etc. Not an ideal situation, but I could live with it.

Now with the new Mac, I was starting without admin access at all. So the first question I had was "How can I install Homebrew without `sudo`?". A bit of digging around and I found an answer. You can clone the repo into a location of your choosing and then set up some environment variables and cross your fingers. Homebrew writes into `/usr/local` or `/opt/homebrew` or `/home/linuxbrew.linuxbrew` (apparently the type of machine determines [which](https://docs.brew.sh/Installation#installation)). You need to have `sudo` to be able to create the directories/files in those locations, so if you can't supply a valid password, the install will fail. If you set the `HOMEBREW_PREFIX` environment variable, this is used instead of those prefixes.

Here are the commands I used:

```bash
xcode-select --install
git clone https://github.com/Homebrew/brew homebrew
export "HOMEBREW_PREFIX=/Users/p/opt/homebrew" >> ~/.zshrc
export "PATH=$PATH:/Users/p/homebrew/bin:$HOMEBREW_PREFIX/bin" >> ~/.zshrc
source ~/.zshrc
brew update
```
After that, Homebrew was on my Mac - no sudo needed. There is a warning that installing like this is *unsupported* on the Homebrew installation page. But we all know that we ignore stuff like that until something goes wrong...who can be bothered to [RTFM](https://en.wikipedia.org/wiki/RTFM#:~:text=RTFM%20is%20an%20initialism%20and,forum%2C%20software%20documentation%20or%20FAQ.)?

Note that the installs do seem to take longer than before - looks like the source is downloaded and then built on your machine rather than using prebuilt binaries...not that brew was ever that fast to begin with.