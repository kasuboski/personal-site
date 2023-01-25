---
title: "NixOS Dev Environment on Mac"
date: 2023-01-24T21:17:07-06:00
draft: false
---

I needed more trend in my dev environment setup. Now more reproducible and declarative.

<!--more-->

## Nix?
Nix gets to be many things. There's an expression language, package manager, and operating system built around it. You can learn more at [nixos.org](https://nixos.org/learn.html). I was mostly interested in using NixOS the operating system. It hopes to enable declarative and reproducible builds.

I had seen Mitchell Hashimoto's [dev setup](https://github.com/mitchellh/nixos-config) that uses a NixOS VM on his mac in VMWare Fusion. It looked like it could solve many of the issues I had with random configs that you forget about polluting your machine. You not only declaratively specify what you want, but it can also be separate from your mac.

The Nix expression language was a little much to get used to, but I have been dabbling for awhile. It seems like Nix Flakes are starting to actually happen now as well (even though they're still experimental). They helped ease my confusion around the package manager channels and versions you would previously use (I think). I went straight to flakes.

## My setup
I decided I also wanted to try out [Lima](https://github.com/lima-vm/lima) at the same time because why not do two new things. Lima is a nice interface to run linux virtual machines on Mac. You define a `yaml` file that specifies the image and directories you want to use along with cpu and memory and then you get a linux VM.

Most of my struggle was trying to get NixOS to run with Lima. Lima expects certain things to be configured on boot and from cloud-init. NixOS generally doesn't ship with cloud-init in the images and I wanted to keep most things in the Nix config. I had found a [repo](https://github.com/patryk4815/ctftools/tree/master/lima-vm) that worked for me, but I had a hard time using my separate flake to reconfigure. It turns out the alpine image for Lima also doesn't have cloud-init. It instead has a bash script to set things up. I just needed to have my lima image run a similar script on startup.

Nix makes it rather simple to add systemd units like below. It's pretty similar to the actual format and you just substitute in your script inline. 

```
systemd.services.lima-init = {
    inherit script;
    description = "Reconfigure the system from lima-init userdata on startup";

    after = [ "network-pre.target" ];

    restartIfChanged = true;
    unitConfig.X-StopOnRemoval = false;

    serviceConfig = {
        Type = "oneshot";
        RemainAfterExit = true;
    };
};
```

I was using my configuration in a nix flake to use [nixos-generators](https://github.com/nix-community/nixos-generators) to make an image that would then let me boot into NixOS directly. I was very much learning Nix as I went so have no idea if I landed on a good setup. My flake ended up with an `img` output that would make the image and then I could add my separate config as a different target.

You can see my image base at [kasuboski/nixos-lima](https://github.com/kasuboski/nixos-lima). It also has an example of user specific configuration. I couldn't decide how to add a user configuration, but eventually realized how to use the output of another flake and decided to move my user config to my dotfiles repo.

## The actual user config
At the time of writing this, I haven't merged the PR with my Nix changes to my dotfiles [repo](https://github.com/kasuboski/dotfiles/tree/nixos/nixos). While getting it working, I decided to do it piecemeal. I had recently switched to using [asdf]({{< ref "local-tools-asdf.md" >}}) and there really isn't anything wrong with that setup. The way I set it up felt like a lot of indirection though and I was still going to be using `brew` for most things.

My hope with Nix is to use it on Mac with a linux VM, on a cloud VM, or maybe in a container and have a similar experience across them all. I currently have my main shell config installing system-wide for NixOS as well as `chezmoi`. I can then use my previous setup using `asdf` and `chezmoi` to set up my dotfiles. I'm not quite ready to dive into the more Nixy [home-manager](https://github.com/nix-community/home-manager) although it does seem nice.

I'm still missing a bunch of things especially like cloud clis, but when needed I tend to use a `nix shell`. This lets you temporarily add packages to your environment without them being global. It is a little annoying typing out the packages to add so I eventually want to either make project specific settings or maybe have flakes based on what I'm doing.

They could be for example DevOps/cloudy or language specific. Luc Perkins has started something similar at [the-nix-way/dev-templates](https://github.com/the-nix-way/dev-templates). They generally use `nix develop`, which (I believe) is meant for when you are developing a package so it isolates to just the items defined there. I'd prefer to keep my `fish` shell and settings instead.

## Moving Forward
I like the setup for now. I definitely want to test it out on other platforms and see how portable my config really is. Flakes for NixOS seem pretty new and I had a hard time finding examples. Even a lot of the examples I did find were very much focused on homelabs and baremetal machines where you would know all the details ahead of time and statically. 

It made it quite hard to understand how my config should deal with not knowing until boot what the user should be and I'm not really used to having a configuration per instance as the examples were per host. It feels a little like managing pets vs cattle. 

Even `nixos-rebuild switch` defaulting to a configuration using the hostname was hard to get in the right mindset for. I've grown accustomed to the hostname being generated and I ran into an issue where my work macbook needed a different architecture, but had the same Lima hostname. My config now has duplicate lima entries `nixos` and the unimaginative `nixos86`.  
