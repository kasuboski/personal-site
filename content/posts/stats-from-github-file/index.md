---
title: "Serve a JSON API with GitHub"
date: 2020-09-17T13:18:28-05:00
draft: false
---

I wanted to add stats to a site, but I already capture them in a GitHub Repo. Let's just pull from there.

<!--more-->

## The Stats Repo
I made a repo that pulls in stats ([kasuboski/stats](https://github.com/kasuboski/stats)). It uses a GitHub Action I made for a [Dev.to](https://dev.to/kasuboski/dev-to-article-stats-github-action-30n4) Hackathon that pulls post stats from Dev.to.

The repo gets periodically updated with a `stats/dev-to.json` file. GitHub lets you browse the contents of files at `raw.githubusercontent.com`. In my case, this file is at [https://raw.githubusercontent.com/kasuboski/stats/main/stats/dev-to.json](https://raw.githubusercontent.com/kasuboski/stats/main/stats/dev-to.json).

## Fetching the data
I have a [landing page](https://joshcorp.co) served from my [Raspberry Pi Cluster]({{< ref "home-k8s-raspberry-update" >}}). It was a placeholder with a link to my [personal site](https://www.joshkasuboski.com). Now it also shows stats from my [Dev.to posts](https://dev.to/kasuboski).

{{< figure src="landing-page-stats.png" link="landing-page-stats.png" alt="Stats on the landing page" >}}

The landing page itself is just vanilla HTML/CSS/JS. It uses [mvp.css](https://andybrewer.github.io/mvp/) to get quick styles. The repo is [kasuboski/joshcorp-site](https://github.com/kasuboski/joshcorp-site). The javascript needed to add the stats is below. It's just in a `script` tag in the body.

```javascript
function getStats() {
  const stats = document.querySelector('#stats');
  const reactions = document.querySelector('#reactions-value');
  const views = document.querySelector('#views-value');
  const url = 'https://raw.githubusercontent.com/kasuboski/stats/main/stats/dev-to.json';
  fetch(url)
    .then(res => res.json())
    .then(data => {
      console.log(data);
      reactions.innerText = data.public_reactions_count;
      views.innerText = data.page_views_count;
      stats.style.display = "block";
    })
    .catch(err => {
      console.error('Error fetching stats: ', err);
    })
}

window.onload = getStats;
```

I'm sure this probably isn't something GitHub exactly recommends... but as long as you don't have too much traffic it should be fine.