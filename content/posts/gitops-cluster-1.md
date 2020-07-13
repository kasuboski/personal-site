---
title: "GitOpsing the cluster"
date: 2020-07-12T18:40:37-05:00
draft: false
---

I kept track of how I set up my [Raspberry Pi cluster]({{< ref "home-k8s-raspberry-update" >}}) along the way, but hadn't committed it to git. Today that changed.

<!--more-->

## GitOps and the repo
If you're not familiar with GitOps, the people at weaveworks have a nice [article](https://www.weave.works/technologies/gitops/).

I pushed my setup to [kasuboski/k8s-gitops](https://github.com/kasuboski/k8s-gitops). It should have everything that's deployed on my cluster.

Each folder at the top level is basically a namespace currently. The ingress folder does contain the `ingress-nginx` and `cert-manager` namespaces.

I hadn't pushed it earlier because I still haven't figured out my strategy for secrets. For now, the only secret needed is for fathom. I used git-crypt to encrypt on push and decrypt on pull. That works fine for now. There's a nice walkthrough [here](https://buddy.works/guides/git-crypt).

I was looking to use [secrethub.io](https://secrethub.io/), but would need to figure out how I want to interface with it. In the past, I've made an operator similar to [kubernetes-external-secrets](https://github.com/godaddy/kubernetes-external-secrets). That will authenticate a workload and fetch its credentials from outside the cluster. I wanted to use [SPIFFE](https://spiffe.io/) for workload identity, but it seemed [Spire](https://spiffe.io/spire/) doesn't publish ARM images.

## Fully reconciling
My cluster is still managed manually, albeit from checked in manifests (it also [upgrades]({{< ref "k8s-auto-upgrades" >}}) automatically).

The next step is to use a GitOps Operator. I've used [ArgoCD](https://argoproj.github.io/projects/argo-cd) before, but [flux](https://fluxcd.io/) has been seeming more lightweight. It may come down to which one supports ARM better.

I also want to make the yaml easier to manage. First, using [kustomize](https://github.com/kubernetes-sigs/kustomize) to tie it all together and then exploring [jk](https://github.com/jkcfg/jk) to make templates in Typescript.