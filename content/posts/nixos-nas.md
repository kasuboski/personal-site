---
title: "NixOS NAS"
date: 2023-09-19T12:46:18-05:00
draft: false
---

I've given in to hoarding and got 20TB of storage 🫢.

<!--more-->


## Previous Setup
I had been using an external USB hard drive as my storage. It's attached to one of my kubernetes nodes which also runs NFS so other nodes can reach the storage. There's actually a very similar setup in Wisconsin which acted as a backup.

This worked pretty well, but I'd have issues with clients disconnecting randomly and rather slow access speeds. I assume this was largely from the combo of NFS and a USB drive, but didn't investigate too much. The hard drive was fairly large at 16TB, but I didn't use it much because of the above issues.

Moving forward I wanted to get more storage that can be accessed reliably as well as have the possibility to grow. 

## Planning
My old gaming PC was sitting unused in a rack mount chassis with hard drive bays. It seemed like the perfect fit for a new server dedicated to storage. There are pictures of the setup in my [post]({{< ref "server-rack-2" >}}) about rack mounting the PC.

I was mainly going to be storing media files and anything important would have multiple copies stored elsewhere. I'm also much too lazy to deal with some of the restrictions that come with a ZFS pool about managing vdevs and drive sizes. I decided on using a [mergerfs](https://github.com/trapexit/mergerfs) pool.

Mergerfs allows you to combine multiple drives under a single mountpoint. Depending on your configuration you can have the files distributed across them as well. So if a drive dies you're only out the files that were on that drive (not everything). Since most of my files won't change that often, I can also use [SnapRaid](https://www.snapraid.it/). It stores parity data of your files so that you can recover from drive failure. SnapRaid isn't as robust as something like ZFS since it works on snapshots, meaning there is a period of time where your new files aren't covered. 

SnapRaid uses a parity drive that must be at least as big as your biggest data drive. I decided to go with two 12TB data drives and an 18TB parity drive. This would give me an increase in storage capacity and allow me to go up to an 18TB data drive in the future.

Part of my problem with the previous solution was that I had to remember how to setup NFS and mount the disk correctly every time. I wanted to make sure that not only was it documented, but there wouldn't be hidden changes I forget about.

I decided to take the plunge to an ephemeral NixOS since I had been liking [nix for development environments]({{< ref "nix-dev-environment.md" >}}).

## Setting it up
Installing NixOS was a little different since I wanted an ephemeral root. This meant everytime the PC restarts the root partition would be erased. This is actually implemented by rolling back to an empty btrfs snapshot, but the effect is the same. Anything that isn't explicitly set to persist will be erased.

There are a few things that are always persisted. The nix store stays around so it doesn't rebuild the world everytime. There is also a `/persist` mount that stays for anything like application state that should stick around. I also currently have a persistent `home`, but it's not really necessary.

I have a pretty rough guide at [kasboski/home-infra/nas.md](https://github.com/kasuboski/home-infra/blob/0fb543fc9a0d5065cc0b1e31b0d2c2742cc62c1a/nas.md). I generally followed [https://guekka.github.io/nixos-server-1/](https://guekka.github.io/nixos-server-1/) with some tweaks for my case.

My server was then setup to be entirely controlled by the nix config. You can skip to just reading the `nix` at [kasuboski/dotfiles](https://github.com/kasuboski/dotfiles/blob/a5bb2bc2b819edea75a08afac350a042f5a533f4/nixos/hosts/fettig/configuration.nix). I'm not sure my dotfiles repo is really the place for this, but I didn't have a better idea.

I was much more confident in editing the server now that I could always revert to a known working config.

Adding the storage disks, I mostly went with the instructions from [SnapRaid-Btrfs](https://wiki.selfhosted.show/tools/snapraid-btrfs/). It formats the data drives with `btrfs` which helps alleviate the issue for snapraid where it needs to snapshot while files are available, but not changing. Snapraid btrfs orchestrates taking a readonly btrfs snapshot and then runs snapraid on the snapshot. 

Snapraid btrfs and the runner mentioned weren't already packaged with nix, which led me to learning more about packaging random projects for nix. It took me quite awhile as the combo wasn't really meant to run without system dependencies. I had issues where it would work under my user which had local dependencies, but then it would fail in an isolated systemd service.

The entire setup when configured manually requires quite a few steps. My resulting configuration is by no means simple, but I think it at least points me in the right direction of how things are setup. The main file is [storage.nix](https://github.com/kasuboski/dotfiles/blob/a5bb2bc2b819edea75a08afac350a042f5a533f4/nixos/hosts/fettig/storage.nix) that lists the drives, maps them as data or parity and mounts them, while setting up the snapraid config.

Getting NFS and Samba shares setup was also somewhat straight forward considering I had followed random guides how to do it before. It's nice to be able to configure a user for the share, a mount point, and the system services all in the same place. I still had to spend quite a bit of time investigating the specific NFS and Samba options I wanted, but I didn't need to hunt down where to add them or how to make it take effect.

## Moving Forward
I'm pretty happy with the setup. Adding services with nix is often as simple as enabling them. I want to setup an actual replication solution since currently I just run rsync to my old external hard drive every now and then. I also want to host various services that take advantage of the storage now. Some examples might be hosting photos or a personal pastebin. 

I had started to use the NAS as a dev server since it was nice to have a powerful machine that could use a nix dev environment. I now kind of want to get a dedicated development machine to connect to instead of messing with that machine. Maybe a [NixOS Container](https://nixos.wiki/wiki/NixOS_Containers) would be isolated enough instead.

I also can't resist trying to run a kubernetes node on it. I am thinking I'd want it to be in a VM or see if a NixOS container would work though. Preferably the storage aspects wouldn't really need to know about the k8s node.
