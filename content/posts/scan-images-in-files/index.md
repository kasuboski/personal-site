---
title: "Scan Container Images Found in Files"
date: 2020-10-01T11:29:05-05:00
draft: false
---

I previously had been scanning images ad-hoc using Trivy. Now I can scan everything in my GitOps repo.

<!--more-->

The previous post about Trivy is [here]({{< ref "image-scanning-trivy" >}}).

## Finding the Images to Scan
Since I track all of my deployments in git at [kasuboski/k8s-gitops](https://github.com/kasuboski/k8s-gitops), I can find all of the images from the files in that repo. For the simple case of just finding it from Kubernetes objects, I can use the command below which finds the images, removes quotes, and makes sure it's unique. Magic 🧙‍♂️.

```bash
ack --type yaml 'image: (.*)' --output '$1' -h --nobreak | tr -d \"\' | sort -u
```

This outputs everything coming after "image: ". This works for Kubernetes resources and it also picked up some values in a helm values file. I render manifests using kustomize, which can change images and pull remotely. This means I should really be running this command on the rendered manifests.

I've been considering rendering everything in git instead of only on the cluster (in ArgoCD). This would capture the exact yaml that was applied and make this analysis easier.

## Scanning with Trivy
You can use `xargs` to run a scan for each image. I did this initially just piping to `trivy image`, but immediately hit the GitHub rate-limit. I thought trivy would reuse the database cache, but that didn't seem to be happening. They do let you pass a [GitHub Token](https://github.com/aquasecurity/trivy#github-rate-limiting) to increase the limit, but that seemed like more effort.

Instead, I decided to use the client server mode of trivy. This let's you run a server that executes the scans and your client communicates with it.

To start the server:
```
trivy server
```

Then (in another terminal) run a `trivy client` for each image:
```bash
ack --type yaml 'image: (.*)' --output '$1' -h --nobreak | tr -d \"\' | sort -u | xargs -L1 trivy client --severity HIGH,CRITICAL --ignore-unfixed
```

This gives output like below:
{{< figure src="results.png" link="results.png" alt="Trivy Results" >}}

It did fail to analyze an image that only had an ARM manifest (I ran this on my amd64 desktop)

## Next Steps 🦶
I would like to run this repeatedly as a GitHub Action on the repo itself. It might be easier to just use a GitHub Token for the rate-limit in that case. I also want to analyze the rendered manifests.

I have been considering setting up my own registry to mirror my images such as [Harbor](https://goharbor.io/). Harbor can run periodic Trivy scans by itself and make the results available. There is also a an experimental [trivy-enforcer](https://github.com/aquasecurity/trivy-enforcer) project that runs in the cluster and can enforce that all images running in your cluster are secure.

This combination of static analysis and runtime enforcement should make it pretty tough to have a known bad image running.