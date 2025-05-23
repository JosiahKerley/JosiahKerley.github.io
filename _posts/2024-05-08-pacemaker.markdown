---
layout: post
title:  "Deploying a Simple High-Availability Nginx Service with Pacemaker"
date:   2024-05-08 11:23:00
series: "Pacemaker"
categories:
  - pacemaker
  - guides
toc: true
---
!["Open books strewn across a green grass field as seen from above."](/assets/img/other/books-in-a-field.png)

In this guide, I will show you how to deploy nginx and it's default page as a high-availability service using Pacemaker.

## Pre-requisites
This guide assumes you have a basic understanding of Pacemaker and have a cluster already set up.
If you don't have a cluster set up, you can follow my guide here in [this post](/pacemaker/guides/2024/05/07/pacemaker.html#deploy-pacemaker-cluster)

## Deploying Nginx

### All nodes

#### Install Nginx
```bash
sudo dnf install -y nginx
```
<div class="codehider">
toggle console output:
<div class="hidecode" markdown="1">
```collapse
Dependencies resolved.
================================================================================
 Package               Arch       Version                   Repository     Size
================================================================================
Installing:
 nginx                 x86_64     1:1.20.1-14.el9_2.1       appstream      36 k
Installing dependencies:
 nginx-core            x86_64     1:1.20.1-14.el9_2.1       appstream     565 k
 nginx-filesystem      noarch     1:1.20.1-14.el9_2.1       appstream     8.5 k
 rocky-logos-httpd     noarch     90.15-2.el9               appstream      24 k

Transaction Summary
================================================================================
Install  4 Packages

Total download size: 634 k
Installed size: 1.8 M
Downloading Packages:
(1/4): nginx-filesystem-1.20.1-14.el9_2.1.noarc  15 kB/s | 8.5 kB     00:00    
(2/4): rocky-logos-httpd-90.15-2.el9.noarch.rpm  43 kB/s |  24 kB     00:00    
(3/4): nginx-1.20.1-14.el9_2.1.x86_64.rpm        61 kB/s |  36 kB     00:00    
(4/4): nginx-core-1.20.1-14.el9_2.1.x86_64.rpm  686 kB/s | 565 kB     00:00    
--------------------------------------------------------------------------------
Total                                           333 kB/s | 634 kB     00:01     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Running scriptlet: nginx-filesystem-1:1.20.1-14.el9_2.1.noarch            1/4 
  Installing       : nginx-filesystem-1:1.20.1-14.el9_2.1.noarch            1/4 
  Installing       : nginx-core-1:1.20.1-14.el9_2.1.x86_64                  2/4 
  Installing       : rocky-logos-httpd-90.15-2.el9.noarch                   3/4 
  Installing       : nginx-1:1.20.1-14.el9_2.1.x86_64                       4/4 
  Running scriptlet: nginx-1:1.20.1-14.el9_2.1.x86_64                       4/4 
  Verifying        : rocky-logos-httpd-90.15-2.el9.noarch                   1/4 
  Verifying        : nginx-filesystem-1:1.20.1-14.el9_2.1.noarch            2/4 
  Verifying        : nginx-1:1.20.1-14.el9_2.1.x86_64                       3/4 
  Verifying        : nginx-core-1:1.20.1-14.el9_2.1.x86_64                  4/4 

Installed:
  nginx-1:1.20.1-14.el9_2.1.x86_64                                              
  nginx-core-1:1.20.1-14.el9_2.1.x86_64                                         
  nginx-filesystem-1:1.20.1-14.el9_2.1.noarch                                   
  rocky-logos-httpd-90.15-2.el9.noarch                                          

Complete!
```
</div>
</div>

#### Ensure that systemd does not manager nginx
```bash
sudo systemctl disable nginx --now
```

#### Allow Nginx through the firewall
```bash
sudo firewall-cmd --add-service=http --permanent
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

#### Create a clone resource of nginx in pacemaker
As I said before, resources in pacemaker can be rather arbitrary. In this case, we are using a systemd resource
to manage the nginx service.  The cluster will handle starting and stopping the service as needed based on
the rules set for the service and its related resources.

```bash
sudo pcs resource create nginx systemd:nginx op monitor interval=10s clone
```

Let's break down the command:
- `sudo pcs resource create nginx` - This creates a new resource named `nginx`
- `systemd:nginx` - This tells pacemaker that the resource is a systemd unit named `nginx`
- `op monitor interval=10s` - This tells pacemaker to check the status of the resource every 10 seconds
- `clone` - This tells pacemaker to create a clone resource.  This means that the resource will be started on all nodes in the cluster

Unlike a resource like perhaps a vip, a clone resource will be started on all nodes in the cluster.
This is useful for services that need to be running on all nodes in the cluster.  Nginx will come up on all nodes
but users will have to connect to each server individually.
In [this post](/pacemaker/guides/2024/05/09/pacemaker.html) I will show you how to set up
a virtual IP address that will float between the nodes in the cluster.

### Verify nginx is running on all nodes
You can check the status of the service on all nodes by running the following command:
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
  * Last updated: Sun May 12 05:40:02 2024 on node1
  * Last change:  Sun May 12 05:39:35 2024 by root via root on node1
  * 3 nodes configured
  * 3 resource instances configured

Node List:
  * Online: [ node1 node2 node3 ]

Full List of Resources:
  * Clone Set: nginx-clone [nginx]:
    * Started: [ node1 node2 node3 ]

Daemon Status:
  corosync: active/disabled
  pacemaker: active/disabled
  pcsd: active/enabled
```
</div>
</div>

but to the real check is to see if the default nginx page is available on all nodes.
```bash
curl node1
curl node2
curl node3
```
<div class="codehider">
toggle console output:
<div class="hidecode" markdown="1">
```collapse
</html>
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/
      
      html {
        height: 100%;
        width: 100%;
      }  
        body {
  background: rgb(20,72,50);
  background: -moz-linear-gradient(180deg, rgba(23,43,70,1) 30%, rgba(0,0,0,1) 90%)  ;
  background: -webkit-linear-gradient(180deg, rgba(23,43,70,1) 30%, rgba(0,0,0,1) 90%) ;
  background: linear-gradient(180deg, rgba(23,43,70,1) 30%, rgba(0,0,0,1) 90%);
  background-repeat: no-repeat;
  background-attachment: fixed;
  filter: progid:DXImageTransform.Microsoft.gradient(startColorstr="#3c6eb4",endColorstr="#3c95b4",GradientType=1); 
        color: white;
        font-size: 0.9em;
        font-weight: 400;
        font-family: 'Montserrat', sans-serif;
        margin: 0;
        padding: 10em 6em 10em 6em;
        box-sizing: border-box;      
        
      }

   
  h1 {
    text-align: center;
    margin: 0;
    padding: 0.6em 2em 0.4em;
    color: #fff;
    font-weight: bold;
    font-family: 'Montserrat', sans-serif;
    font-size: 2em;
  }
  h1 strong {
    font-weight: bolder;
    font-family: 'Montserrat', sans-serif;
  }
  h2 {
    font-size: 1.5em;
    font-weight:bold;
  }
  
  .title {
    border: 1px solid black;
    font-weight: bold;
    position: relative;
    float: right;
    width: 150px;
    text-align: center;
    padding: 10px 0 10px 0;
    margin-top: 0;
  }
  
  .description {
    padding: 45px 10px 5px 10px;
    clear: right;
    padding: 15px;
  }
  
  .section {
    padding-left: 3%;
   margin-bottom: 10px;
  }
  
  img {
  
    padding: 2px;
    margin: 2px;
  }
  a:hover img {
    padding: 2px;
    margin: 2px;
  }

  :link {
    color: rgb(199, 252, 77);
    text-shadow:
  }
  :visited {
    color: rgb(122, 206, 255);
  }
  a:hover {
    color: rgb(16, 44, 122);
  }
  .row {
    width: 100%;
    padding: 0 10px 0 10px;
  }
  
  footer {
    padding-top: 6em;
    margin-bottom: 6em;
    text-align: center;
    font-size: xx-small;
    overflow:hidden;
    clear: both;
  }
 
  .summary {
    font-size: 140%;
    text-align: center;
  }

  #rocky-poweredby img {
    margin-left: -10px;
  }

  #logos img {
    vertical-align: top;
  }
  
  /* Desktop  View Options */
 
  @media (min-width: 768px)  {
  
    body {
      padding: 10em 20% !important;
    }
    
    .col-md-1, .col-md-2, .col-md-3, .col-md-4, .col-md-5, .col-md-6,
    .col-md-7, .col-md-8, .col-md-9, .col-md-10, .col-md-11, .col-md-12 {
      float: left;
    }
  
    .col-md-1 {
      width: 8.33%;
    }
    .col-md-2 {
      width: 16.66%;
    }
    .col-md-3 {
      width: 25%;
    }
    .col-md-4 {
      width: 33%;
    }
    .col-md-5 {
      width: 41.66%;
    }
    .col-md-6 {
      border-left:3px ;
      width: 50%;
      

    }
    .col-md-7 {
      width: 58.33%;
    }
    .col-md-8 {
      width: 66.66%;
    }
    .col-md-9 {
      width: 74.99%;
    }
    .col-md-10 {
      width: 83.33%;
    }
    .col-md-11 {
      width: 91.66%;
    }
    .col-md-12 {
      width: 100%;
    }
  }
  
  /* Mobile View Options */
  @media (max-width: 767px) {
    .col-sm-1, .col-sm-2, .col-sm-3, .col-sm-4, .col-sm-5, .col-sm-6,
    .col-sm-7, .col-sm-8, .col-sm-9, .col-sm-10, .col-sm-11, .col-sm-12 {
      float: left;
    }
  
    .col-sm-1 {
      width: 8.33%;
    }
    .col-sm-2 {
      width: 16.66%;
    }
    .col-sm-3 {
      width: 25%;
    }
    .col-sm-4 {
      width: 33%;
    }
    .col-sm-5 {
      width: 41.66%;
    }
    .col-sm-6 {
      width: 50%;
    }
    .col-sm-7 {
      width: 58.33%;
    }
    .col-sm-8 {
      width: 66.66%;
    }
    .col-sm-9 {
      width: 74.99%;
    }
    .col-sm-10 {
      width: 83.33%;
    }
    .col-sm-11 {
      width: 91.66%;
    }
    .col-sm-12 {
      width: 100%;
    }
    h1 {
      padding: 0 !important;
    }
  }
        
  
  </style>
  </head>
  <body>
    <h1>HTTP Server <strong>Test Page</strong></h1>

    <div class='row'>
    
      <div class='col-sm-12 col-md-6 col-md-6 '></div>
          <p class="summary">This page is used to test the proper operation of
            an HTTP server after it has been installed on a Rocky Linux system.
            If you can read this page, it means that the software is working
            correctly.</p>
      </div>
      
      <div class='col-sm-12 col-md-6 col-md-6 col-md-offset-12'>
     
       
        <div class='section'>
          <h2>Just visiting?</h2>

          <p>This website you are visiting is either experiencing problems or
          could be going through maintenance.</p>

          <p>If you would like the let the administrators of this website know
          that you've seen this page instead of the page you've expected, you
          should send them an email. In general, mail sent to the name
          "webmaster" and directed to the website's domain should reach the
          appropriate person.</p>

          <p>The most common email address to send to is:
          <strong>"webmaster@example.com"</strong></p>
    
          <h2>Note:</h2>
          <p>The Rocky Linux distribution is a stable and reproduceable platform
          based on the sources of Red Hat Enterprise Linux (RHEL). With this in
          mind, please understand that:

        <ul>
          <li>Neither the <strong>Rocky Linux Project</strong> nor the
          <strong>Rocky Enterprise Software Foundation</strong> have anything to
          do with this website or its content.</li>
          <li>The Rocky Linux Project nor the <strong>RESF</strong> have
          "hacked" this webserver: This test page is included with the
          distribution.</li>
        </ul>
        <p>For more information about Rocky Linux, please visit the
          <a href="https://rockylinux.org/"><strong>Rocky Linux
          website</strong></a>.
        </p>
        </div>
      </div>
      <div class='col-sm-12 col-md-6 col-md-6 col-md-offset-12'>
        <div class='section'>
         
          <h2>I am the admin, what do I do?</h2>

        <p>You may now add content to the webroot directory for your
        software.</p>

        <p><strong>For systems using the
        <a href="https://httpd.apache.org/">Apache Webserver</strong></a>:
        You can add content to the directory <code>/var/www/html/</code>.
        Until you do so, people visiting your website will see this page. If
        you would like this page to not be shown, follow the instructions in:
        <code>/etc/httpd/conf.d/welcome.conf</code>.</p>

        <p><strong>For systems using
        <a href="https://nginx.org">Nginx</strong></a>:
        You can add your content in a location of your
        choice and edit the <code>root</code> configuration directive
        in <code>/etc/nginx/nginx.conf</code>.</p>
        
        <div id="logos">
          <a href="https://rockylinux.org/" id="rocky-poweredby"><img src="icons/poweredby.png" alt="[ Powered by Rocky Linux ]" /></a> <!-- Rocky -->
          <img src="poweredby.png" /> <!-- webserver -->
        </div>       
      </div>
      </div>
      
      <footer class="col-sm-12">
      <a href="https://apache.org">Apache&trade;</a> is a registered trademark of <a href="https://apache.org">the Apache Software Foundation</a> in the United States and/or other countries.<br />
      <a href="https://nginx.org">NGINX&trade;</a> is a registered trademark of <a href="https://">F5 Networks, Inc.</a>.
      </footer>
      
  </body>
</html>

```
</div>
</div>

If all is well, each curl command should have returned some HTML from the default nginx page.
