---
title: "Getting Push Notifications for Everything with Pushover"
date: 2020-09-10T13:52:17-05:00
draft: false
---

I wanted to know when my personal site was deployed and decided to get push notifications.

<!--more-->

I make changes to my site locally and then push to GitHub so it is automatically deployed using GitHub Pages. I went over that in [Deploying this site with GitHub Actions]({{< ref deploy-site-github-actions >}}). After I pushed though, I would often have to keep checking to see when it should be available. Now I get notified.

## Setting up Pushover
In order to get push notifications from multiple apps, I signed up for [Pushover](https://pushover.net/). This lets me install just the Pushover app and all notifications will come through there.

I signed up for an account and downloaded the app. I was immediately able to manually send notifications. My original intention was to set this up for GitHub Actions though.

I created a Pushover app to get an API Token. Using this token and your user key, you can send notifications with an API call. 

## Adding to GitHub Actions
To get the status of my [personal-site workflow](https://github.com/kasuboski/personal-site/blob/master/.github/workflows/gh-pages.yaml), I just need to curl the endpoint with my token and user key.

I added both to my repo secrets and then could add the below step at the end of my workflow.

```yaml
- name: Notify
  if: always()
  uses: wei/curl@v1
  with:
    args: -X POST -F 'token=${{ secrets.PUSHOVER_TOKEN }}' -F 'user=${{ secrets.PUSHOVER_USER }}' -F 'message=Personal Site Pipeline ${{ job.status }}' https://api.pushover.net/1/messages.json
```

## And more
Pushover has a number of [integrations](https://pushover.net/apps). I have it setup to send me [UptimeRobot](https://uptimerobot.com/) alerts and also use it for Radarr. UptimeRobot just required me to add my user key and I immediately got a test message.

No longer will I be checking the status of anything :wink:.