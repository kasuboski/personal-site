---
title: "Test HTML in GitHub Actions"
date: 2020-10-07T16:08:40-05:00
draft: false
---

I want to start adding tests for this site so that I don't break it 😢. `htmltest` is first.

<!--more-->

## Setup htmltest
I went with [htmltest](https://github.com/wjdp/htmltest) mainly because it's just a go binary. I've seen other options in the past that required a Ruby environment and generally don't like dealing with that.

Installing `htmltest` into the current directory can be as simple as `curl https://htmltest.wjdp.uk | bash`. I installed it system-wide but 🤷‍♂️.

`htmltest` operates on HTML files...whereas my site is set up with [Hugo](https://gohugo.io/) so is largely Markdown. I have to generate my site first using `hugo` and then run `htmltest` on the `public` folder. Fortunately, htmltest has a config file that lets you set the directory. This way testing my site (once it's rendered) is just a matter of running `htmltest`.

The first time running htmltest my site had a number of issues. It found a broken link to an svg I had removed, my LinkedIn link was incorrect, and a number of missing `alt` tags. There were also a couple other issues that I ended up having ignored. I have some [indieweb](https://indieweb.org/) tags that didn't respond kindly to a `GET` request and one site I linked out to kept timing out in the test.

## Add to GitHub Actions
I installed htmltest in my pipeline using the magic `curl` script. This way it always gets the latest version and I didn't have to write anything. htmltest will cache the results of the remote links check so the workflow will cache the `tmp/.htmltest` folder to help keep it quick. Overall this adds six seconds to my pipeline.

This is all that was needed to add to the workflow yaml. It assumes htmltest is configured with `.htmltest.yml`. You can find my [config](https://github.com/kasuboski/personal-site/blob/master/.htmltest.yml) in the repo.
```yaml
- uses: actions/cache@v2
  with:
    path: tmp/.htmltest
    key: ${{ runner.os }}-htmltest

- name: HTML Test
  run: |
    curl https://htmltest.wjdp.uk | bash
    bin/htmltest
```

{{< figure src="actions-run.png" link="actions-run.png" alt="GitHub actions run" >}}

I did have to configure it not to fail on external links as it was failing every so often. Every change to my site now gets tested to ensure the links are valid.

## Next Steps 🦶
I want to add more tests. I think the first might be a spellchecker. I write these posts in VS Code which will highlight misspellings, but I don't always notice. I've used [vale](https://github.com/errata-ai/vale) before and it will even recommend some grammar changes.

I also want to test the Lighthouse score and make sure it doesn't get worse. This GitHub action seems nice [lighthouse-check](https://github.com/marketplace/actions/lighthouse-check). It could generate an HTML report like below.

{{< figure src="lighthouse-report.png" link="lighthouse-report.png" alt="lighthouse report" >}}