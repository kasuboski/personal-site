---
title: "Server Rack Beginnings"
date: 2020-07-05T13:58:33-05:00
draft: false
---

My network setup was starting to be a mess of a corner. The natural thing to do is to get a server rack to contain everything :wink:

<!--more-->

## The Mess
{{< figure src="network-mess.jpg" link="network-mess.jpg" alt="A Jumbled mess of cables" width="300px" >}}

The network corner was already a mess, but adding my [Raspberry Pi cluster]({{< ref "home-k8s-raspberry-update" >}}) definitely didn't help.

You can see the following items if you look closely:
* Linksys Router
* 5-port switch for the Raspberry Pi cluster
* Raspberry Pi Cluster
* Raspberry Pi 3B+
* Hue Hub

Not pictured is my ISP modem and the USB power hub for the Pi cluster.

## Solution?
I have been following [r/selfhosted](https://www.reddit.com/r/selfhosted/) and [r/homelab](https://www.reddit.com/r/homelab/) for a while. It was perhaps a mistake.

I now aspire to the setups such as [this](https://hydn.dev/homelab/) and [this](https://tynick.com/blog/06-06-2019/my-humble-homelab-with-raspberry-pi-rack/).

The first link has a nice monitor on top and the second has a rackmount Pi enclosure (which I have purchased the bracket for).

I bought a [15U open enclosure rack](https://www.ebay.com/itm/15U-4-Post-Open-Frame-Server-Rack-Enclosure-19-Adjustable-Depth/151584303195?ssPageName=STRK%3AMEBIDX%3AIT&_trksid=p2057872.m2749.l2649). It's maybe not the most efficient solution, but definitely feels ... cool.

## Setting up the Rack
The rack arrived with some assembly required. It took me an hour to put it together with the help of a magnetic level.

{{< figure src="rack-box.jpg" link="rack-box.jpg" alt="The server rack some assembly required" width="300px" >}}

{{< figure src="assembled-rack.jpg" link="assembled-rack.jpg" alt="Assembled rack" width="300px" >}}

I got everything into a [shelf](https://www.ebay.com/itm/Cantilever-Server-Shelf-Vented-Black-Shelves-Rack-Mount-19-1U-12-300mm-Deep/152062041884?ssPageName=STRK%3AMEBIDX%3AIT&_trksid=p2057872.m2749.l2649) and I'm not convinced it looks all that much better. The messy desk obviously isn't helping :cry:.

{{< figure src="completed-rack.jpg" link="completed-rack.jpg" alt="Completed rack" width="300px" >}}

## Moving Forward
The rack may not have immediately fixed my issues, but I'm planning to either replace or augment my set up to be rackmountable.

My first thing to rackmount is the Pi Cluster. I'm going to use something like [these 3D printed mounts](https://www.kaibader.de/3d-printed-raspberry-pi-rack-mount-with-heat-sink-passive-cooling/) to put the Pis in the rack.

I'm also looking at getting a 4U case to transfer my desktop to and switching to a rackmounted switch and router.

In the meantime the Pi cluster is pretty photogenic.

{{< figure src="artsy-pi.jpg" link="artsy-pi.jpg" alt="Artsy Pi" width="400px" >}}