---
title: "Setting Up Microsub"
description: "Trying to replace Feedly with an open source IndieWeb alternative"
date: "2020-01-15T18:48:10-06:00"
author: "Josh"
draft: false
---

I use [Feedly](https://feedly.com) to manage my content subscriptions. I subscribe to a number of bigger sites and personal blogs. I would like to move to an open source version so that I can customize my workflow and use the data in multiple places.

<!--more-->

I've been trying to utilize [IndieWeb](https://indieweb.org) pieces more and more. This site supports [IndieAuth](https://indieauth.com/) so I can login at supporting websites by giving my URL. The posts and contact card are also marked up with [microformat](http://microformats.org/) to be parseable.

## Experimenting with Aperture and Monocle
I decided to use [Aperture](https://aperture.p3k.io/) for now as my [microsub](https://indieweb.org/Microsub) server. A microsub server is responsible for fetching the content you subscribe to and making it available in a common format for a microsub reader.

I was able to login and subscribe to [Aaron Parecki's](https://aaronparecki.com/) personal site, which immediately loaded some 1600 entries for me. After adding a `rel=microsub` link to my homepage, I was able to log into [Monocle](https://monocle.p3k.io/) and view that feed in my home channel. The default view for Monocle seems to be to show everything. There is an option to only show unread, but it's not what you get by default.

## Moving forward
Monocle doesn't quite fit how I want to view updates, but it did help me understand the concepts better. The free hosted Aperture only saves your data for 7 days so I probably need to either host it myself or find a different microsub server.

I've been looking at [ekster](https://github.com/pstuifzand/ekster). I like that it's a go binary and comes with a CLI. It has the option of importing an opml feed, which Feedly would export. It seems all of your channels and feeds are stored in a config file (that you can generate with the opml import) and redis is really meant to be a cache.

It does seem to support the [follow action](https://indieweb.org/Microsub-spec#Following) however and it doesn't look like that updates the file. In the future, I'll probably just try to run it and see what happens.

Ekster also has a reader associated with it, but there are a number of [others](https://indieweb.org/Microsub#Clients) to try including mobile apps.