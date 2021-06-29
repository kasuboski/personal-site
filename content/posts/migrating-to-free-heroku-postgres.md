---
title: "Migrating to Free Heroku Postgres (or not)"
date: 2021-06-29T15:17:39-05:00
draft: false
---

I want to move my miniflux database to a managed service. Spoilers... it didn't go well.

<!--more-->

## Current setup
I currently have [miniflux](https://miniflux.app/) running in my [raspberry pi kubernetes cluster]({{< ref "home-k8s-raspberry-update.md" >}}). Miniflux requires a Postgresql database. I haven't had great experiences running persistent storage on the Raspberry Pis so I opted to run the database on my thinkcentre instead. It's running as a container and hasn't been an issue since.

There have been times however, where I have broken my entire homelab. The power grid in Texas has also been less than reliable 🥶. It was rather annoying to be freezing and also not have access to my feed reader. Since I want my feed reader to be more available than my homelab permits, I'm naturally trying to extend to the cloud.

## Managing the Database
Free managed databases seem to be hard to come by. Many cloud providers offer them in the trial period, but not in the always-free plans. Heroku, however, has a pretty nice offering. 

You get 1GB of storage and 10,000 rows. You're even able to retain 2 backups. I checked my current miniflux database and it's only using 150 MB. There should be plenty of room to grow. I think miniflux also clears entries after a certain period.

I created a Postgresql add-on in the Heroku console. They only really tell you how to connect from a Heroku app, which isn't what I'm after. I eventually found this help [doc](https://devcenter.heroku.com/articles/connecting-to-heroku-postgres-databases-from-outside-of-heroku) that pretty explicitly says you shouldn't rely on the connection info statically. The recommended option is to use the Heroku cli to get your connection string.

At first, I tried to create a container that has the Heroku cli in it. Unfortunately, it wouldn't build for ARM, which I need for the Pis and my upcoming free Oracle ARM VM. I still built it for amd64 so if that's all you need it's at [kasuboski/heroku-cli](https://github.com/kasuboski/heroku-cli).

My next idea was to just call the API directly. There are a couple of clients to help. The Golang one hadn't been updated in years so I went with [nodejs](https://github.com/heroku/node-heroku-client). You can see the project I made at [kasuboski/heroku-ps-url-grabber](https://github.com/kasuboski/heroku-ps-url-grabber).

It's a bit of a mouthful, but it will get the config variables associated with your Heroku app. In my case, it only has a `DATABASE_URL` for the Postgresql addon. This little script means I can get the connection string and store it in a file that miniflux can read.

## Migrating to the Managed Database
After getting that working and building on ARM, I moved on to migrating the data. The Heroku cli has a nice [pg:push](https://devcenter.heroku.com/articles/heroku-postgresql#pg-push) command that will push from one Postgresql to the one in your account. It took me a second to figure out how to have it point to another host, but once it did everything went smoothly. An example command is below.

```
heroku pg:push postgres://user:pass@olddb/miniflux postgresql-random-id --app your-app
```

I was adding an initContainer to my miniflux kubernetes deployment, when I noticed an email from Heroku. My database had 16,000 rows and would have `INSERT` permission removed in 6 hours. The miniflux backup apparently contained 16,000 rows, far above the 10,000 free limit 😒.

The free Heroku Postgresql isn't going to work for me it seems.

## Next Steps
I still want to get my miniflux running elsewhere (or at least have the ability to easily switch it). I had been considering adding a cloud node to my cluster. Once that is setup, I will look into running miniflux on that including its postgres.

I have been wanting to do something like [this](https://itnext.io/how-to-deploy-a-single-kubernetes-cluster-across-multiple-clouds-using-k3s-and-wireguard-a5ae176a6e81). I already run [tailscale](https://tailscale.com/) on all of my nodes so running the kubernetes control plane in the cloud with nodes spread about may be the perfect solution.

You can see what my tailscale setup is like in [Connect Your Home to the Cloud with Tailscale]({{< ref "connect-home-cloud-tailscale" >}}). Oracle was currently out of capacity for ARM VMs, but hopefully I'll be able to provision soon.