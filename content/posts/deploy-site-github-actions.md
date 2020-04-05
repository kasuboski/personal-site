---
title: "Deploying this site with GitHub Actions"
description: "This site is built with Hugo and deployed to GitHub Pages using GitHub Actions"
date: 2020-04-05T16:18:12-05:00
author: "Josh"
draft: false
---

[GitHub pages](https://pages.github.com/) is a free static hosting provider that unsurprisingly works well with a git workflow. It enables `git push` to deploy type workflows. This site itself is a static site built with [hugo](https://gohugo.io/) and deployed to GitHub Pages using GitHub Actions.
<!--more-->

## GitHub Pages Repo
GitHub Pages expects that you have a repo named something like `user.github.io` that has static files you want deployed. However, if your site requires any sort of building this doesn't really work for you (at least if you want to automate the build step).

One way around this is to treat your GitHub Pages Repo as the output for your build. You create a separate repo that just has the built static files. You can see the repo for this site at [kasuboski/kasuboski.github.io](https://github.com/kasuboski/kasuboski.github.io).

You'll notice it just has static html, css, etc. I write the content for this site in markdown however, using Hugo.

## Code Repo
The actual content is stored in a separate repo that I'm referring to as the code repo. The repo for this site is [kasuboski/personal-site](https://github.com/kasuboski/personal-site). This repo should look a little more familiar to anyone who has used Hugo.

The content is under `content/posts` and is written in markdown. This is the repo that I actually modify and have checked out locally. If I wanted to create a new release of it manually I would build the site and push it to the GitHub Pages Repo with the commands below.

```
hugo --minify
cp ./public ../kasuboski.github.io/
cd ../kasuboski.github.io
git commit -am "cool new post"
git push origin master
```

## Automatic Deploy with GitHub Actions
I don't really want to mess with multiple repositories locally, especially when one of them is essentially machine generated. GitHub Actions can build the site with Hugo and then push it to the GitHub Pages Repo for me.

The workflow file for this can be found [here](https://github.com/kasuboski/personal-site/blob/master/.github/workflows/gh-pages.yaml) if you just want to copy it.

The basic steps are: check out the code repo, build the site with hugo, push the files to the GitHub Pages Repo, and notify me of the status using [Pushover](https://pushover.net).

It relies heavily on already available actions. The only prerequisite setup is to make a deploy key for the GitHub Pages Repo and to register an app with Pushover.

You can find a walkthrough of creating a deploy key [here](https://github.com/peaceiris/actions-gh-pages#%EF%B8%8F-create-ssh-deploy-key).

Configuring a Pushover app is [here](https://pushover.net/apps/build). You'll need the app token and user token. I added both as secrets on the Code Repo.

Now anytime I push a change to the code repo, GitHub Actions will generate the files, update the GitHub Pages repo and send me a push notification with the status.