---
title: "My Backup System"
publishDate: "27 February 2023"
description: "Everyone should be making backups. Here's how I do mine."
tags: ["tech", "privacy"]
---

Backups are one of the most important things you can make. If you don't make backups, and something happens to your computer, it's all gone. Reduced to ashes. This article is meant to be a short little brain dump on how I do my backups. My backup system is divided into three layers. Layer one is the broadest backup solution, backing up my entire filesystem to an external SSD. Layers two and three are much narrower, backing up only my critical files (such as my ssh and gpg keys) to **two** separate   locations.

## Layer 1

Layer 1 is my first line of defense for loss of data. It's an external SSD that I back up to using Time Machine, since I use a Mac. Time Machine is by far the best backup solution I've ever used. It essentially backs up the entirety of the Mac's filesystem to an external disk or NAS, and since it's built in to MacOS, it's very reliable. I've had to restore from a Time Machine backup three times, and each time it has worked flawlessly. If I lose my laptop or the filesystem somehow becomes corrupted, this is the first thing I reach for to restore everything.

## Layer 2

But what if something happens to the Time Machine drive? Say I accidentally drop it in water or the data on the drive becomes corrupted. What then? Well, in that case I'd like to have a copy of all my **important, sensitive** files. Some examples of this are my gpg keys, my ssh keys, and my 2FA recovery keys. These are files that, if lost, will lock me out of most of my accounts or services. They are very important, so I back them up to two sources. Layer 2 is a cloud-based Borg Backup of the aforementioned files on [BorgBase](https://www.borgbase.com/). The free plan is limited to 10GB, but that is more than enough for my purposes.

## Layer 3

But what if something happens to BorgBase? What if they decide to shut down the services or delete my server? That's where Layer 3 comes into play. I need to bring a small USB flash drive with me wherever I go for school anyway. Why not back up my files there? All I need to do is create an encrypted volume on it. I followed [this](https://sunknudsen.com/privacy-guides/how-to-back-up-and-encrypt-data-using-rsync-and-veracrypt-on-macos) guide to set up a backup system with Veracrypt and Rsync. I hope I'll never need to use this, but it's good to have it just in case.
