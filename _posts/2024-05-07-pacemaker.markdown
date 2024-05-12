---
layout: post
title:  "Setting up a basic Pacemaker cluster on Rocky Linux 9"
date:   2024-05-07 12:00:00
categories:
  - pacemaker
  - guides
toc: true
---
!["A datacenter being struck from a meteorite"](/assets/img/other/datacenter-struck-by-meteorite.png)

## TL;DR
{: .no_toc }
Pacemaker is used to automatically manage resources such as systemd units, IP addresses, and filesystems for a cluster.

* TOC
{:toc}

## What is Pacemaker?
Pacemaker is a high-availability cluster resource manager.  It is used to build and manage highly available services.
It is often used in database clusters, web services, and other services that need to be highly available.
Often deployed in telecommunications, banking, and other industries where high availability is a requirement.

## Deploying {#deploy-pacemaker-cluster}
### All nodes

#### Enable HA repository
```bash
sudo dnf config-manager --set-enabled highavailability
```

#### Install pacemaker packages
```bash
sudo dnf install -y pacemaker pcs
```

<div class="codehider">
toggle console output:
<div class="hidecode" markdown="1">
```collapse
Rocky Linux 9 - High Availability               259 kB/s | 250 kB     00:00    
Dependencies resolved.
==========================================================================================
 Package                          Arch    Version                  Repository         Size
==========================================================================================
Installing:
 pacemaker                        x86_64  2.1.7-5.el9_4            highavailability  465 k
 pcs                              x86_64  0.11.7-2.el9_4           highavailability  7.9 M
Installing dependencies:
 bzip2                            x86_64  1.0.8-8.el9              baseos             52 k
 corosync                         x86_64  3.1.8-1.el9              highavailability  262 k
 corosynclib                      x86_64  3.1.8-1.el9              appstream          50 k
 device-mapper-event              x86_64  9:1.02.197-2.el9         baseos             33 k
 device-mapper-event-libs         x86_64  9:1.02.197-2.el9         baseos             31 k
 device-mapper-persistent-data    x86_64  1.0.9-3.el9_4            baseos            1.0 M
 libaio                           x86_64  0.3.111-13.el9           baseos             23 k
 libknet1                         x86_64  1.28-1.el9               highavailability   79 k
 libknet1-compress-bzip2-plugin   x86_64  1.28-1.el9               highavailability   11 k
 libknet1-compress-lz4-plugin     x86_64  1.28-1.el9               highavailability   13 k
 libknet1-compress-lzma-plugin    x86_64  1.28-1.el9               highavailability   12 k
 libknet1-compress-lzo2-plugin    x86_64  1.28-1.el9               highavailability   12 k
 libknet1-compress-plugins-all    x86_64  1.28-1.el9               highavailability  7.3 k
 libknet1-compress-zlib-plugin    x86_64  1.28-1.el9               highavailability   11 k
 libknet1-compress-zstd-plugin    x86_64  1.28-1.el9               highavailability   11 k
 libknet1-crypto-nss-plugin       x86_64  1.28-1.el9               highavailability   16 k
 libknet1-crypto-openssl-plugin   x86_64  1.28-1.el9               highavailability   14 k
 libknet1-crypto-plugins-all      x86_64  1.28-1.el9               highavailability  7.1 k
 libknet1-plugins-all             x86_64  1.28-1.el9               highavailability  7.1 k
 libnozzle1                       x86_64  1.28-1.el9               highavailability   29 k
 libpkgconf                       x86_64  1.7.3-10.el9             baseos             35 k
 libqb                            x86_64  2.0.8-1.el9              appstream          91 k
 libxslt                          x86_64  1.1.34-9.el9             appstream         240 k
 lvm2                             x86_64  9:2.03.23-2.el9          baseos            1.5 M
 lvm2-libs                        x86_64  9:2.03.23-2.el9          baseos            1.0 M
 net-snmp-libs                    x86_64  1:5.9.1-13.el9           appstream         740 k
 net-tools                        x86_64  2.0-0.62.20160912git.el9 baseos            292 k
 nss-tools                        x86_64  3.90.0-7.el9_4           appstream         431 k
 pacemaker-cli                    x86_64  2.1.7-5.el9_4            highavailability  332 k
 pacemaker-cluster-libs           x86_64  2.1.7-5.el9_4            highavailability  128 k
 pacemaker-libs                   x86_64  2.1.7-5.el9_4            highavailability  774 k
 pacemaker-schemas                noarch  2.1.7-5.el9_4            highavailability   80 k
 perl-TimeDate                    noarch  1:2.33-6.el9             appstream          51 k
 pkgconf                          x86_64  1.7.3-10.el9             baseos             40 k
 pkgconf-m4                       noarch  1.7.3-10.el9             baseos             14 k
 pkgconf-pkg-config               x86_64  1.7.3-10.el9             baseos             10 k
 python3-cffi                     x86_64  1.14.5-5.el9             baseos            241 k
 python3-cryptography             x86_64  36.0.1-4.el9             baseos            1.2 M
 python3-lxml                     x86_64  4.6.5-3.el9              appstream         1.2 M
 python3-ply                      noarch  3.11-14.el9.0.1          baseos            103 k
 python3-pycparser                noarch  2.20-6.el9               baseos            124 k
 python3-pycurl                   x86_64  7.43.0.6-8.el9           appstream         173 k
 python3-pyparsing                noarch  2.4.7-9.el9              baseos            150 k
 resource-agents                  x86_64  4.10.0-52.el9            highavailability  458 k
 ruby                             x86_64  3.0.4-161.el9            appstream          38 k
 ruby-libs                        x86_64  3.0.4-161.el9            appstream         3.2 M
 rubygem-io-console               x86_64  0.5.7-161.el9            appstream          22 k
 rubygem-json                     x86_64  2.5.1-161.el9            appstream          51 k
 rubygem-psych                    x86_64  3.3.2-161.el9            appstream          48 k
 rubygem-rexml                    noarch  3.2.5-161.el9            appstream          92 k
 rubygems                         noarch  3.2.33-161.el9           appstream         253 k
Installing weak dependencies:
 ruby-default-gems                noarch  3.0.4-161.el9            appstream          29 k
 rubygem-bigdecimal               x86_64  3.0.0-161.el9            appstream          51 k
 rubygem-bundler                  noarch  2.2.33-161.el9           appstream         369 k
 rubygem-rdoc                     noarch  6.3.3-161.el9            appstream         396 k

Transaction Summary
==========================================================================================
Install  57 Packages

Total download size: 24 M
Installed size: 77 M
Downloading Packages:
(1/57): libaio-0.3.111-13.el9.x86_64.rpm         74 kB/s |  23 kB     00:00    
(2/57): bzip2-1.0.8-8.el9.x86_64.rpm            132 kB/s |  52 kB     00:00    
(3/57): python3-pyparsing-2.4.7-9.el9.noarch.rp 347 kB/s | 150 kB     00:00    
(4/57): pkgconf-pkg-config-1.7.3-10.el9.x86_64.  50 kB/s |  10 kB     00:00    
(5/57): pkgconf-m4-1.7.3-10.el9.noarch.rpm       63 kB/s |  14 kB     00:00    
(6/57): pkgconf-1.7.3-10.el9.x86_64.rpm         132 kB/s |  40 kB     00:00    
(7/57): libpkgconf-1.7.3-10.el9.x86_64.rpm      127 kB/s |  35 kB     00:00    
(8/57): device-mapper-persistent-data-1.0.9-3.e 1.1 MB/s | 1.0 MB     00:00    
(9/57): net-tools-2.0-0.62.20160912git.el9.x86_ 365 kB/s | 292 kB     00:00    
(10/57): lvm2-libs-2.03.23-2.el9.x86_64.rpm     1.1 MB/s | 1.0 MB     00:00    
(11/57): device-mapper-event-libs-1.02.197-2.el 107 kB/s |  31 kB     00:00    
(12/57): device-mapper-event-1.02.197-2.el9.x86 111 kB/s |  33 kB     00:00    
(13/57): lvm2-2.03.23-2.el9.x86_64.rpm          1.2 MB/s | 1.5 MB     00:01    
(14/57): python3-cffi-1.14.5-5.el9.x86_64.rpm   295 kB/s | 241 kB     00:00    
(15/57): python3-ply-3.11-14.el9.0.1.noarch.rpm 144 kB/s | 103 kB     00:00    
(16/57): python3-cryptography-36.0.1-4.el9.x86_ 941 kB/s | 1.2 MB     00:01    
(17/57): python3-pycparser-2.20-6.el9.noarch.rp 217 kB/s | 124 kB     00:00    
(18/57): perl-TimeDate-2.33-6.el9.noarch.rpm    103 kB/s |  51 kB     00:00    
(19/57): python3-pycurl-7.43.0.6-8.el9.x86_64.r 326 kB/s | 173 kB     00:00    
(20/57): python3-lxml-4.6.5-3.el9.x86_64.rpm    1.4 MB/s | 1.2 MB     00:00    
(21/57): net-snmp-libs-5.9.1-13.el9.x86_64.rpm  1.0 MB/s | 740 kB     00:00    
(22/57): rubygem-rexml-3.2.5-161.el9.noarch.rpm 246 kB/s |  92 kB     00:00    
(23/57): rubygems-3.2.33-161.el9.noarch.rpm     438 kB/s | 253 kB     00:00    
(24/57): ruby-default-gems-3.0.4-161.el9.noarch 127 kB/s |  29 kB     00:00    
(25/57): rubygem-rdoc-6.3.3-161.el9.noarch.rpm  608 kB/s | 396 kB     00:00    
(26/57): rubygem-bundler-2.2.33-161.el9.noarch. 553 kB/s | 369 kB     00:00    
(27/57): libxslt-1.1.34-9.el9.x86_64.rpm        469 kB/s | 240 kB     00:00    
(28/57): corosynclib-3.1.8-1.el9.x86_64.rpm     184 kB/s |  50 kB     00:00    
(29/57): nss-tools-3.90.0-7.el9_4.x86_64.rpm    882 kB/s | 431 kB     00:00    
(30/57): rubygem-psych-3.3.2-161.el9.x86_64.rpm 177 kB/s |  48 kB     00:00    
(31/57): rubygem-io-console-0.5.7-161.el9.x86_6 108 kB/s |  22 kB     00:00    
(32/57): rubygem-json-2.5.1-161.el9.x86_64.rpm  178 kB/s |  51 kB     00:00    
(33/57): rubygem-bigdecimal-3.0.0-161.el9.x86_6 158 kB/s |  51 kB     00:00    
(34/57): ruby-3.0.4-161.el9.x86_64.rpm          135 kB/s |  38 kB     00:00    
(35/57): libqb-2.0.8-1.el9.x86_64.rpm           219 kB/s |  91 kB     00:00    
(36/57): resource-agents-4.10.0-52.el9.x86_64.r 829 kB/s | 458 kB     00:00    
(37/57): ruby-libs-3.0.4-161.el9.x86_64.rpm     2.7 MB/s | 3.2 MB     00:01    
(38/57): corosync-3.1.8-1.el9.x86_64.rpm        516 kB/s | 262 kB     00:00    
(39/57): pacemaker-schemas-2.1.7-5.el9_4.noarch 212 kB/s |  80 kB     00:00    
(40/57): pacemaker-cluster-libs-2.1.7-5.el9_4.x 302 kB/s | 128 kB     00:00    
(41/57): pacemaker-cli-2.1.7-5.el9_4.x86_64.rpm 677 kB/s | 332 kB     00:00    
(42/57): pacemaker-libs-2.1.7-5.el9_4.x86_64.rp 1.1 MB/s | 774 kB     00:00    
(43/57): libnozzle1-1.28-1.el9.x86_64.rpm       108 kB/s |  29 kB     00:00    
(44/57): libknet1-plugins-all-1.28-1.el9.x86_64  32 kB/s | 7.1 kB     00:00    
(45/57): libknet1-crypto-plugins-all-1.28-1.el9  44 kB/s | 7.1 kB     00:00    
(46/57): pacemaker-2.1.7-5.el9_4.x86_64.rpm     809 kB/s | 465 kB     00:00    
(47/57): libknet1-crypto-openssl-plugin-1.28-1.  89 kB/s |  14 kB     00:00    
(48/57): libknet1-compress-zstd-plugin-1.28-1.e  49 kB/s |  11 kB     00:00    
(49/57): libknet1-crypto-nss-plugin-1.28-1.el9.  40 kB/s |  16 kB     00:00    
(50/57): libknet1-compress-zlib-plugin-1.28-1.e  29 kB/s |  11 kB     00:00    
(51/57): libknet1-compress-plugins-all-1.28-1.e  26 kB/s | 7.3 kB     00:00    
(52/57): libknet1-compress-lzo2-plugin-1.28-1.e  55 kB/s |  12 kB     00:00    
(53/57): libknet1-compress-lzma-plugin-1.28-1.e  42 kB/s |  12 kB     00:00    
(54/57): libknet1-compress-lz4-plugin-1.28-1.el  54 kB/s |  13 kB     00:00    
(55/57): libknet1-compress-bzip2-plugin-1.28-1.  65 kB/s |  11 kB     00:00    
(56/57): libknet1-1.28-1.el9.x86_64.rpm         196 kB/s |  79 kB     00:00    
(57/57): pcs-0.11.7-2.el9_4.x86_64.rpm          4.0 MB/s | 7.9 MB     00:01    
--------------------------------------------------------------------------------
Total                                           2.2 MB/s |  24 MB     00:11     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : ruby-libs-3.0.4-161.el9.x86_64                        1/57 
  Installing       : rubygem-psych-3.3.2-161.el9.x86_64                    2/57 
  Installing       : rubygem-json-2.5.1-161.el9.x86_64                     3/57 
  Installing       : rubygem-io-console-0.5.7-161.el9.x86_64               4/57 
  Installing       : rubygem-bigdecimal-3.0.0-161.el9.x86_64               5/57 
  Installing       : rubygem-rdoc-6.3.3-161.el9.noarch                     6/57 
  Installing       : rubygem-bundler-2.2.33-161.el9.noarch                 7/57 
  Installing       : ruby-default-gems-3.0.4-161.el9.noarch                8/57 
  Installing       : ruby-3.0.4-161.el9.x86_64                             9/57 
  Installing       : rubygems-3.2.33-161.el9.noarch                       10/57 
  Installing       : libknet1-1.28-1.el9.x86_64                           11/57 
  Running scriptlet: libknet1-1.28-1.el9.x86_64                           11/57 
  Installing       : libqb-2.0.8-1.el9.x86_64                             12/57 
  Installing       : libxslt-1.1.34-9.el9.x86_64                          13/57 
  Installing       : corosynclib-3.1.8-1.el9.x86_64                       14/57 
  Installing       : device-mapper-event-libs-9:1.02.197-2.el9.x86_64     15/57 
  Installing       : libknet1-crypto-nss-plugin-1.28-1.el9.x86_64         16/57 
  Installing       : libaio-0.3.111-13.el9.x86_64                         17/57 
  Installing       : device-mapper-event-9:1.02.197-2.el9.x86_64          18/57 
  Running scriptlet: device-mapper-event-9:1.02.197-2.el9.x86_64          18/57 
Created symlink /etc/systemd/system/sockets.target.wants/dm-event.socket → /usr/lib/systemd/system/dm-event.socket.

  Installing       : lvm2-libs-9:2.03.23-2.el9.x86_64                     19/57 
  Installing       : python3-lxml-4.6.5-3.el9.x86_64                      20/57 
  Installing       : libknet1-crypto-openssl-plugin-1.28-1.el9.x86_64     21/57 
  Installing       : libknet1-crypto-plugins-all-1.28-1.el9.x86_64        22/57 
  Installing       : libknet1-compress-zstd-plugin-1.28-1.el9.x86_64      23/57 
  Installing       : libknet1-compress-zlib-plugin-1.28-1.el9.x86_64      24/57 
  Installing       : libknet1-compress-lzo2-plugin-1.28-1.el9.x86_64      25/57 
  Installing       : libknet1-compress-lzma-plugin-1.28-1.el9.x86_64      26/57 
  Installing       : libknet1-compress-lz4-plugin-1.28-1.el9.x86_64       27/57 
  Installing       : libknet1-compress-bzip2-plugin-1.28-1.el9.x86_64     28/57 
  Installing       : libknet1-compress-plugins-all-1.28-1.el9.x86_64      29/57 
  Installing       : libknet1-plugins-all-1.28-1.el9.x86_64               30/57 
  Installing       : rubygem-rexml-3.2.5-161.el9.noarch                   31/57 
  Installing       : libnozzle1-1.28-1.el9.x86_64                         32/57 
  Running scriptlet: libnozzle1-1.28-1.el9.x86_64                         32/57 
  Installing       : nss-tools-3.90.0-7.el9_4.x86_64                      33/57 
  Installing       : net-snmp-libs-1:5.9.1-13.el9.x86_64                  34/57 
  Installing       : corosync-3.1.8-1.el9.x86_64                          35/57 
  Running scriptlet: corosync-3.1.8-1.el9.x86_64                          35/57 
  Installing       : python3-pycurl-7.43.0.6-8.el9.x86_64                 36/57 
  Installing       : perl-TimeDate-1:2.33-6.el9.noarch                    37/57 
  Installing       : python3-ply-3.11-14.el9.0.1.noarch                   38/57 
  Installing       : python3-pycparser-2.20-6.el9.noarch                  39/57 
  Installing       : python3-cffi-1.14.5-5.el9.x86_64                     40/57 
  Installing       : python3-cryptography-36.0.1-4.el9.x86_64             41/57 
  Installing       : net-tools-2.0-0.62.20160912git.el9.x86_64            42/57 
  Running scriptlet: net-tools-2.0-0.62.20160912git.el9.x86_64            42/57 
  Installing       : libpkgconf-1.7.3-10.el9.x86_64                       43/57 
  Installing       : pkgconf-1.7.3-10.el9.x86_64                          44/57 
  Installing       : pkgconf-m4-1.7.3-10.el9.noarch                       45/57 
  Installing       : pkgconf-pkg-config-1.7.3-10.el9.x86_64               46/57 
  Installing       : pacemaker-schemas-2.1.7-5.el9_4.noarch               47/57 
  Running scriptlet: pacemaker-libs-2.1.7-5.el9_4.x86_64                  48/57 
  Installing       : pacemaker-libs-2.1.7-5.el9_4.x86_64                  48/57 
  Installing       : pacemaker-cluster-libs-2.1.7-5.el9_4.x86_64          49/57 
  Installing       : device-mapper-persistent-data-1.0.9-3.el9_4.x86_64   50/57 
  Installing       : lvm2-9:2.03.23-2.el9.x86_64                          51/57 
  Running scriptlet: lvm2-9:2.03.23-2.el9.x86_64                          51/57 
Created symlink /etc/systemd/system/sysinit.target.wants/lvm2-monitor.service → /usr/lib/systemd/system/lvm2-monitor.service.
Created symlink /etc/systemd/system/sysinit.target.wants/lvm2-lvmpolld.socket → /usr/lib/systemd/system/lvm2-lvmpolld.socket.

  Installing       : resource-agents-4.10.0-52.el9.x86_64                 52/57 
  Installing       : python3-pyparsing-2.4.7-9.el9.noarch                 53/57 
  Installing       : bzip2-1.0.8-8.el9.x86_64                             54/57 
  Installing       : pacemaker-2.1.7-5.el9_4.x86_64                       55/57 
  Running scriptlet: pacemaker-2.1.7-5.el9_4.x86_64                       55/57 
  Installing       : pacemaker-cli-2.1.7-5.el9_4.x86_64                   56/57 
  Running scriptlet: pacemaker-cli-2.1.7-5.el9_4.x86_64                   56/57 
  Installing       : pcs-0.11.7-2.el9_4.x86_64                            57/57 
  Running scriptlet: pcs-0.11.7-2.el9_4.x86_64                            57/57 
  Verifying        : bzip2-1.0.8-8.el9.x86_64                              1/57 
  Verifying        : libaio-0.3.111-13.el9.x86_64                          2/57 
  Verifying        : python3-pyparsing-2.4.7-9.el9.noarch                  3/57 
  Verifying        : device-mapper-persistent-data-1.0.9-3.el9_4.x86_64    4/57 
  Verifying        : pkgconf-pkg-config-1.7.3-10.el9.x86_64                5/57 
  Verifying        : pkgconf-m4-1.7.3-10.el9.noarch                        6/57 
  Verifying        : pkgconf-1.7.3-10.el9.x86_64                           7/57 
  Verifying        : libpkgconf-1.7.3-10.el9.x86_64                        8/57 
  Verifying        : net-tools-2.0-0.62.20160912git.el9.x86_64             9/57 
  Verifying        : lvm2-libs-9:2.03.23-2.el9.x86_64                     10/57 
  Verifying        : lvm2-9:2.03.23-2.el9.x86_64                          11/57 
  Verifying        : device-mapper-event-libs-9:1.02.197-2.el9.x86_64     12/57 
  Verifying        : device-mapper-event-9:1.02.197-2.el9.x86_64          13/57 
  Verifying        : python3-cryptography-36.0.1-4.el9.x86_64             14/57 
  Verifying        : python3-cffi-1.14.5-5.el9.x86_64                     15/57 
  Verifying        : python3-ply-3.11-14.el9.0.1.noarch                   16/57 
  Verifying        : python3-pycparser-2.20-6.el9.noarch                  17/57 
  Verifying        : perl-TimeDate-1:2.33-6.el9.noarch                    18/57 
  Verifying        : python3-lxml-4.6.5-3.el9.x86_64                      19/57 
  Verifying        : python3-pycurl-7.43.0.6-8.el9.x86_64                 20/57 
  Verifying        : net-snmp-libs-1:5.9.1-13.el9.x86_64                  21/57 
  Verifying        : rubygems-3.2.33-161.el9.noarch                       22/57 
  Verifying        : rubygem-rexml-3.2.5-161.el9.noarch                   23/57 
  Verifying        : rubygem-rdoc-6.3.3-161.el9.noarch                    24/57 
  Verifying        : rubygem-bundler-2.2.33-161.el9.noarch                25/57 
  Verifying        : ruby-default-gems-3.0.4-161.el9.noarch               26/57 
  Verifying        : libxslt-1.1.34-9.el9.x86_64                          27/57 
  Verifying        : nss-tools-3.90.0-7.el9_4.x86_64                      28/57 
  Verifying        : corosynclib-3.1.8-1.el9.x86_64                       29/57 
  Verifying        : rubygem-psych-3.3.2-161.el9.x86_64                   30/57 
  Verifying        : rubygem-json-2.5.1-161.el9.x86_64                    31/57 
  Verifying        : rubygem-io-console-0.5.7-161.el9.x86_64              32/57 
  Verifying        : rubygem-bigdecimal-3.0.0-161.el9.x86_64              33/57 
  Verifying        : ruby-libs-3.0.4-161.el9.x86_64                       34/57 
  Verifying        : ruby-3.0.4-161.el9.x86_64                            35/57 
  Verifying        : libqb-2.0.8-1.el9.x86_64                             36/57 
  Verifying        : resource-agents-4.10.0-52.el9.x86_64                 37/57 
  Verifying        : corosync-3.1.8-1.el9.x86_64                          38/57 
  Verifying        : pacemaker-schemas-2.1.7-5.el9_4.noarch               39/57 
  Verifying        : pacemaker-libs-2.1.7-5.el9_4.x86_64                  40/57 
  Verifying        : pacemaker-cluster-libs-2.1.7-5.el9_4.x86_64          41/57 
  Verifying        : pacemaker-cli-2.1.7-5.el9_4.x86_64                   42/57 
  Verifying        : pacemaker-2.1.7-5.el9_4.x86_64                       43/57 
  Verifying        : libnozzle1-1.28-1.el9.x86_64                         44/57 
  Verifying        : libknet1-plugins-all-1.28-1.el9.x86_64               45/57 
  Verifying        : libknet1-crypto-plugins-all-1.28-1.el9.x86_64        46/57 
  Verifying        : libknet1-crypto-openssl-plugin-1.28-1.el9.x86_64     47/57 
  Verifying        : libknet1-crypto-nss-plugin-1.28-1.el9.x86_64         48/57 
  Verifying        : libknet1-compress-zstd-plugin-1.28-1.el9.x86_64      49/57 
  Verifying        : libknet1-compress-zlib-plugin-1.28-1.el9.x86_64      50/57 
  Verifying        : libknet1-compress-plugins-all-1.28-1.el9.x86_64      51/57 
  Verifying        : libknet1-compress-lzo2-plugin-1.28-1.el9.x86_64      52/57 
  Verifying        : libknet1-compress-lzma-plugin-1.28-1.el9.x86_64      53/57 
  Verifying        : libknet1-compress-lz4-plugin-1.28-1.el9.x86_64       54/57 
  Verifying        : libknet1-compress-bzip2-plugin-1.28-1.el9.x86_64     55/57 
  Verifying        : libknet1-1.28-1.el9.x86_64                           56/57 
  Verifying        : pcs-0.11.7-2.el9_4.x86_64                            57/57 

Installed:
  bzip2-1.0.8-8.el9.x86_64                                                      
  corosync-3.1.8-1.el9.x86_64                                                   
  corosynclib-3.1.8-1.el9.x86_64                                                
  device-mapper-event-9:1.02.197-2.el9.x86_64                                   
  device-mapper-event-libs-9:1.02.197-2.el9.x86_64                              
  device-mapper-persistent-data-1.0.9-3.el9_4.x86_64                            
  libaio-0.3.111-13.el9.x86_64                                                  
  libknet1-1.28-1.el9.x86_64                                                    
  libknet1-compress-bzip2-plugin-1.28-1.el9.x86_64                              
  libknet1-compress-lz4-plugin-1.28-1.el9.x86_64                                
  libknet1-compress-lzma-plugin-1.28-1.el9.x86_64                               
  libknet1-compress-lzo2-plugin-1.28-1.el9.x86_64                               
  libknet1-compress-plugins-all-1.28-1.el9.x86_64                               
  libknet1-compress-zlib-plugin-1.28-1.el9.x86_64                               
  libknet1-compress-zstd-plugin-1.28-1.el9.x86_64                               
  libknet1-crypto-nss-plugin-1.28-1.el9.x86_64                                  
  libknet1-crypto-openssl-plugin-1.28-1.el9.x86_64                              
  libknet1-crypto-plugins-all-1.28-1.el9.x86_64                                 
  libknet1-plugins-all-1.28-1.el9.x86_64                                        
  libnozzle1-1.28-1.el9.x86_64                                                  
  libpkgconf-1.7.3-10.el9.x86_64                                                
  libqb-2.0.8-1.el9.x86_64                                                      
  libxslt-1.1.34-9.el9.x86_64                                                   
  lvm2-9:2.03.23-2.el9.x86_64                                                   
  lvm2-libs-9:2.03.23-2.el9.x86_64                                              
  net-snmp-libs-1:5.9.1-13.el9.x86_64                                           
  net-tools-2.0-0.62.20160912git.el9.x86_64                                     
  nss-tools-3.90.0-7.el9_4.x86_64                                               
  pacemaker-2.1.7-5.el9_4.x86_64                                                
  pacemaker-cli-2.1.7-5.el9_4.x86_64                                            
  pacemaker-cluster-libs-2.1.7-5.el9_4.x86_64                                   
  pacemaker-libs-2.1.7-5.el9_4.x86_64                                           
  pacemaker-schemas-2.1.7-5.el9_4.noarch                                        
  pcs-0.11.7-2.el9_4.x86_64                                                     
  perl-TimeDate-1:2.33-6.el9.noarch                                             
  pkgconf-1.7.3-10.el9.x86_64                                                   
  pkgconf-m4-1.7.3-10.el9.noarch                                                
  pkgconf-pkg-config-1.7.3-10.el9.x86_64                                        
  python3-cffi-1.14.5-5.el9.x86_64                                              
  python3-cryptography-36.0.1-4.el9.x86_64                                      
  python3-lxml-4.6.5-3.el9.x86_64                                               
  python3-ply-3.11-14.el9.0.1.noarch                                            
  python3-pycparser-2.20-6.el9.noarch                                           
  python3-pycurl-7.43.0.6-8.el9.x86_64                                          
  python3-pyparsing-2.4.7-9.el9.noarch                                          
  resource-agents-4.10.0-52.el9.x86_64                                          
  ruby-3.0.4-161.el9.x86_64                                                     
  ruby-default-gems-3.0.4-161.el9.noarch                                        
  ruby-libs-3.0.4-161.el9.x86_64                                                
  rubygem-bigdecimal-3.0.0-161.el9.x86_64                                       
  rubygem-bundler-2.2.33-161.el9.noarch                                         
  rubygem-io-console-0.5.7-161.el9.x86_64                                       
  rubygem-json-2.5.1-161.el9.x86_64                                             
  rubygem-psych-3.3.2-161.el9.x86_64                                            
  rubygem-rdoc-6.3.3-161.el9.noarch                                             
  rubygem-rexml-3.2.5-161.el9.noarch                                            
  rubygems-3.2.33-161.el9.noarch                                                

Complete!
```
</div>
</div>

#### Start pcsd service
```bash
sudo systemctl enable pcsd --now
```
<div class="codehider">
toggle console output:
<div class="hidecode" markdown="1">
```collapse
Created symlink /etc/systemd/system/multi-user.target.wants/pcsd.service → /usr/lib/systemd/system/pcsd.service.
```
</div>
</div>


#### Set password for hacluster user
```bash
echo "ChangeMe" | sudo passwd --stdin hacluster
```
<div class="codehider">
toggle console output:
<div class="hidecode" markdown="1">
```collapse
Changing password for user hacluster.
passwd: all authentication tokens updated successfully.
```
</div>
</div>


#### Open firewall ports
In production, you should probably be creating a cluster network zone and limit the access to the cluster members.
```bash
sudo firewall-cmd --add-service=high-availability --permanent
sudo firewall-cmd --reload
```
<div class="codehider">
toggle console output:
<div class="hidecode" markdown="1">
```collapse
success
success
```
</div>
</div>

### First node only

#### Authenticate pcs with the cluster
```bash
sudo pcs host auth node1 node2 node3 -u hacluster -p "ChangeMe"
```
<div class="codehider">
toggle console output:
<div class="hidecode" markdown="1">
```collapse
node1: Authorized
node3: Authorized
node2: Authorized
```
</div>
</div>

#### Create a new cluster
In this example, I'm not using IP addresses, but in production, you should use static IP addresses.
`democluster` is the name of the cluster, and `node1`, `node2`, and `node3` are the names of the nodes.
```bash
sudo pcs cluster setup democluster node1 node2 node3
```
<div class="codehider">
toggle console output:
<div class="hidecode" markdown="1">
```collapse
No addresses specified for host 'node1', using 'node1'
No addresses specified for host 'node2', using 'node2'
No addresses specified for host 'node3', using 'node3'
Destroying cluster on hosts: 'node1', 'node2', 'node3'...
node3: Successfully destroyed cluster
node1: Successfully destroyed cluster
node2: Successfully destroyed cluster
Requesting remove 'pcsd settings' from 'node1', 'node2', 'node3'
node1: successful removal of the file 'pcsd settings'
node2: successful removal of the file 'pcsd settings'
node3: successful removal of the file 'pcsd settings'
Sending 'corosync authkey', 'pacemaker authkey' to 'node1', 'node2', 'node3'
node1: successful distribution of the file 'corosync authkey'
node1: successful distribution of the file 'pacemaker authkey'
node2: successful distribution of the file 'corosync authkey'
node2: successful distribution of the file 'pacemaker authkey'
node3: successful distribution of the file 'corosync authkey'
node3: successful distribution of the file 'pacemaker authkey'
Sending 'corosync.conf' to 'node1', 'node2', 'node3'
node1: successful distribution of the file 'corosync.conf'
node2: successful distribution of the file 'corosync.conf'
node3: successful distribution of the file 'corosync.conf'
Cluster has been successfully set up.
```
</div>
</div>

#### Start the cluster on the first node
```bash
sudo pcs cluster start
```
<div class="codehider">
toggle console output:
<div class="hidecode" markdown="1">
```collapse
Starting Cluster...
```
</div>
</div>

#### Verify the cluster status
```bash
sudo pcs status
```
<div class="codehider">
toggle console output:
<div class="hidecode" markdown="1">
```collapse
Cluster name: democluster

WARNINGS:
No stonith devices and stonith-enabled is not false

Cluster Summary:
  * Stack: unknown (Pacemaker is running)
  * Current DC: NONE
  * Last updated: Sun May 12 05:21:38 2024 on node1
  * Last change:  Sun May 12 05:21:21 2024 by hacluster via hacluster on node1
  * 3 nodes configured
  * 0 resource instances configured

Node List:
  * Node node1: UNCLEAN (offline)
  * Node node2: UNCLEAN (offline)
  * Node node3: UNCLEAN (offline)

Full List of Resources:
  * No resources

Daemon Status:
  corosync: active/disabled
  pacemaker: active/disabled
  pcsd: active/enabled
```
</div>
</div>
Notice in the output at first some of the nodes may show as UNCLEAN (offline).
After a few seconds, the first node should show as ONLINE.


#### Start the cluster on the other nodes
```bash
sudo pcs cluster start --all
```
<div class="codehider">
toggle console output:
<div class="hidecode" markdown="1">
```collapse
node3: Starting Cluster...
node1: Starting Cluster...
node2: Starting Cluster...
```
</div>
</div>

#### Verify the cluster status
```bash
sudo pcs status
```
<div class="codehider">
toggle console output:
<div class="hidecode" markdown="1">
```collapse
Cluster name: democluster

WARNINGS:
No stonith devices and stonith-enabled is not false

Cluster Summary:
  * Stack: corosync (Pacemaker is running)
  * Current DC: node2 (version 2.1.7-5.el9_4-0f7f88312) - partition with quorum
  * Last updated: Sun May 12 05:35:07 2024 on node1
  * Last change:  Sun May 12 05:34:34 2024 by hacluster via hacluster on node2
  * 3 nodes configured
  * 0 resource instances configured

Node List:
  * Online: [ node1 node2 node3 ]

Full List of Resources:
  * No resources

Daemon Status:
  corosync: active/disabled
  pacemaker: active/disabled
  pcsd: active/enabled
```
</div>
</div>
After a moment all nodes should show as ONLINE and clean.

#### Disable stonith for now
For now, we are disabling the stonith device. I can go into what stonith is and why it's important in a later post.
```bash
sudo pcs property set stonith-enabled=false
```

## Conclusion
Now that the cluster is up and running you can start adding resources to the cluster.
In the next post, I'll go over adding resources to the cluster.
