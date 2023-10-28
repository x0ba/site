---
title: "Getting started with Nix"
publishDate: "3 March 2023"
description: "Part 1 in a series of articles explaining NixOS configurations in an easy-to-understand way."
tags: ["nix", "linux"]
---

It's happened. I've fallen down the Nix rabbit hole. It was inevitable. I never really **needed** Nix, per se. All of my systems worked perfectly fine with "regular" dot files, but there were always these tiny, annoying things that combined to make the overall experience less than ideal. For example, even though my dotfiles could be backed up, there was no easy way to _declare_ which packages were to be installed. Of course, installation scripts that applied one's dotfiles and installed required packages could be made, but those need to constantly be updated as new programs are installed. Another problem with the "traditional" approach to system configuration is that packages and configurations are prone to breakage on update. If the developer of a package suddenly decides to deprecate or rename certain configuration options, the configurations of that particular package will break and it might take a lot of time and effort to fix this breakage. Individually, these problems are immaterial, but as they compound on one another, traditional package and configuration management begins to feel like a chore.

Nix solves all of these problems. Packages are managed in a _declarative_ way, defined in a `.nix` configuration file that will result in the exact same package versions every single time. Configuration options can also be defined with Nix, using a system called Home Manager, which essentially lets you define your configurations in a `.nix` file and converts them to "regular" dotfiles with a simple `home-manager --switch`. With Nix, updates **can** cause breakage, but unlike traditional package managers, Nix is _generational_; when you edit your configurations and rebuild your system (more on that later), the previous generation (a snapshot of the currently installed packages and configuration options), is saved, and a rollback to that saved generation is trivial. All of this combined with Nixpkgs, the biggest and most up-to-date package repository in the world makes Nix a great experience once you get started.

But how does one get started? Nix has a notoriously high learning curve, as evidenced by the wise words of Hlissner:

> Question: Should I use NixOS?
>
> Short answer: no.
>
> Long answer: no really. Don't.
>
> Long long answer: I'm not kidding. Don't.

The Nix language is obtuse and issues are unique and difficult to Google. Getting started with a flake config, which is all but required nowadays, can seem overwhelming, and there are no good articles purely on flake system configuration out there. I will try to take a different approach: instead of teaching the basics of the Nix language, I will attempt to jump straight to system configuration with Nix. This will be part one of what will (hopefully) be a multi-part series.

## Installing Nix

One thing to remember about learning Nix is that *the community is your best friend*. Taking inspiration from or copying others' Nix configs is a *perfectly valid* way of learning how to Nix. For reference, my Nix config is hosted [here](https://github.com/x0ba/dotfiles).

That being said, my Nix configuration started as a humble standard flake from [Misterio's nix starter config repo](https://github.com/misterio77/nix-starter-configs). I would highly recommend you start out with the standard template from that repository instead of writing your own flake from scratch, as things like custom modules, packages, and overlays all come perfectly configured and linked with Home Manager out of the box with that flake.

I'm using Nix on a MacOS device, so I'll need to install the Nix Package Manager first. From [NixOS's Website](https://nixos.org),

```bash
sh <(curl -L https://nixos.org/nix/install) --daemon
```

Run through the installation, giving the script sudo privileges, and eventually you'll have access to the new `nix` command.

## Initializing Your First Flake

As stated above, we are going to be using a premade template. In order to do so, navigate to whichever directory you want to store your Nix configuration flake in. Then, initialize a git repository (everything in a flake needs to be tracked by git; more on that later).

```bash
cd ~/Documents
git init nix-config
cd nix-config
```

Then, enable the `nix-command` and `flake` experimental features

```bash
nix --version
export NIX_CONFIG="experimental-features = nix-command flakes"
```

And finally initialize the flake.

```bash
nix flake init -t github:misterio77/nix-starter-config#standard
```

The boilerplate has been dropped (it made a loud clanging sound).

## Basic Flake Anatomy

Open up `flake.nix` in your favorite text editor. If you're familiar with `json` syntax, you'll notice that there are two main sections to this file: **inputs** and **outputs**.

Put simply, the **inputs** are external sources from which your flake config can read files or packages. These can either be other flakes or just a git repository you'd like to pull a file from. Either way, the versions of these files are pinned in place by a `flake.lock` file, which stores the exact revisions of each input for complete reproducibility.

A Nix flake can not only accept inputs, but also provide arbitrary Nix values, such as packages and NixOS Modules. These are called its **outputs**. Things such as NixOS configurations, Nix-Darwin configurations (more on those in a future article) and Home-Manager configurations use outputs as entry points.

Before we continue, follow the instructions in the comments above the `homeConfigurations` section in the flake's outputs to name the output.

## Basic Home-Manager configurations and options

For now, let's just get started with a very basic home-manager configuration. As you play around with Nix in the future, your configurations will get more and more complicated and modular, but we'll start with a single-file `home.nix`. Open up the `home-manager/home.nix` file in your preferred text editor. For now, we'll scroll down past all the gibberish that are the imports and overlays sections until we get to the `home` section. It should look something like this:

```nix
  home = {
    username = "your-username";
    homeDirectory = "/home/your-username";
  };

  # Add stuff for your user as you see fit:
  # programs.neovim.enable = true;
  # home.packages = with pkgs; [ steam ];

  # Enable home-manager and git
  programs.home-manager.enable = true;
  programs.git.enable = true;

  # Nicely reload system units when changing configs
  systemd.user.startServices = "sd-switch";

  # https://nixos.wiki/wiki/FAQ/When_do_I_update_stateVersion
  home.stateVersion = "23.05";
```

Let's go through this line-by-line. `home.username` and `home.homeDirectory` are self-explanatory; they set your user's username and your user's home directory (e.g. `/home/john` on Linux). `programs.*program*.enable` installs a program from the `nixpkgs` archive (more on that in a future article) into your home environment. This also allows you to configure that program using `home-manager`, as we will do in a bit. `systemd.user.startServices` is a Linux thing that reloads all systemd services when you perform a `home-manager switch`. State version is very important, though. It essentially sets the options available at that version. You should update it very rarely, only when you have fully read the release notes for a new state version and fully understood what it will change.

With all the boring explanations out of the way, let's get started with a basic `home-manager` configuration for `zsh`. Below `programs.git.enable`, add the following code:

```nix
programs.zsh = {
	enable = true;
	shellAliases = {
		pls = "sudo";
	  cls = "clear";
	};
	initExtra = ''
    PATH=$HOME/.local/bin:$PATH
	'';
};
```

Home-manager takes Nix expressions defined in your `home.nix`, converts them into "traditional" dot files, and symlinks them into your home directory. In order to do so, the user must define home-manager **options**. A tool I use all the time is [home-manager options search](https://mipmip.github.io/home-manager-option-search/), which allows you to search for home-manager options. It's mostly pretty intuitive. The names of the options are formatted like so `programs.zsh.shellAliases`, and can be one of multiple types: an attrset, a string, or a list. Attrsets are defined using curly braces, strings are defined with either quotation marks or something as seen in the `initExtra` option above, which is two apostrophes, both opening and closing. Those concentanate new lines. Lists are defined using square brackets.

If multiple options are grouped under the same "parent" option, they can be defined in an attrset together under that "parent" option, as shown above with `programs.zsh` as the "parent" option and `enable`, `shellAliases` and `initExtra` as the "child" options.

In order to apply this configuration option, run `nix develop` to install the needed tooling for this, including `home-manager` and `git`. Then, apply the configurations with `home-manager switch --flake .#user@host`. Congratulations! You've set up a basic home-manager configuration for zsh with a couple of aliases and path modifications.

## Ending Note

Like I said earlier, this will be part one of a series of articles explaining NixOS configurations in an easy-to-understand way. There aren't a lot of easy to understand guides on this topic floating out there, so I hope to rectify that. Until the next article!
