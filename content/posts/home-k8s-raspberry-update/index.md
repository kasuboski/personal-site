---
title: "Homelab Raspberry Pi Kubernetes Cluster Current State"
description: "The k3s cluster is alive and well"
date: 2020-05-10T12:20:48-05:00
draft: false
---

The cluster is alive and well and exposed to the internet :scream:

<!--more-->

In a previous [post]({{< ref "home-k8s-raspberry-pi.md" >}}), I put out my plan for the cluster. I mostly followed along, but did end up getting [this](https://www.amazon.com/gp/product/B00A128S24) switch and [these](https://www.amazon.com/gp/product/B003L1AET2) ethernet cables.

I ended up with this set-up which is described in more detail below.

{{< figure src="traffic-diagram.png" link="traffic-diagram.png" alt="Traffic Diagram" >}}

## Hardware and Monitoring
The cluster is set up with Raspbian Lite on 3 Raspberry Pi 4 4gb boards. Raspbian Lite seems to not support 64bit (yet) so I may be changing the operating system. The cluster has the monitoring stack from carlosedp's [repo](https://github.com/carlosedp/cluster-monitoring). I changed some things (mainly the ingress) in my forked [repo](https://github.com/kasuboski/cluster-monitoring).

This deploys some nice Grafana dashboards to see the state of the cluster as seen below. `blueberry` is the master node so hovers around 768m cpu cores and 1285Mi memory usage.

{{< figure src="cluster-metrics.png" link="cluster-metrics.png" alt="Cluster Metrics" >}}

## Ingress
This dashboard is exposed outside the cluster using [ingress-nginx](https://kubernetes.github.io/ingress-nginx/deploy/baremetal/#over-a-nodeport-service) with NodePorts. These nodeports are load balanced by an haproxy running in a container on my existing Raspberry Pi 3B+. This is a single point of failure for ingress :cold_sweat:, but should be good enough for now.

The Ingress is secured using [vouch-proxy](https://github.com/vouch/vouch-proxy) with the ingress-nginx auth [annotations](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/). It uses the [IndieAuth](https://indieauth.net/) backend so I can login using this site as an identity.

## Expose to the world
The final piece of the puzzle to expose cluster services to the internet is a Linode [nanode](https://www.linode.com/products/nanodes/). I chose Linode largely because I had a credit... It's really just providing a public ip address that can forward to my network. I didn't set up port forwarding to my network instead the haproxy Raspberry Pi and the Linode VM are running [tailscale](https://tailscale.com/).

Tailscale allows the nanode to connect to my Raspberry Pi with a WireGuard connection through my crazy NAT situation. The nanode is also running an haproxy on ports 80 and 443 on the public ip interface and then proxying that to the tailscale ip of the Raspberry Pi.

I changed the DNS of `*.joshcorp.co` to point to the nanode. Now that `*.joshcorp.co` resolves to my cluster on the internet, I can easily use [cert-manager](https://cert-manager.io/docs/) to get a certificate from LetsEncrypt.

After all of that, I can access `grafana.joshcorp.co` outside of my network while authenticating using [www.joshkasuboski.com](https://www.joshkasuboski.com).

## What's Next
I'm pretty happy with the current state. My next move will be making sure everything is tracked in git and can be reproduced. From there I can start deploying.