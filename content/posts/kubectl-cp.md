---
title: "Copy files to and from Kubernetes Pods with kubectl"
date: 2020-11-05T20:46:06-06:00
draft: false
---

I wanted a simple backup of some OpenEBS volumes. Why not just copy them out.

<!--more-->

## Why bother
I run a number of things on my [Raspberry Pi kubernetes cluster]({{< ref "home-k8s-raspberry-pi" >}}) that require storage. The main one I actually care about is my [fathom](https://github.com/usefathom/fathom) instance.

It has the website analytics for my site [joshkasuboski.com](https://www.joshkasuboski.com). It's certainly not the end of the world to lose this information, but I'd really rather not. Previously, I had tried to get [velero](https://velero.io/) setup.

I tried to use both the OpenEBS specific plugin and the generic restic. The OpenEBS one failed because my setup couldn't take snapshots. My USB drives are pretty slow so I just chalk it up to that. The restic option seemed to work, but it didn't actually restore data in my test. In addition, it kept exceeding my [backblaze](https://www.backblaze.com/) s3 api limits.

Anyway, I just wanted something simple before I upgraded OpenEBS. Then I found the `kubectl cp` command.

## Backing up the volumes
I manually backed up all of my persistent volumes. This is only five items for me, but could get out of hand quickly. It was really as simple as running the command for each and storing the files locally.

Copying files from a pod to your machine
`kubectl cp <namespace>/<podname>:/mount/path /local/path`

I did this for all paths I cared about. If something goes wrong you can always copy the files back to the pod. It's the same command just swapping source and destination.

Copying files to a pod from your machine
`kubectl cp /local/path <namespace>/<podname>:/mount/path`

I still hope to get OpenEBS snapshots working in the future, but this worked great for now.