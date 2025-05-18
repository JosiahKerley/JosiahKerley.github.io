---
layout: post
title:  "Virt-Maker Lightning Talk"
date:   2025-02-14 17:50:00
fallback_thumbnail: /assets/img/posts/2025-02-14-virt-maker-lightning-talk/splash.png
categories:
  - virt-maker
  - projects
  - presentations
---

<center><iframe width="1024" height="768" src="http://www.youtube.com/embed/hIJjJHF5ehE" frameborder="0" allowfullscreen></iframe></center>

In this lightning talk, I present the Virt-Maker project.

Virt-Maker, a disk image building tool for GNU/Linux that uses snapshots for aggressive caching and fast turnaround using simple procedural code.
In traditional virtual machine infrastructure you may find yourself manually installing an iso for golden image creation just as you would on bare metal.
But the problem you run into is inconsistent manual builds and wasted time even when a golden image is created and used as a template for cloning from.
Virt-maker provides a way to define input sources such as generic cloud images, Virt-builder images, or even isos consistently and quickly by declaring your build steps with parameters using a simple and readable YAML syntax.

Like a blueprint that automatically builds all your traditional Vms, cloud images, and even source images for disk based baremetal deploying or booting from iscsi.

It is free and open source and writen in python and can installed on a distro like Rockylinux with just a few steps 

## Installation on GNU/Linux system with virtualization support

For this demo, let’s run it in podman.  Start the podman instance with rocky.
```bash
podman run \
  --privileged \
  --rm \
  -it \
    rockylinux:9 \
      /bin/bash
```

Install EPEL.
```bash
dnf install -y epel-release
```

Install the preq packages.
```bash
dnf install -y python python-pip qemu-kvm guestfs-tools wget xz lz4 pv vim
```

Install virt-maker.
```bash
pip install \
  https://github.com/JosiahKerley/virt-maker/archive/refs/heads/master.zip
```

## Creating and building a spec

Create a YAML file that pulls down the Rocky 9 generic cloud image, installs some packages, and exports a qcow2 image.
```bash
vim virtmaker.yml
```

```yaml
spec:
  import:
    download:
      url: >-
        https://download.rockylinux.org/pub/rocky/9/images/x86_64/Rocky-9-GenericCloud.latest.x86_64.qcow2
  steps:
    - install:
      - neofetch
    - hostname: lightning.local
  export:
    qcow2:
      path: lightning.qcow2
```

Build the image.
```bash
virt-maker build -f virtmaker.yml
```
Output:
```shell
[ FILE ] virtmaker.yml: Starting
[IMPORT] virtmaker.yml: {'url': 'https://download.rockylinux.org/pub/rocky/9/images/x86_64/Rocky-9-GenericCloud.latest.x86_64.qcow2'}
--2025-02-11 21:43:31--  https://download.rockylinux.org/pub/rocky/9/images/x86_64/Rocky-9-GenericCloud.latest.x86_64.qcow2
Resolving download.rockylinux.org (download.rockylinux.org)... 199.232.198.132, 199.232.194.132, 2a04:4e42:4c::644, ...
Connecting to download.rockylinux.org (download.rockylinux.org)|199.232.198.132|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 609812480 (582M) [application/octet-stream]
Saving to: ‘/root/.cache/virt-maker/b3b6e0e8761ecd0cdc4a2f6db40b810c.qcow2_in-progress’

/root/.cache/virt-maker/b3b6e0e8761ecd0cdc4a2f6db4 100%[==============================================================================================================>] 581.56M  8.67MB/s    in 1m 42s  

2025-05-17 21:45:14 (5.68 MB/s) - ‘/root/.cache/virt-maker/b3b6e0e8761ecd0cdc4a2f6db40b810c.qcow2_in-progress’ saved [609812480/609812480]

[INSTAL] virtmaker.yml: ['neofetch']
[   0.0] Examining the guest ...
[  24.7] Setting a random seed
[  24.8] Setting the machine ID in /etc/machine-id
[  24.8] Installing packages: neofetch
Rocky Linux 9 - BaseOS                          1.0 MB/s | 2.3 MB     00:02    
Rocky Linux 9 - AppStream                       4.3 MB/s | 8.4 MB     00:01    
Rocky Linux 9 - Extras                           36 kB/s |  16 kB     00:00    
No match for argument: neofetch
Error: Unable to find a match: neofetch
virt-customize: error: dnf -y install 'neofetch': command exited with an 
error

If reporting bugs, run virt-customize with debugging enabled and include 
the complete output:

  virt-customize -v -x [...]
step failed, not finalizing snapshot
```

Notice that the installation of the `nefetch` package failed.  This is because the EPEL repository is not enabled by default in Rocky Linux 9.  To fix this, we need to enable the EPEL repository in the YAML file.

Modify the YAML file to include the EPEL repository.
```yaml
spec:
  import:
    download:
      url: >-
        https://download.rockylinux.org/pub/rocky/9/images/x86_64/Rocky-9-GenericCloud.latest.x86_64.qcow2
  steps:
    - install:
      - epel-release
    - install:
      - neofetch
    - hostname: lightning.local
  export:
    qcow2:
      path: lightning.qcow2
```

Run again:
```shell
[root@c373a7f61192 /]# time virt-maker build -f virtmaker.yml
[ FILE ] virtmaker.yml: Starting
[IMPORT] virtmaker.yml: {'url': 'https://download.rockylinux.org/pub/rocky/9/images/x86_64/Rocky-9-GenericCloud.latest.x86_64.qcow2'}
[INSTAL] virtmaker.yml: ['epel-release']
[   0.0] Examining the guest ...
[  12.3] Setting a random seed
[  12.3] Setting the machine ID in /etc/machine-id
[  12.3] Installing packages: epel-release
[  25.8] Finishing off
[INSTAL] virtmaker.yml: ['neofetch']
[   0.0] Examining the guest ...
[  12.8] Setting a random seed
[  12.9] Installing packages: neofetch
[ 129.6] Finishing off
[HOSTNA] virtmaker.yml: lightning.local
[   0.0] Examining the guest ...
[  13.0] Setting a random seed
[  13.1] Setting the hostname: lightning.local
[  13.2] Finishing off
[EXPORT] virtmaker.yml: {'path': 'lightning.qcow2', 'create_dir': False, 'level': 1, 'sparsify': False}
    (100.00/100%)

real    3m14.295s
user    1m57.053s
sys     0m36.948s
```

Notice that we were able to skip downloading the qcow2 image again since it is already cached.

This time when the command finishes, we can see that the image was built successfully and a qcow2 image was created.
```shell
[root@c373a7f61192 /]# file -sL lightning.qcow2 
lightning.qcow2: QEMU QCOW2 Image (v3), 10737418240 bytes
```

Now let's say a new request comes in and we need to add some new packages to the image.  We can do this by modifying the YAML file again.
```yaml
spec:
  import:
    download:
      url: >-
        https://download.rockylinux.org/pub/rocky/9/images/x86_64/Rocky-9-GenericCloud.latest.x86_64.qcow2
  steps:
    - install:
      - epel-release
    - install:
      - neofetch
      - vim
      - nano
    - hostname: lightning.local
  export:
    qcow2:
      path: lightning.qcow2
```

Since we successfully completed the previous steps, we can skip them and just use the snapshots and build a new image with the new packages.
```shell
[root@c373a7f61192 /]# time virt-maker build -f virtmaker.yml
[ FILE ] virtmaker.yml: Starting
[IMPORT] virtmaker.yml: {'url': 'https://download.rockylinux.org/pub/rocky/9/images/x86_64/Rocky-9-GenericCloud.latest.x86_64.qcow2'}
[INSTAL] virtmaker.yml: ['epel-release']
[INSTAL] virtmaker.yml: ['neofetch', 'vim', 'nano']
[   0.0] Examining the guest ...
[  11.0] Setting a random seed
[  11.0] Installing packages: neofetch vim nano
[ 130.5] Finishing off
[HOSTNA] virtmaker.yml: lightning.local
[   0.0] Examining the guest ...
[  13.0] Setting a random seed
[  13.0] Setting the hostname: lightning.local
[  13.2] Finishing off
[EXPORT] virtmaker.yml: {'path': 'lightning.qcow2', 'create_dir': False, 'level': 1, 'sparsify': False}
    (100.00/100%)

real    2m54.768s
user    1m42.602s
sys     0m26.565s
```

Now let’s say we want to change the domain in the hostname from `local` to `mcqueen`.  We can do this by modifying the YAML file again.
```yaml
spec:
  import:
    download:
      url: >-
        https://download.rockylinux.org/pub/rocky/9/images/x86_64/Rocky-9-GenericCloud.latest.x86_64.qcow2
  steps:
    - install:
      - epel-release
    - install:
      - neofetch
      - vim
      - nano
    - hostname: lightning.mcqueen ## Note the change
  export:
    qcow2:
      path: lightning.qcow2
```

Notice that we are skipping the `install` steps since we already have the packages installed when running the build command again.
```shell
[root@c373a7f61192 /]# time virt-maker build -f virtmaker.yml
[ FILE ] virtmaker.yml: Starting
[IMPORT] virtmaker.yml: {'url': 'https://download.rockylinux.org/pub/rocky/9/images/x86_64/Rocky-9-GenericCloud.latest.x86_64.qcow2'}
[INSTAL] virtmaker.yml: ['epel-release']
[INSTAL] virtmaker.yml: ['neofetch', 'vim', 'nano']
[HOSTNA] virtmaker.yml: lightning.mcqueen
[   0.0] Examining the guest ...
[  10.6] Setting a random seed
[  10.7] Setting the hostname: lightning.mcqueen
[  10.8] Finishing off
[EXPORT] virtmaker.yml: {'path': 'lightning.qcow2', 'create_dir': False, 'level': 1, 'sparsify': False}
    (100.00/100%)

real    0m38.221s
user    0m15.289s
sys     0m9.070s
```

But what if we make no change and just run the command again?
```shell
[root@c373a7f61192 /]# time virt-maker build -f virtmaker.yml
[ FILE ] virtmaker.yml: Starting
[IMPORT] virtmaker.yml: {'url': 'https://download.rockylinux.org/pub/rocky/9/images/x86_64/Rocky-9-GenericCloud.latest.x86_64.qcow2'}
[INSTAL] virtmaker.yml: ['epel-release']
[INSTAL] virtmaker.yml: ['neofetch', 'vim', 'nano']
[HOSTNA] virtmaker.yml: lightning.mcqueen
[EXPORT] virtmaker.yml: {'path': 'lightning.qcow2', 'create_dir': False, 'level': 1, 'sparsify': False}

real    0m1.258s
user    0m0.834s
sys     0m0.165s
```

Since nothing in the spec changed, there's nothing to do.


## Parameterized builds

The YAML file can also be parameterized.
This allows you to define variables in the YAML file and use them in the spec.
This is useful for defining things like the hostname, image name, and other parameters that may change between builds.

Any value in the spec can be evaluated as a Jinja2 template.

In this example, let's define a parameter for the hostname and use it in the spec to set the hostname and the image name.
```yaml
{% raw %}
params: ## Adding parameters
  hostname: lightning.mcqueen ## Setting a default value
spec:
  import:
    download:
      url: >-
        https://download.rockylinux.org/pub/rocky/9/images/x86_64/Rocky-9-GenericCloud.latest.x86_64.qcow2
  steps:
    - install:
      - epel-release
    - install:
      - neofetch
      - vim
      - nano
    - hostname: '{{ hostname }}'
  export:
    qcow2:
      path: '{{ hostname }}.qcow2'
{% endraw %}
```

Since we already satisfied the hostname, nothing new was ran, but since we made the export image path a variable, now we have a new image.
```shell
[root@c373a7f61192 /]# time virt-maker build -f virtmaker.yml
[ FILE ] virtmaker.yml: Starting
[IMPORT] virtmaker.yml: {'url': 'https://download.rockylinux.org/pub/rocky/9/images/x86_64/Rocky-9-GenericCloud.latest.x86_64.qcow2'}
[INSTAL] virtmaker.yml: ['epel-release']
[INSTAL] virtmaker.yml: ['neofetch', 'vim', 'nano']
[HOSTNA] virtmaker.yml: lightning.mcqueen
[EXPORT] virtmaker.yml: {'path': 'lightning.mcqueen.qcow2', 'create_dir': False, 'level': 1, 'sparsify': False}
    (100.00/100%)

real    0m21.986s
user    0m8.219s
sys     0m6.560s
```

And now we have the original qcow2 as well as the one with the parameterized filename.
```shell
[root@c373a7f61192 /]# file -sL *.qcow2
lightning.mcqueen.qcow2: QEMU QCOW2 Image (v3), 10737418240 bytes
lightning.qcow2:         QEMU QCOW2 Image (v3), 10737418240 bytes
```

Now let's build a new image with a different hostname.
```shell
[root@c373a7f61192 /]# time virt-maker build -f virtmaker.yml -p hostname=steve.mcqueen
[ FILE ] virtmaker.yml: Starting
[IMPORT] virtmaker.yml: {'url': 'https://download.rockylinux.org/pub/rocky/9/images/x86_64/Rocky-9-GenericCloud.latest.x86_64.qcow2'}
[INSTAL] virtmaker.yml: ['epel-release']
[INSTAL] virtmaker.yml: ['neofetch', 'vim', 'nano']
[HOSTNA] virtmaker.yml: steve.mcqueen
[   0.0] Examining the guest ...
[   9.9] Setting a random seed
[   9.9] Setting the hostname: steve.mcqueen
[  10.0] Finishing off
[EXPORT] virtmaker.yml: {'path': 'steve.mcqueen.qcow2', 'create_dir': False, 'level': 1, 'sparsify': False}
    (100.00/100%)

real    0m31.482s
user    0m14.567s
sys     0m8.530s
```

And now we have three qcow2 images.
```shell
[root@c373a7f61192 /]# file -sL *.qcow2
lightning.mcqueen.qcow2: QEMU QCOW2 Image (v3), 10737418240 bytes
lightning.qcow2:         QEMU QCOW2 Image (v3), 10737418240 bytes
steve.mcqueen.qcow2:     QEMU QCOW2 Image (v3), 10737418240 bytes
```
