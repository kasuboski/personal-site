---
title: "Add Multi-Arch Dependencies Easily"
date: 2020-11-19T11:02:45-06:00
draft: false
---

I wanted to build a multi-arch docker image for media transcoding. Time to get the dependencies from someone who already did it.

<!--more-->

## The Goal
I wanted a multi-arch image to run [mdhiggins/sickbeard_mp4_automator](https://github.com/mdhiggins/sickbeard_mp4_automator). I just needed the `manual.py` script not the radarr integration. The images published by mdhiggins are based on images like the linuxserver/radarr image and aren't multi-arch.

You can skip ahead to just see the Dockerfile at [kasuboski/manual-sma](https://github.com/kasuboski/manual-sma/blob/main/Dockerfile).

The main issue for making the image multi-arch is `ffmpeg`. In the [mdhiggins/radarr-sma](https://github.com/mdhiggins/radarr-sma) Dockerfile, it is always downloading the amd64 version of ffmpeg. This obviously won't go well for other architectures.

There are ffmpeg builds published for other architectures. You would just need to make sure to download the correct one.

## The Lazy Solution
It wouldn't be too bad to figure out which architecture is being built and then download the correct version of ffmpeg. It is one more script to maintain though.

I noticed the linuxserver repo has a multi-arch ffmpeg container already. They include a statically compiled ffmpeg so getting it into my image is as easy as copying the binary.

Dockerfiles have a `COPY --from=<image>` option. This lets you copy files from another image. That image will automatically be the correct one for your architecture (as long as the image supports it).

So instead of figuring out the correct architecture and downloading the corresponding ffmpeg. You can just add `FROM linuxserver/ffmpeg as ffmpeg` and then `COPY --from=ffmpeg /usr/local/bin/ff* /usr/local/bin/`.

This will copy both ffmpeg and ffprobe to the built container.

To build the image, I used the same method as my [building multiarch images post]({{< ref "build-multiarch-image" >}}). Basically, setting up docker buildx in GitHub Actions. You can see the workflow at [kasuboski/manual-sma](https://github.com/kasuboski/manual-sma/blob/main/.github/workflows/docker.yml).