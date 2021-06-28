---
title: "M1 Macbook Air Setup"
date: 2021-06-28T15:10:16-05:00
draft: false
---

I recently got an M1 Macbook Air. I wanted to try out a new development setup and track my dotfiles in git.

<!--more-->

## The Machine
I got the standard config with the 7-core GPU. I went with the standard one largely because bumping to more RAM meant it wouldn't arrive for two weeks and I'm impatient. I don't plan to use this for anything too intensive so hopefully the 8GB will be enough.

So far it's been great. I had kind of forgotten how small a 13.3" display would be, but the extra portability has been nice. The battery actually hasn't gone below 50% yet while I've been setting the machine up. I'm sure the only two usb-c ports will come to annoy me, but for now I'm just happy I didn't need a different charging standard.

## The Config
I decided to use [chezmoi](https://www.chezmoi.io/) to manage the config for this machine. I had been meaning to do it for awhile on my WSL Ubuntu, but never got around to it. I'm very worried I won't be able to recreate the set up in WSL... Using chezmoi, I can codify my dotfiles in a git repo that then gets applied. You can find the repo at [kasuboski/dotfiles](https://github.com/kasuboski/dotfiles). I still need to work on it supporting more than just mac. I've read it can integrate nicely with codespaces so I'm especially interested in that or [toolbox](https://github.com/containers/toolbox).

In theory, once I have the config working for multiple types of machines, I should be able to do `sh -c "$(curl -fsLS git.io/chezmoi)" -- init --apply kasuboski` and have a similar machine. It will be nice to have a similar config across the various devices I use. 

## The Tooling
You can see the packages I installed with brew in an [install script](https://github.com/kasuboski/dotfiles/blob/main/run_once_before_10-install-packages-darwin.sh.tmpl). Chezmoi only runs the file when its contents change. Sending the list of packages to brew in this way makes it easy to only rerun when new packages are installed.

The biggest change to my setup is probably using the [fish](https://fishshell.com/) shell. I had been using zsh in wsl, but like a lot of the built-in features of fish. I've really enjoyed seeing the completions while learning chezmoi for instance. I'm a little worried I'll run into issues using something that's less popular. I've already found the kubectl aliases I usually use don't have an easy fish equivalent, but there is at least [kubectl completions](https://github.com/evanlucas/fish-kubectl-completions) available. It's even available to install as a [fisher](https://github.com/jorgebucaran/fisher) plugin.

I also installed a bunch of the rust cli tools that replace common unix commands. My most frequently used is certainly [lsd](https://github.com/Peltoche/lsd) that makes your `ls` output much prettier. I have `ls` aliased to `lsd` so I use it exclusively now.

I'm also taking this opportunity to try and get back into using vim or rather [neovim](https://neovim.io/). Usually, I use VSCode and I'll probably end up installing it again, but want to see if I can stay mainly in the terminal. I still haven't really figured out all of my neovim plugins. I need to set up [coc](https://github.com/neoclide/coc.nvim) for neovim to have VSCode like language server protocol and extension features.

## M1 Thoughts
This mac is incredibly responsive. I haven't done anything stressing yet, but all apps have opened instantly. I did have to use Rosetta for Discord and it's probably the only thing I've noticed is slower than I expect. I'm glad I waited a bit so things had some time to be compatible with darwin arm64.

I already mentioned the battery life, but coming from my Mid-2012 Macbook Pro it's night and day. That macbook was starting to show its age in general use as well. 

Using Touch ID to unlock the mac is also something I hadn't really thought about. It's very nice to come back and just touch the keyboard for it to unlock. Maybe it will even get me to use Apple Pay.


