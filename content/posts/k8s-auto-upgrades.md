---
title: "Automated Upgrades for k3s"
date: 2020-07-08T18:50:17-05:00
draft: false
---

My cluster was falling behind the latest k3s version. Time to upgrade.

<!--more-->

## Enter the System Upgrade Controller
I basically just did the instructions linked [here](https://rancher.com/docs/k3s/latest/en/upgrades/automated/).

It deploys the system upgrade controller into its own namespace where it won't do anything yet.

You have to give it some plans.

It will read Plans in that same namespace and run the specified image. I used the plan linked above. Instead of tying it to a specific version I set it to the latest channel.

```
# Server plan
apiVersion: upgrade.cattle.io/v1
kind: Plan
metadata:
  name: server-plan
  namespace: system-upgrade
spec:
  concurrency: 1
  cordon: true
  nodeSelector:
    matchExpressions:
    - key: node-role.kubernetes.io/master
      operator: In
      values:
      - "true"
  serviceAccountName: system-upgrade
  upgrade:
    image: rancher/k3s-upgrade
  channel: https://update.k3s.io/v1-release/channels/stable
---
# Agent plan
apiVersion: upgrade.cattle.io/v1
kind: Plan
metadata:
  name: agent-plan
  namespace: system-upgrade
spec:
  concurrency: 1
  cordon: true
  nodeSelector:
    matchExpressions:
    - key: node-role.kubernetes.io/master
      operator: DoesNotExist
  prepare:
    args:
    - prepare
    - server-plan
    image: rancher/k3s-upgrade:v1.17.4-k3s1
  serviceAccountName: system-upgrade
  upgrade:
    image: rancher/k3s-upgrade
  channel: https://update.k3s.io/v1-release/channels/stable
  ```

You'll notice there are actually 2 plans. One applied to the master nodes and the other the worker nodes. This lets you upgrade the master node to the new version before the workers.

The entire process took maybe 5 minutes for my 3 node cluster. I was able to `kubectl get nodes` periodically and watch the progress.

