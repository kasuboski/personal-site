---
title: "Connect to Kubernetes Pods with Tailscale"
date: 2022-04-03T13:52:19-05:00
draft: false
---

I wasn't ready to add auth to my new feedreader. So I built a moat with Tailscale.

<!--more-->

## What does that _mean_?
I added a kubernetes pod to my tailnet so it's accessible from anywhere that can route to Tailscale nodes. It also gets a nice domain name so `http://feedreader` works.

Tailscale actually has a [blog post](https://tailscale.com/blog/kubecon-21/) and [example](https://github.com/tailscale/tailscale/tree/main/docs/k8s) for how to set this up. I wanted it to be mildly different so modified their run script. They also don't publish their example image so I needed to build one.

You can see my version at [kasuboski/tailscale-proxy](https://github.com/kasuboski/tailscale-proxy). The main differences are it takes a `HOSTNAME` and `DEST_PORT` parameters. 

The `HOSTNAME` is so you can set the name the node will show up as. This was important for kubernetes because I don't want the generated name of the pod to be how I access it. `DEST_PORT` is so you can have it forward to a different port. This allows you to run your app on `8080`, but the `tailscale-proxy` will route any port to `8080` meaning you can hit it on `80` in your browser. This is required to not have `:8080` ugliness after your URL.

## But Why?
I've been making a [feedreader](https://github.com/kasuboski/feedreader) to replace my running [miniflux](https://miniflux.app/). It's my first _real_ rust project and I wanted an even more minimal feedreader.

I haven't gotten around to figuring out users or authentication in the feedreader though. Despite this, it finally reached the point where I can use it as my main feedreader. However, I didn't exactly want an unauthenticated app hanging out on the internet for someone to ruin my day.

If you don't want to make authentication, just make it inaccessible. This is where [Tailscale](https://tailscale.com/) comes in. I already run Tailscale on all the nodes, which is how I'm able to have a [multi-region k3s cluster]({{< ref "multi-region-k3s" >}}). That doesn't make my pods routable though.

Tailscale has an option for a [subnet router](https://github.com/tailscale/tailscale/tree/main/docs/k8s#subnet-router) that is actually highlighted as how to access all things k8s in the examples. This probably would have been nice (and I might still add it), but I wouldn't automatically get dns routing I believe.

You can see how it's all put together in my [k8s-gitops repo](https://github.com/kasuboski/k8s-gitops/blob/main/default/feedreader/add-tailscale-proxy.yaml). The gist is that you add a sidecar container that starts tailscale and in my case adds an `iptables` rule which forwards all traffic to the app port. You can see my poor Doppler secret naming in there too where everything is using `miniflux-secret`.

## So, no more auth?
I still want to add users to the feedreader so that you can have multiple users on one instance. It remains to be seen whether I'll just add basic auth or something more complicated. Tailscale let me focus on getting something usable quickly though.

After seeing how convenient it was to expose a pod with a dns name, I also want to make a debugging tool injecting tailscale to pods. I could then finely be rid of the finicky `kubectl port-forward`.
