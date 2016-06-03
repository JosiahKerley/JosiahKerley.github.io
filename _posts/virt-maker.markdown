---
layout: default
title:  "Virt-Maker"
date:   2015-08-12 17:50:00
categories: projects
---

Virt-Maker
==========

## Automated virtual machine build tool based around libguestfs.


I started `virt-maker` a while back, mostly to ease the creation of mutliple similar `QCOW` images for use in an `OpenStack` environment.
I had used packer to accomplish this goal, as well as wrapping some CI goodness around `Foreman`.  I really needed a tool that could take
a base qcow that was cloud-ready and just make minor changes to it so it could crank out specialized, prebuilt, ready-to-fly images.
Enter LibGuestFS.  Using `libguestfs`, specifically the tool `virt-customize` I was able to build a base application image in about
half the time as packer, and then loop through a few singleton steps and make multiple copies of images with all the right config-goodness.
This got me thinking, I could use the `QCOW` external snapshot functionality to create a forking tree on a per-step basis that would allow
me to create many different base/golden images from publicly available cloud images rapidly.

This is when I started `virt-maker` (https://github.com/JosiahKerley/virt-maker).  Over the course of about a week I was able to create a prototype
tool that could take in a simple build spec and build host images.

Effectively, it imports a starting image, usually a cloud image from http://docs.openstack.org/image-guide/obtain-images.html or natively from `virt-builder`,
and creates a snapshot of the image.  When the snapshot is created, it is uniquely identified my the hashsum of the string that constitutes the step
plus the hashsum of the previous step creating a rolling hash.  In this way, if the same work is ever attempted to be repeated, it will look to see if the work
was already completed before and if so, start from that point.  Now we're cooking.  Now I can use the long hardening script that I was using with `packer` but
now after the first time it is run it get's cached and is never needing to run again.  For a single image, using a pre-built cloud image has cut down the base
image creation time by about half, but the real value is when I need to deploy several images from that base image.  Normally in `Foreman` or `packer`, this would
take /forever/.

In one way it's analogous to `docker`.  Docker cuts every intermediate step into snapshots and changes made during build time only have to execute since at the
point in the process were the change was made. This allows for fast failure and so does `virt-maker`.

The current state of the is, well, bad.  My original intent was to keep it simple so that normal pythonistas could hop in and modify the core code and step providers
easily.  Thinking back now, I wish I had started with a more traditional OOP aproach.  This would make it easier to abstract away some core mechanisms and allow for
diverse backends.  I started drawing up new designs for how a "v.2" `virt-maker` would work internally, as well as things like meta-templates and a more straight-forward
input/modify/output chain (whereas now it is extremely linear).

Anyway, check it out.  Let me know what you think.  I have it published at https://github.com/JosiahKerley/virt-maker.