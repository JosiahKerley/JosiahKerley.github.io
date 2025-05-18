---
layout: post
title:  "Configuring the Bacula Storage Daemon on NixOS" 
date:   2025-05-10 00:00:00
toc: true
series: "Immutable Enterprise Backup"
categories:
  - bacula-on-nixos
  - bacula
  - guide
---

!["The logos of the bacula and nixos project"](/assets/img/posts/2025-04-17-immutable-bacula/splash.png)

## Considerations for the Storage Daemon
The storage daemon acts as the storage service for Bacula, managing the storage of backup data. It is responsible for writing and reading data to and from the storage devices.
A device is a physical or virtual storage medium, such as a tape drive, disk, or cloud storage. Bacula supports various device types, including file, tape, and cloud devices. The configuration of the storage daemon is crucial for ensuring efficient and reliable backup operations.

In this post, we will configure the storage daemon to use a single file device, which is a common choice for testing and development purposes.
The file device allows Bacula to write backup data to a file on the local filesystem, making it easy to manage and access the backup data.

## Laying out the Configuration
Unlike with the previous post where we designed the director's configuration code to use a directory structure and multiple files, we will instead create a single `sd.nix` file for the storage daemon configuration.
This is because the storage daemon's configuration is typically less complex than the director's configuration, and a single file is sufficient for our needs.
We will create a new file called `sd.nix` in the `bacula` directory, which will contain the configuration for the storage daemon.

```nix
{ config, pkgs, lib, ... }:
{
  services = {
    bacula-sd = {
      enable = true;
      name = "bacula-sd";
      tls.enable = false;

      director."bacula-dir" = {
        tls.enable = false;
        password = "ChangeMe!";
      };

      device."File1" = {
        archiveDevice = "/var/lib/bacula/files/File1";
        mediaType = "File";
      };

    };
  };

  systemd.services.bacula-sd = {
    serviceConfig = {
      ExecStartPre = [
        "${pkgs.bash}/bin/bash -c '[[ -d /var/lib/bacula/files/File1 ]] || ${pkgs.coreutils}/bin/mkdir -p /var/lib/bacula/files/File1'"
        "-${pkgs.coreutils}/bin/chown -R bacula:bacula /var/lib/bacula"
      ];
    };
  };

}
```

This will use the device `File1` to store the backup data in the directory `/var/lib/bacula/files/File1`.
The `ExecStartPre` option is used to create the directory and set the ownership of the `/var/lib/bacula` directory to the `bacula` user and group before starting the storage daemon.
Once the configuration is in place, run `sudo nixos-rebuild switch` to apply the changes.

Now we can use bconsole to get the status of the storage daemon and see if it is running correctly.
```shell
[josiah@bacula:~]$ bconsole -c /etc/bacula/bconsole.conf 
Connecting to Director localhost:9101
1000 OK: 10002 bacula-dir Version: 15.0.2 (21 March 2024)
Enter a period to cancel a command.
*status storage
Automatically selected Storage: bacula-sd_file1
Connecting to Storage daemon bacula-sd_file1 at bacula.lab:9103

bacula-sd Version: 15.0.2 (21 March 2024) x86_64-pc-linux-gnu unknown_distro 
Daemon started 10-May-25 10:31. Jobs: run=0, running=0 max=20.
 Ulimits: nofile=1024 memlock=8388608 status=nofile
 Heap: heap=667,648 smbytes=205,706 max_bytes=391,286 bufs=133 max_bufs=134
 Sizes: boffset_t=8 size_t=8 int32_t=4 int64_t=8 mode=0,0 newbsr=0
 Crypto: fips=N/A crypto=OpenSSL 3.3.2 3 Sep 2024
 Res: ndevices=1 nautochgr=0
 Caps: APPEND_ONLY, IMMUTABLE

Running Jobs:
Director connected using TLS at: 10-May-25 11:23
No Jobs running.
====

Jobs waiting to reserve a drive:
====

Terminated Jobs:
====

Device status:

Device File: "File1" (/var/lib/bacula/files/File1) is not open.
   Available Space=191.3 GB
==
====

Used Volume status:
====

====

*
```

So far, so good! The storage daemon is running and is able to connect to the director.
Let's go ahead and create a volume to use for our backups.

## Creating a Volume
On File based devices, Bacula uses a volume to store the backup data.
Volumes are files that are created on the storage device and are used to store the backup data (you can also think of them as tarballs).

To create a volume, we will use the `bconsole` command line tool.
```shell
*label
Automatically selected Storage: bacula-sd_file1
Connecting to Storage daemon bacula-sd_file1 at bacula.lab:9103 ...
3998 Device ""File1" (/var/lib/bacula/files/File1)" is not an autochanger.
Enter autochanger drive[0]: 
Enter new Volume name: Vol-0001
Enter slot (0 or Enter for none): 
Automatically selected Pool: FilePool
Connecting to Storage daemon bacula-sd_file1 at bacula.lab:9103 ...
Sending label command for Volume "Vol-0001" Slot 0 ...
3000 OK label. VolBytes=237 VolABytes=0 VolType=1 UseProtect=0 VolEncrypted=0 Volume="Vol-0001" Device="File1" (/var/lib/bacula/files/File1)
Catalog record for Volume "Vol-0001", Slot 0  successfully created.
Requesting to mount File1 ...
3002 Device ""File1" (/var/lib/bacula/files/File1)" is mounted.
*
```

Now if we look in the directory `/var/lib/bacula/files/File1`, we should see a file called `Vol-0001` which is the volume we just created.
```shell
[josiah@bacula:~]$ sudo ls -laht /var/lib/bacula/files/File1/Vol-0001
-rw-r----- 1 bacula bacula 237 May 10 11:26 /var/lib/bacula/files/File1/Vol-0001
```

## Conclusion
In this post, we have configured the Bacula storage daemon to use a file device for storing backup data.
We have also created a volume to use for our backups.
In the next post, we will configure the file daemon to use the storage daemon and create a job to back up some data.
