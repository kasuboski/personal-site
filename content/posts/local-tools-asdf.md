---
title: "Using asdf for Installing my Local Dev Environment"
date: 2022-07-29T12:07:36-05:00
draft: false
---

Azure CLI had a breaking change that `brew` autoupdated me to. Time to install differently.

<!--more-->

## What's asdf?
[asdf](https://asdf-vm.com/) claims to let you "manage multiple runtime versions with a single CLI tool". It was originally intended to replace `nvm`, `rvm` and such tools where you need multiple versions of `nodejs` and `ruby`. It's based on a plugin system though so there are many [things](https://github.com/asdf-vm/asdf-plugins) you can install using it.

The important part for me is that it lets you choose a specific version to use and even have multiple. I had (and continue) to use Homebrew to install most things on my mac and it will always try to update you to the latest version. This is usually what I want, but some things are finicky. The various cloud CLIs tend to have specific version ranges they work with so if you're not able to update everything you can end up in trouble.

For me, this manifested as the Azure CLI made a breaking change and suddenly my Pulumi code no longer worked. I wanted to rollback to the previous version, but with Homebrew this seemed to involve manually checking out a git repo and being very careful to run the correct incantation 🧙‍♂️.

I keep all of my config and tools in a [dotfiles](https://github.com/kasuboski/dotfiles) repo so manually updating something out of band kind of defeats the purpose.

## My Setup
I use [chezmoi](https://www.chezmoi.io/) to manage my config. It can read the files in my dotfiles repo and copy them to the correct place while also using templates to manage different machines. This way I can keep the various devices I use mostly in sync and starting from scratch can be accomplished with `sh -c "$(curl -fsLS https://chezmoi.io/get)" -- init --apply kasuboski`.

I currently still install most things with a Brewfile in that repo. I usually want the latest version of things so haven't had much of a reason to change it. However, some things that I was either installing with a script or I wanted to track versions of have moved to asdf.

The asdf plugins are installed in [this](https://github.com/kasuboski/dotfiles/blob/main/run_once_after_10-install-asdf-plugins.sh.tmpl) script that chezmoi calls. The versions are then pulled from a [.tool-versions](https://github.com/kasuboski/dotfiles/blob/main/dot_tool-versions.tmpl) file.

I don't really love the verbosity, but it accomplishes what I want and I can install different versions for different folders I'm in.

## Managing Versions
Pinning the version is nice to avoid breakage, but I generally want to stay on the latest unless there's a reason not to. To automate version discovery and upgrade I configured [Renovate](https://docs.renovatebot.com/).

Renovate has many built-in datasources and managers that might detect and manage your versions out of the box. There isn't built-in asdf support though. I found the [Regex Manager](https://docs.renovatebot.com/configuration-options/#regexmanagers) to be very simple to set up.

It needs to match some capture groups of the data Renovate needs, but after that it seems to be magic. I added a specific comment above each tool in my `.tool-versions` that controls where Renovate should look for new versions. It then matches the current version from the asdf format. Below is an example for something coming from Github Releases.

```bash
# renovate: datasource=github-releases depName=kubernetes-sigs/krew versioning=loose
krew 0.4.3
```

I then have a nice dashboard [issue](https://github.com/kasuboski/dotfiles/issues/2) that lists the dependencies Renovate knows about and is tracking. It will also show you any open PRs for a version bump.

## Next Steps
I might convert more tools over to asdf depending on how annoying the upgrade PRs become. I also want to make a container image so I can have a portable dev environment in many places including [Gitpod](https://www.gitpod.io/).
