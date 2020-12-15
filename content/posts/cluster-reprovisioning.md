---
title: "Re-provisioning the Cluster"
date: 2020-12-15T12:29:47-06:00
draft: false
---

The SD card died in the server node of my cluster 😢. Time to start again.

<!--more-->

## Tragedy
The SD card in blueberry died taking down the kubernetes api with it. The workloads were still running so it didn't matter too much, but I couldn't do anything new.

I didn't have a backup for the SD card... so was going to have to start again. I took the opportunity to try out some new things. I had been wanting to try out Ubuntu on the Raspberry Pi since they recently announced official support.

My thought with Ubuntu is to standardize more on the various machines I run. Previously, Raspbian had to have its own process for all automations.

## Re-provisioning
After deciding to go with Ubuntu, I needed to flash the sd cards with ubuntu and reinstall k3s. I used a pre-installed image of Ubuntu so was able to have them boot headlessly using cloud-init.

After flashing, I could edit the cloud-init files so the pi would come up with the correct hostname and my ssh keys. I used this [repo](https://github.com/billimek/homelab-infrastructure/tree/master/k3s/arm64) from [billimek](https://github.com/billimek).

## Installing k3s
I decided to install [cilium](https://docs.cilium.io/en/v1.9/gettingstarted/k3s/) as the cni instead of the bundled flannel. I initially wanted to use the multi-cluster features of cilium, but that seems to require running etcd separately... so I will have to figure out if that's worth running on the Raspberry Pis. I was able to use the Hubble UI to have real-time telemetry info. In the future, I want to use the Cilium cluster-wide network policies.

Cilium on k3s requires disabling the built-in flannel. I had to use the deprecated `--no-flannel` since `--flannel-backend=none` didn't seem to be supported. I also had to manually install the loopback cni plugin from [containernetworking/plugins](https://github.com/containernetworking/plugins).

## Next Steps 🦶
I started making a new `home-infra` repo where I'll start putting the steps (hopefully automated). Currently, just doing everything manually wasn't all that bad. Getting the stateless workloads running again was basically reinstalling ArgoCD and then pointing it at my gitops [repo](https://github.com/kasuboski/k8s-gitops).

One thing that was a little troublesome is remembering to label the node corresponding to the fast and slow storage. It was nice that my workloads requiring storage did land on the same node again, but it would be better for this to happen automatically. OpenEBS could probably help with this using the automatic detection of block devices and then letting you map that to storage classes.

Manually setting up the folders for the [local-path-provisioner](https://github.com/rancher/local-path-provisioner) is pretty convenient though. I could probably just use [local persistent volumes as well](https://kubernetes.io/blog/2019/04/04/kubernetes-1.14-local-persistent-volumes-ga/) 🤷‍♂️.