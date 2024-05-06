---
layout: default
title:  "Virt-Maker"
date:   2024-06-06 13:44:00
categories: projects
---

Revisiting Virt-Maker
=====================


It has been quite a long time since I pushed updates to Virt-Maker, but over the years and in the background I have been
modifying and updating the codebase.  There has been a lot of changes, I now have it in a state where I think the idea
is solidified.

## The History

### Anaconda and Kickstarts
The kernel of the idea came from my time as an system administrator in the early 2010s.  We had a fairly normalized
automation for installing EL based systems (mostly CentOS 5/6) using kickstarts.  We would have a set of application
servers, baremetal boxes that would run mostly labs, and we began to break up those applications and move to
a more conventional one application per server model.  The migration was fairly painless as we had a good set of
kickstarts that would install the base OS.  During this migration, we also began to formalize our application installs
into kickstarts as well.  These deployments were short-lived and would often be deleted after a month or so.

I eventually got the process down to an almost completely automated process where I could go into Hudson, set a few
configuration options, and the VM would be created, PXE booted, installed, and tested.  Everytime a changed was made
to the kickstarts, I would build a test VM and run the kickstart to make sure it passed testing.  It was at this point
I started experimenting some ideas on how the lifecycle of a VM could be managed.  Specifically the idea of a VM as
an appliance where the core system lived on one virtual disk, and the data lived on another.  The data disk would be
permanent and the system disk would be ephemeral.  In this way, if I needed to update the system, I could just poweroff
the VM, rename the OS disk, create a new OS disk, boot into PXE, and install the new OS.  If the install failed, I could
just poweroff the VM, rename the OS disk back to the original name, and boot back into the old OS.  This never got past
the experimental stage, but it was a fun idea.  It could have worked if the turnarounds were faster, but the PXE install
would take too long to be practical compared to a `yum update`.  I was transitioning to a new compant and the idea was
put on the shelf.

### Libguestfs
Fast forward to 2014, I was working at a new company and we were implementing a private cloud.  I was working on creating
base images for some of our various workloads that would run the control plane and was using Foreman to build a VM using
convetional methods, then taking the resulting disk image and uploading it to Glance.  The turnaround time was better
than manually building the images, but it was still slow.  I had heard of libguestfs and decided to give it a try.
I loved it.  I could create a gossamer base image once (or at least infrequently) and then pull it down, run a bash
script to install the packages I needed from my workstation, and then upload it to Glance.

This got me thinking about the idea of a VM as an appliance again.  I wanted to make a sort of translator that would
take a kickstart and convert it into a libguestfs script.  Then I could keep kickstart as the source of truth and
I could either PXE boot the VM or run the libguestfs script on my workstation.

### ks2guest
I started working on a project called ks2guest that would take a kickstart and convert it into a libguestfs script.
During the development of this project, I started defining a subset of kickstart that I would support.  Subsequently,
I would have to break the kickstarts that we had up into PXE only and ks2guest kickstarts.  This was becoming a pain
and after a certain point, the builds were taking just as long as the PXE installs.  It was a time for a change.

### Virt-Maker Version 0
Building off the ideas of ks2guest, I spent a few nights and weekend creating Virt-Maker.
I abandoned the idea of a kickstart subset and instead decided on create a DSL for defining VMs.  I would no longer be 
able to use the kickstarts as the source of truth, but that idea was becoming irrelevant at that time anyway.
The second idea I wanted was around making the build process faster.  For this, I used the qcow snapshot facility to
create a rolling hash of the build process.  In this way, I could take a base image, perform the system updates,
application installation, etc early in the build process, and then take network configuration, hostname, etc later in
the build process.  This meant that the very first build of an appliance would take ~30 minutes, but subsequent builds
would take less than a minute in most cases.  This was a huge win. Not only could I use the spec as a way of building
cloud images, but the same tool could be used to bake appliances.

I created the tool and pushed it to github and there it sat with little or no interest.  We didn't use it at work and no
one in the community seemed to care.  Docker had began to take over the world.  So in my homelab it sat.  I would make
incremental changes to it, but never bothered to push them to github.  I would use it to build my own appliances,
but that was about it.

### Virt-Maker Version 1
Fast forward to 2024, I had changed the codebase so much at this point that virtually everything had changed.  How the
build steps worked, how the snapshots worked, and even dropping the DSL for a YAML/Jinja based template system.  I found
myself making fewer and fewer changes to the codebase other than adding step runners.  The YAML api had been the same
for a few years at this point and I didn't see any changes needed.  At this point it's been using the new template spec
longer than the DSL in my lab.  So, I decided to clean up the codebase and push it to github with the major version of 1.

I'll be pushing to https://github.com/JosiahKerley/virt-maker once I have some documentation written up with cooresponding
examples.  I'll also be pushing to PYPI as "virt-maker" as well.  I'll be posting an update here once that is done and
once it is, feel free to check it out.  If you have a linux box with qemu-kvm and libguestfs, you should be able to build
all the examples, but if not, please let me know by opening an issue on github.

