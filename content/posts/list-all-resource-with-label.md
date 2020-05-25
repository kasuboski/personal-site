---
title: "Kubectl List All Resources With Label"
date: 2020-05-24T22:19:00-05:00
draft: false
---

I recently had to remove a long gone helm release that left behind a bunch of resources. This conflicted when reinstalling, so I needed to find them.

<!--more-->

## A magic command
I'm mainly putting this here because it seemed non obvious to me.

```
kubectl api-resources --verbs=list -o name | xargs -n 1 kubectl get -o name -l release=prometheus-operator
```

That will find all `api-resources` that have the label `release=prometheus-operator`. I had tried to use `kubectl get all -A`, but this seemed to only return built-in types.