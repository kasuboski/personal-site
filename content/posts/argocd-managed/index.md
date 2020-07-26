---
title: "Managing the Cluster with ArgoCD"
date: 2020-07-26T14:24:26-05:00
draft: false
---

I had to manually apply changes to my cluster, but now a lot of it is controlled from git thanks to ArgoCD.

<!--more-->

## Managed with a cluster repo
The cluster state is kept in [kasuboski/k8s-gitops](https://github.com/kasuboski/k8s-gitops). Each folder is a different function for the cluster. The gitops folder is special. It has the [ArgoCD](https://argoproj.github.io/projects/argo-cd) manifests and the apps folder.

ArgoCD will sync manifests from a git repo to the cluster. It continuously watches to make sure the desired state (from git) matches the observed state (in the cluster). You configure ArgoCD using CRDs. Argo has Apps and Projects. A Project configures access for Apps and an App represents a repo that deploys Kubernetes objects.

You can see my apps at [gitops/apps](https://github.com/kasuboski/k8s-gitops/tree/master/gitops/apps). I use an app of apps [pattern](https://argoproj.github.io/argo-cd/operator-manual/cluster-bootstrapping/#app-of-apps-pattern) to deploy. I apply this root "apps" app that then deploys the other apps for the cluster.

ArgoCD also comes with a dashboard that shows the apps and their status. You can also get a badge to put in the repo to show overall status.

![](https://argocd.joshcorp.co/api/badge?name=apps&revision=true)

{{< figure src="argo-dashboard.png" link="argo-dashboard.png" alt="The argo dashboard shows apps" width="800px" >}}

## But why
Managing Kubernetes like this (once everything is automated) allows me to stand up a cluster with the same state simply by installing ArgoCD and applying an ArgoCD App.

It also keeps me from accidentally messing with the cluster since even if I delete a Deployment ArgoCD will reapply it.