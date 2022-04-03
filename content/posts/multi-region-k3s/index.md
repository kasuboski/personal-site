---
title: "Multi-Region K3s"
date: 2021-11-11T15:56:57-06:00
draft: false
---

I had stood up a cluster in the cloud before moving. Now, I've added some nodes at home to round it out. Maybe I'm just in it for the buzzwords.

<!--more-->

## Why not just the cloud?
As previously mentioned in [moving my cluster to the cloud]({{< ref "moving-cluster-to-the-cloud.md" >}}), I run a VM for the control plane and then a free Oracle ARM 4x24 VM as a worker. You might think this is more than enough (and you'd probably be right), but there are some things that make more sense to run at home.

I currently run media services and backups at home. I also hope to mess around with more home automation things in the future. The media consumption is nice to be local, but it also saves on cloud network transfer costs. Overall, the cloud worker VM is going to be much more reliable than the small form factor PCs that make up my home region.

## How's it actually work?
I install [Tailscale](https://tailscale.com/) on all of the machines. This lets them easily connect to each other as if they were on the same local network. The kubernetes API server only binds to the Tailscale interface so all of the workers are able to reach it. After setting this up, it mostly just works™️.

The nodes are separated into a cloud and home region. Each region is broken down into zones for the cloud provider or location. That ends up cloud-oracle, cloud-racknerd, home-austin, home-wisconsin region-zone pairs. I'm then able to use those labels for scheduling decisions.

So far I'm only using the zones for austin and wisconsin to pin my media options to the respective machines. Other things, including my RSS reader, are able to float between regions.

## The Setup
The cluster is still managed with GitOps from the repo [kasuboski/k8s-gitops](https://github.com/kasuboski/k8s-gitops). I'm using ArgoCD to apply the manifests, but might mess with other options.

I recently changed the secrets management from [SealedSecrets](https://github.com/bitnami-labs/sealed-secrets) to [Doppler](https://www.doppler.com/). There wasn't anything wrong with SealedSecrets, but it felt less magic since I had to make sure to manage the keys and reencrypt on changes. I have a script under `hack/` in the repo that manages importing the correct Doppler project token. After creating their `DopplerSecret` CRD, the secrets then just show up.

Previously, I had run a separate VM to act as the entrypoint to the cluster. Now I use a loadbalancer from Oracle that points to the free ARM VM there. The DNS then points to this loadbalancer. I still want to add a separate ingress locally so I can avoid always going out and back in, but haven't gotten around to it.

I also still have 4 Raspberry Pis that I haven't setup yet since moving. The general layout is as below.

 {{< figure src="homelab.png" link="homelab.png" alt="Homelab Diagram" >}}

## Further Improvements
I have a lot of the setup documented in [kasuboski/home-infra](https://github.com/kasuboski/home-infra), but realized as I was setting up the machine in Austin that it leaves a lot to be desired. In particular, the setup for storage had me looking back through the command history of the Wisconsin machine to figure out what I had done.

I'm thinking to make a tool that will set this up or at least walk me through it. The `home-infra` repo needs to be cleaned up a little as well. It still contains instructions for multiple past iterations of my homelab.