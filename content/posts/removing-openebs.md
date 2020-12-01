---
title: "Removing OpenEBS"
date: 2020-12-01T16:37:25-06:00
draft: false
---

My OpenEBS install seemingly stopped working. 🔥

<!--more-->

## Discovery
I actually didn't really notice it had stopped working until two days later. I have monitoring set up for some sites, but it was only testing that the page loaded, not that data loaded. Lesson learned there I guess.

The pods showed as having the storage attached, but it didn't work. Looking for the status of the various OpenEBS objects revealed that it was not healthy. I ran a couple commands that were mentioned in the kubernetes slack, but couldn't get it back.

Luckily, I had recently backed up the data. You can see how at [copy files using kubectl]({{< ref "kubectl-cp" >}}).

## Moving On
I decided I wanted to simplify the storage while I was redoing it anyway. In manually mounting my usb drives, I discovered that one was now permanently readonly. This was almost certainly the problem with OpenEBS, but I'd already uninstalled everything 🤷‍♂️.

I'm now using the [local-storage-path](https://github.com/rancher/local-path-provisioner) provisioner that's included with k3s. One Pi has the same model of microcenter usb drive as the one that failed and the other a [Corsair Drive](https://kit.co/kasuboski/homelab/corsair-flash-voyage) that I had seen benchmarked as fast specifically on a Raspberry Pi.

Apps with storage are specifically pinned to either the slow disk node or the fast disk node. The apps on the fast disk node had issues with the OpenEBS and slow usb drive setup. They now perform significantly better.

Just a note, that uninstalling OpenEBS took me quite some time... I think maybe I did it in the wrong order because basically all resources got stuck with a finalizer that I then had to manually remove.

I'd like to try OpenEBS again, but maybe in a setup where I actually have three storage nodes and faster disks.

I'm excited to try Prometheus again, since previously the storage was just too slow. Although, I have plans to run it with Thanos so that most of the data will be in Minio (backed by my nas).