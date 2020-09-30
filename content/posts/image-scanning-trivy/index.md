---
title: "Container Image Scanning with Trivy"
date: 2020-09-25T11:01:50-06:00
draft: false
---

I wanted to have some peace of mind when running random container images. Trivy let's me scan them for common vulnerabilities.

<!--more-->

## Installing Trivy
You can find the Trivy repo on GitHub at [aquasecurity/trivy](https://github.com/aquasecurity/trivy). Installing with Homebrew is just `brew install aquasecurity/trivy/trivy`. Trivy is written in Golang so you just need to get the binary. They also have a magic script you can use

```curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/master/contrib/install.sh | sh -s -- -b /usr/local/bin```

Once you have `trivy` in your `$PATH`, you can run `trivy` and see the options. Trivy can do a number of scans: a remote image, local filesystem, or a remote repository.

The various options make it easy to scan code repos, images before they are pushed, and third-party images you want to use.

## Scanning an image
I use [ArgoCD](https://argoproj.github.io/projects/argo-cd) and had to use an image other than the official one since I wanted multi-arch support. This saved me from having to build it myself which I've done in the [past]({{< ref "build-multiarch-image" >}}).

I wanted to scan this image for vulnerabilities (the build is open source, but you never really know). With Trivy this is as easy as `trivy image alinbalutoiu/argocd:v1.7.1`. It will download (and cache) the vulnerability database and then pull and scan the image. It then outputs a nice table of vulnerabilities as seen below. You can also filter by severity and ignore unfixed.

{{< figure src="trivy-results.png" link="trivy-results.png" alt="Trivy Results" >}}

## Next Stop CI
This is great for testing an image ad-hoc, but I want to add this to my [gitops repo](https://github.com/kasuboski/k8s-gitops) so that all images are scanned periodically. There is already a Trivy GitHub Action, but I think it's intended more for images you are building.

I also want to run something in my cluster that will periodically check all images that are running. Something like [starboard](https://github.com/aquasecurity/starboard) could be the beginning of that.