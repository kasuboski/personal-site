---
title: "Homelab Raspberry Pi Kubernetes Cluster"
description: "Setting up a k3s cluster at home on Raspberry Pis"
date: 2020-05-02T16:17:39-05:00
draft: false
---

A platform to deploy my self-hosted services onto with almost $0 recurring costs.

<!--more-->

Previously, I was running a few things on a Raspberry Pi 3B+. Mainly, [Blocky](https://github.com/0xERR0R/blocky) as an ad-blocking local dns cache and some adventures with [Home Assistant](https://www.home-assistant.io/). A Kubernetes cluster will enable me to easily set up new things and have proper monitoring as well.

## Hardware Acquisition
I bought 3 [Raspberry Pi 4 4GB](https://www.canakit.com/raspberry-pi-4-4gb.html) boards from Canakit. I also bought a pack of 5 [SD cards](https://www.amazon.com/gp/product/B07NP96DX5). I got a [6 port usb charging station](https://www.amazon.com/gp/product/B01NAG3V8E) and USB-A to USB-C [cables](https://www.amazon.com/gp/product/B01JRY0VE4) to power them.

I have 3 spare ethernet ports on my router so I will be using that directly for now. In the future, I may get a switch for them to connect to.

## Setup
I'm planning to install Raspian Lite and use [k3sup](https://github.com/alexellis/k3sup) to install k3s. I looked into using [k3os](https://github.com/rancher/k3os) directly, but it seems it's still not the easiest for a Raspberry Pi [issue](https://github.com/rancher/k3os/issues/309).

I will try to manage the cluster in a [GitOps](https://www.weave.works/technologies/gitops/) style. As such, I will disable installing the Traefik Ingress Controller and Service Load Balancer by default and instead install them separately (or choose a different option).

## Networking
I will start out with the default flannel with vxlan, but am considering the wireguard backend. I would like to be able to access some services in the cluster using something like [Tailscale](https://tailscale.com/) as well.

This might involve running separate ingress controllers like [here](https://medium.com/@carlosedp/multiple-traefik-ingresses-with-letsencrypt-https-certificates-on-kubernetes-b590550280cf).

## Monitoring
I'm going to deploy the standard `kube-prometheus` set up with the Prometheus Operator using this [repo](https://github.com/carlosedp/cluster-monitoring). It has k3s specific settings.