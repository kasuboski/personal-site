---
title: "Connect Your Home to the Cloud with Tailscale"
date: 2020-08-24T11:05:17-05:00
draft: false
---

I set up my Raspberry Pi cluster to be accessible from the internet without configuring a port-forward on my router.

<!--more-->

## Tailscale
[Tailscale](https://tailscale.com/) will create a private network using [Wireguard](https://www.wireguard.com/). Wireguard isn't really that difficult to configure on its own, but you do have to manually generate and distribute keys. Tailscale will take care of that for you and they also have some [fallbacks](https://tailscale.com/blog/how-nat-traversal-works/) for difficult networks. It doesn't look like any of my nodes are using a fallback option based on the dashboard.

Setting up Tailscale is as easy as installing it and running `tailscale up`. Until recently, this required you to login interactively. Tailscale now supports [pre-authenticated](https://tailscale.com/kb/1068/acl-tags#pre-authenticated-keys) keys which means you can automate the setup.

## Installing on Raspberry PIs
I made [kasuboski/tailscale-install](https://github.com/kasuboski/tailscale-install) to automate the installation and start of Tailscale on Raspberry PIs. I plan to expand it to work on more varied platforms in the future.

It's a [PyInfra deploy](https://pyinfra.com/) that basically just adds the package and runs `tailscale up` with a key sourced from the environment. I was able to add my Raspberry Pi cluster to the network in around 5 minutes, using this.

## Exposing to the internet
My cluster ingress is now slightly different than described [here]({{< ref "home-k8s-raspberry-update" >}}). Traffic from the Linode now goes directly to the Kubernetes nodes on the port exposed by the nginx-ingress controller. This just removes the extra hop that was initially an internal haproxy running on a different Raspberry Pi.

{{< figure src="tailscale-diagram.png" alt="Network Diagram" >}}