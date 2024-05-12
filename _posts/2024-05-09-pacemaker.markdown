---
layout: post
title:  "Deploying a VIP on a Pacemaker Cluster"
date:   2024-05-09 09:43:00
categories:
  - pacemaker
  - guides
toc: true
---

!["Three men throwing a ball back and forth in green grass field as seen from above."](/assets/img/other/three-men-playing-ball.png)
 

In this guide, I will show you how to configure a virtual ip address (VIP) on a pacemaker cluster.
This VIP will float between the nodes in the cluster and allow users to connect to a single IP address to access the service.

## Pre-requisites
This guide assumes you have a basic understanding of Pacemaker and have a cluster already set up.
If you don't have a cluster set up, you can follow my guide here in [this post](/pacemaker/guides/2024/05/07/pacemaker.html#deploy-pacemaker-cluster)

You will also need a static ip address that is not in use on your network.

## Deploying a VIP
In this example, I will be using the vip of `192.168.30.10/24`.
You should replace this with an ip address thatis not in use on your network.

### Checking to make sure the VIP is not in use
```bash
ping 192.168.30.10
```
<div class="codehider">
toggle console output:
<div class="hidecode" markdown="1">
```collapse
PING 192.168.30.10 (192.168.30.10) 56(84) bytes of data.
From 192.168.30.186 icmp_seq=1 Destination Host Unreachable
From 192.168.30.186 icmp_seq=2 Destination Host Unreachable
From 192.168.30.186 icmp_seq=3 Destination Host Unreachable
^C
--- 192.168.30.10 ping statistics ---
4 packets transmitted, 0 received, +3 errors, 100% packet loss, time 3055ms
```
</div>
</div>
If nothing returns, then the ip address may not be in use.  I am assuming you are deploying this in a lab environment and not in production.
Obviously, in production, you should be getting an ip address from IPAM or your network team.

### First node only
```bash
sudo pcs resource create vip ocf:heartbeat:IPaddr2 ip=192.168.30.10 cidr_netmask=24 op monitor interval=10s
```
Let's break down the command:
- `sudo pcs resource create vip` - This creates a new resource named `vip`
- `ocf:heartbeat:IPaddr2` - This tells pacemaker that the resource is an IP address resource
- `ip=192.168.30.10` - This tells pacemaker to use the ip address
- `cidr_netmask=24` - This tells pacemaker to use a /24 netmask
- `op monitor interval=10s` - This tells pacemaker to check the status of the resource every 10 seconds

### Verify the VIP is running a node
```bash
sudo pcs status
```
<div class="codehider">
toggle console output:
<div class="hidecode" markdown="1">
```collapse
Cluster name: democluster
Cluster Summary:
  * Stack: corosync (Pacemaker is running)
  * Current DC: node2 (version 2.1.7-5.el9_4-0f7f88312) - partition with quorum
  * Last updated: Sun May 12 05:42:43 2024 on node1
  * Last change:  Sun May 12 05:42:32 2024 by root via root on node1
  * 3 nodes configured
  * 4 resource instances configured

Node List:
  * Online: [ node1 node2 node3 ]

Full List of Resources:
  * Clone Set: nginx-clone [nginx]:
    * Started: [ node1 node2 node3 ]
  * vip (ocf:heartbeat:IPaddr2):         Started node1

Daemon Status:
  corosync: active/disabled
  pacemaker: active/disabled
  pcsd: active/enabled
```
</div>
</div>

### Testing the VIP with ping
```bash
ping 192.168.30.10
```
<div class="codehider">
toggle console output:
<div class="hidecode" markdown="1">
```collapse
PING 192.168.30.10 (192.168.30.10) 56(84) bytes of data.
64 bytes from 192.168.30.10: icmp_seq=1 ttl=64 time=0.082 ms
64 bytes from 192.168.30.10: icmp_seq=2 ttl=64 time=0.067 ms
64 bytes from 192.168.30.10: icmp_seq=3 ttl=64 time=0.069 ms
^C
--- 192.168.30.10 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2074ms
rtt min/avg/max/mdev = 0.067/0.072/0.082/0.006 ms
```
</div>
</div>
Now that the VIP resource is up, you should be able to ping the ip address from any node in the cluster.
