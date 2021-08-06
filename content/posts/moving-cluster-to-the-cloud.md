---
title: "Moving My Cluster to the Cloud"
date: 2021-08-05T22:06:31-05:00
draft: false
---

I was going to move and didn't want to be without my rss reader (which I self host). To the cloud ☁️  it is.

<!--more-->

## Searching for Free VMs
I had been thinking of using the generous Oracle free-tier to run a k3s cluster. Especially since they recently announced a free 4x24 ARM node. The entrypoint to my homelab has already been moved from Linode to a free Oracle VM. I wasn't able to get the free ARM node in my always-free account. I added a payment method and was able to provision it in a not always free region. The compute is free, but I pay for the disk which is like $2 a month.

I had originally been thinking to use Postgresql as the storage for k3s, but it was using a ton of bandwidth between the Linode and Oracle. That's less of a concern now that I'm not using Linode, but I think I'll stick with sqlite. I'm less worried about their being an issue with it since it's hosted on a cloud VM.

## Setting up the Cluster
I ended up buying a VPS from RackNerd as part of a special for around $1.25 per month. It's 3x2.5 and comes with 6.5TB of bandwidth. I put the k3s server on there and am using the giant ARM node as only an agent.

This time for my cluster, I set up Tailscale first. I then had the api server bind to the Tailscale interface so it's only accessible while connected to tailscale. This made my cilium networking not quite work and I ended up going back to their default tunnel option. Then it just worked.

Since my cluster was already managed in git, I was able to just create a new branch and configure it for the new cluster. I took this time to move [k8s-gitops](https://github.com/kasuboski/k8s-gitops/tree/main) to using main as the default branch. Once I'm settled after the move I plan to add my Raspberry Pis and small form factor PCs to the cluster as well. One of the small form factor PCs will actually be moving to a different state as a backup.

## Finally Getting the RSS Reader
After I had the cluster up and running, I was able to use my existing kubernetes yaml to deploy miniflux. I did need to reencrypt the secrets so that it would work with the key on the new cluster. I'm still thinking to use a separate secrets management solution, but haven't gotten around to it. Even though I now have cloud disks, I still didn't really want to manage the miniflux Postgresql database. My previous attempts to find a free postgres didn't pan out with [heroku]({{< ref "migrating-to-free-heroku-postgres.md" >}}).

I found that [Supabase](https://supabase.io/pricing) has a very nice free plan. The database you get also gives you full access. I ended up configuring the miniflux database the same way I did when running it myself. Transferring the data went smoothly. I scaled down miniflux in the existing cluster and then used standard postgres backup and import tools to go from the one I hosted to Supabase. After adding the new connection info to the miniflux secret, it just worked 🧙.

## Moving Forward
I still need to set everything up at my new place and join them to the cluster. There are a bunch of things that I don't want to run in the cloud, so I'm still deciding if I want to taint the cloud nodes and use tolerations or just rely on affinities. I did already label the cloud nodes with a topology region of cloud so I can use it for scheduling. Ultimately, I'm hoping I can have it so things that can run anywhere will be able to bounce between going whereever is most efficient.

I'd also like to setup a separate ingress that is local only so I'm not always going out and back in to access cluster apps.
