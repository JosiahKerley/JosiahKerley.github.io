---
layout: post
title:  "Building an enterprise backup solution with Bacula and NixOS"
date:   2025-04-17 00:00:00
series: "Immutable Enterprise Backup"
categories:
  - bacula-on-nixos
  - immutable-infrastructure
  - bacula
  - nixos
  - guides
---

!["The logos of the bacula and nixos project"](/assets/img/posts/2025-04-17-immutable-bacula/splash.png)

## Abstract
The goal of this series of posts is to provide an actionable reference for building an enterprise-grade backup solution using Bacula and NixOS.
This post will cover the following topics:
- What is enterprise-grade backup and why use Bacula?
- What is an immutable operating system and why use NixOS?
- Why use NixOS and Bacula pair so well together?

## First things first
### What is Enterprise Grade Backup?
When I think of enterprise-grade backup, I think of the following features:
  - Automatic backup scheduling
  - Mulit-OS support (Linux, Windows, MacOS)
  - Incremental and differential backups
  - Support for multiple file systems and storage media (tape, disk, network)
  - Support for backing up files as well as arbitrary streams of data (i.e., databases that dump to stdout)
  - The ability to orchestrate the backup so that snapshots, database dumps, setup/cleanup scripts, etc. can be run before and after the backup
  - Backups that are optimized for minimal restore time (i.e., if a restore needs to be done, the backed up data can be stored in a manner that enables minimal tape swapping and seeking)
  - Centralized management and monitoring
  - Highly customizable configuration options
  - Distributed architecture for scalability and availability

Since the early 2010s, Bacula has been my go to backup solution for building a disaster recovery service for enterprise environments.

Bacula (Backup, Archiving, and Recovery) is an open-source, enterprise-grade data protection system.
It provides a comprehensive solution for backing up and recovering critical data in the event of hardware failure, software corruption, or other disasters. 

Bacula is widely used in enterprise environments to protect critical data from various types of threats.
Its flexibility, scalability, and reliability make it a popular choice for organizations of all sizes. 

If you are coming from other backup solutions, Bacula may be a bit overwhelming at first, especially if you do not have a unix-like background.
Deploying Bacula in a large production environment can be a daunting task because it often feels less like you are working with a concise service and more like you are given a toolbox to build your own backup solution.
There are many moving parts to Bacula, but in this post, we will focus on a minimal setup that will allow you to get started with Bacula.

### What is an Immutable Operating System?
An immutable operating system is an operating system that is designed to be unchangeable after it has been deployed.
Specifically, there is a hard line drawn in the sand between the system's installation files and the data files.

You may be familiar with configuration management tools like Puppet, Chef, or Ansible, and the idea of a, "patrolling the system" to ensure that the system is in a known state.
Basically, you have a tool that runs and (hopefully) brings your system into a sane state if a configuration drift occurs.

However with an immutable operating system, the idea is that you do not need to worry about configuration drift because the system is designed to be unchangeable.
I.e., you 'compile' the system files and make them read-only.

Nowadays there are a few linux distributions that are immutable by default, such as Fedora Silverblue, OpenSUSE MicroOS, and NixOS.

In general, I prefer NixOS because it is a declarative operating system that allows you to define the entire system in code and apply changes (relatively) quickly in an atomic manner with rollback capabilities.
The language that NixOS uses is called Nix, and it is a functional programming language that is designed to be used for package management and system configuration.
There is no package manager per-se, but rather when a package needs to be installed, the nix code that produces the binary is imported into your system.
One advantage of this is that you can have multiple versions of a package installed at the same time, and you can easily roll back to a previous version if something goes wrong.
You can also have multiple versions of libraries installed at the same time so you don't run into 'dependency hell' when you are trying to install a new package.

### Why NixOS and Bacula?
One thing to know about Bacula is that it is a very config file driven system.  I mean _very_ config file driven.
This is what I mean by that:
```shell
[root@bacula:/etc/bacula]# wc -l ./*
  195 ./archive-jobs.conf
  395 ./backup-jobs.conf
    5 ./bconsole.conf
  355 ./clients.conf
    7 ./dynamic.conf
  333 ./filesets.conf
   79 ./jobdefs.conf
   40 ./jobs.conf
   77 ./pools.conf
   43 ./schedules.conf
   32 ./storages.conf
 1561 total
```
Those are the line counts for most (but not all -- we'll get into that later) of the config files on a system I operate to backup ~15 systems.
You may be saying to yourself, "Jo, that is a lot of config files, why would I want to use Bacula?".
For one, Bacula is a very powerful and flexible.  Those config files represent a fine-tuned backup system that does exactly what I want it to do in the way that I want it to do it.
I can fearlessly hop onto one of those systems and delete the database and feel confident that I can restore it from the backup in a couple of minutes.
And if I found out that a file from a month ago was corrupted, I can go back and restore it from the backup and restore it in less than an hour.
But you are right, that is a lot of config files to manage.

That's were NixOS comes in.  One of the beauties of NixOS is that it is a declarative system, so once I have a tuned-in nix code that generates the config files, I can just run `nixos-rebuild switch` and it will generate the config files for me.
If something breaks, I can just roll back to the previous version of the system and it will be in a known state.
Plus, the way I have developed the system, adding clients, changing schedules, modifying jobs, etc. have been abstracted away into a site-constant that is easy to modify and the code handles the rest.


## Conclusion
In this post, we have covered the basics of Bacula and NixOS, and how they can be used together to create an enterprise-grade backup solution.
In the next post, we will go over setting up a lab and installing NixOS from an ISO image.
