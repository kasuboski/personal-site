---
title: "Building Multiarch Images"
date: 2020-05-17T17:28:55-05:00
draft: false
---

I wanted to use [fathom](https://github.com/usefathom/fathom) on my Raspberry Pi k3s cluster, but they didn't publish a compatible ARM image. Not to be deterred I forked the repo and built my own.

<!--more-->

## How does one build for multiple architectures easily
Docker has a tool called buildx. It acts as a frontend to buildkit and allows building images for multiple platforms at once.

I followed [this](https://www.docker.com/blog/multi-arch-build-and-images-the-simple-way/) guide to create a GitHub Actions workflow to build it.

## GitHub Actions it up
You can skip ahead and look at my final workflow [here](https://github.com/kasuboski/fathom/blob/master/.github/workflows/docker.yml).

There was already a buildx action. The workflow ends up seemingly simple.

* Checkout the repo
* Set up buildx
* Login to DockerHub
* Build for linux/amd64,linux/arm/v7,linux/arm64

That last part includes the platforms I would care about (for now). amd64 is for your run of the mill computer, arm/v7 being what is on my Pi thanks to 32-bit Raspbian, and arm64 being a potential if I switch the OS on my Pi.

The workflow works well, but every run so far was between 14mins and 30mins... not exactly fast.

## Moving Forward
There have been a number of images that aren't multi-arch that I want to run. I may look into forking and building those, but it would be nice to have a simpler solution with me not managing it.

I could contribute to [multi-arch-images](https://github.com/raspbernetes/multi-arch-images) instead. They seem to be doing something similar but just copying the Dockerfiles needed.