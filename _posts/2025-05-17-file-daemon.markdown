---
layout: post
title:  "Configuring a Bacula client and it's File Daemon" 
date:   2025-05-17 00:00:00
toc: true
series: "Immutable Enterprise Backup"
categories:
  - bacula-on-nixos
  - bacula
  - guide
---

!["The logos of the bacula and nixos project"](/assets/img/posts/2025-04-17-immutable-bacula/splash.png)

## Backing up ourselves
In the previous post, we set up a Storage Daemon. In this post, we will configure a Bacula client and its File Daemon.
The File Daemon is responsible for backing up files on the client machine and communicating with the Director.

## Laying out the Configuration
As with the storage daemon's `sd.nix`, we will create a file in the `bacula` module directory called `fd.nix`.
This file will contain the configuration for the File Daemon.

Import it in the `configuration.nix` file of the `bacula` module:
```nix
...
  imports =
    [ # Include the results of the hardware scan.
      ./hardware-configuration.nix
      ./bacula/dir/main.nix
      ./bacula/sd.nix
      ./bacula/fd.nix ## Add this line
    ];
...
```

## Configuring the File Daemon
Like the storage daemon, the file daemon config is pretty straight forward.
```nix
{ config, pkgs, lib, ... }:
{
  services = {
    bacula-fd = {
      enable = true;
      director."bacula-dir" = { ## The name of the director from bacula/dir/services.nix
        tls.enable = false;
        password = "ChangeMe!"; ## The password from the client resource in bacula/dir/services.nix
      };

      extraClientConfig = ''
        Plugin Directory = ${pkgs.bacula}/lib
        Maximum Concurrent Jobs = 10
      '';
    };
  };
}
```

Run `sudo nixos-rebuild switch` to apply the changes.
To verify, let's check the status of the File Daemon:
```shell
[josiah@bacula:~]$ bconsole -c /etc/bacula/bconsole.conf 
Connecting to Director localhost:9101
1000 OK: 10002 bacula-dir Version: 15.0.2 (21 March 2024)
Enter a period to cancel a command.
*status client
Automatically selected Client: bacula-fd
Connecting to Client bacula-fd at bacula.lab:9102

bacula-fd Version: 15.0.2 (21 March 2024)  x86_64-pc-linux-gnu unknown_distro 
Daemon started 17-May-25 12:11. Jobs: run=0 running=0 max=10.
 Heap: heap=557,056 smbytes=198,088 max_bytes=198,089 bufs=99 max_bufs=99
 Sizes: boffset_t=8 size_t=8 debug=0 trace=0 mode=0,0 bwlimit=0kB/s
 Crypto: fips=N/A crypto=OpenSSL 3.3.2 3 Sep 2024
 Plugin: bpipe(2) 

Running Jobs:
Director connected using TLS at: 17-May-25 10:11
No Jobs running.
====

Terminated Jobs:
====
*
```
Looks good.  Name resolution is working and since the client is running on the same machine as the Director, we shouldn't have any firewall issues.

## Running our first backup
Now that we have a File Daemon, we can run our first backup.

To manually start the backup, we can use the `bconsole` command:
```shell
[josiah@bacula:~]$ bconsole -c /etc/bacula/bconsole.conf 
Connecting to Director localhost:9101
1000 OK: 10002 bacula-dir Version: 15.0.2 (21 March 2024)
Enter a period to cancel a command.
*run
Automatically selected Catalog: PostgreSQL
Using Catalog "PostgreSQL"
A job name must be specified.
Automatically selected Job: bacula backup DefaultSet
Run Backup job
JobName:  bacula backup DefaultSet
Level:    Incremental
Client:   bacula-fd
FileSet:  DefaultSet
Pool:     FilePool (From Job resource)
Storage:  bacula-sd_file1 (From Pool resource)
When:     2025-05-18 10:15:45
Priority: 30
OK to run? (Yes/mod/no): y
Job queued. JobId=2
*
```
Looks like it's queued up and ready to go.  Let's check the status of the client:
```shell
*status client
Automatically selected Client: bacula-fd
Connecting to Client bacula-fd at bacula.lab:9102

bacula-fd Version: 15.0.2 (21 March 2024)  x86_64-pc-linux-gnu unknown_distro 
Daemon started 17-May-25 10:14. Jobs: run=0 running=1 max=10.
 Heap: heap=557,056 smbytes=537,589 max_bytes=537,894 bufs=203 max_bufs=205
 Sizes: boffset_t=8 size_t=8 debug=0 trace=0 mode=0,0 bwlimit=0kB/s
 Crypto: fips=N/A crypto=OpenSSL 3.3.2 3 Sep 2024
 Plugin: bpipe(2) 

Running Jobs:
JobId 2 Job bacula_backup_DefaultSet.2025-05-18_10.15.49_03 is running.
    Full Backup Job started: 17-May-25 12:15
    Files=460 Bytes=6,759,958 AveBytes/sec=281,664 LastBytes/sec=268,396 Errors=0
    Bwlimit=0 ReadBytes=6,759,958
    Files: Examined=460 Backed up=460
    Processing file: /boot/grub/i386-pc/lspci.mod
    SDReadSeqNo=8 fd=7 SDtls=1
Director connected using TLS at: 17-May-25 12:16
====

Terminated Jobs:
====
*
```
Yep, it's currently backing up the `/boot` directory.  Let's check the status of the volume it's backing up to:
```shell
*list volume
Using Catalog "PostgreSQL"
Pool: FilePool
+---------+------------+-----------+---------+----------+----------+--------------+---------+------+-----------+-----------+---------+----------+-------------+-----------+
| mediaid | volumename | volstatus | enabled | volbytes | volfiles | volretention | recycle | slot | inchanger | mediatype | voltype | volparts | lastwritten | expiresin |
+---------+------------+-----------+---------+----------+----------+--------------+---------+------+-----------+-----------+---------+----------+-------------+-----------+
|       1 | Vol-0001   | Append    |       1 |      237 |        0 |    1,296,000 |       1 |    0 |         0 | File      |       1 |        0 |             |         0 |
+---------+------------+-----------+---------+----------+----------+--------------+---------+------+-----------+-----------+---------+----------+-------------+-----------+
*
```
We can see that the volume is currently in the Append state.  This means that it can be written to, but nothing has been written to it yet.
Let's check the status of the volume again:
```shell
*list volume
Pool: FilePool
+---------+------------+-----------+---------+------------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+-----------+
| mediaid | volumename | volstatus | enabled | volbytes   | volfiles | volretention | recycle | slot | inchanger | mediatype | voltype | volparts | lastwritten         | expiresin |
+---------+------------+-----------+---------+------------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+-----------+
|       1 | Vol-0001   | Append    |       1 | 65,975,866 |        0 |    1,296,000 |       1 |    0 |         0 | File      |       1 |        0 | 2025-05-17 12:18:00 | 1,295,953 |
+---------+------------+-----------+---------+------------+----------+--------------+---------+------+-----------+-----------+---------+----------+---------------------+-----------+
*
```
Aah!  Now it's starting to fill up.  We can see that it's written about 65MB to the volume.
Now we wait.

After a few minutes, the backup should finish.  Let's check the status of the client again:
```shell
*list jobs
+-------+--------------------------+---------------------+------+-------+----------+------------+-----------+
| jobid | name                     | starttime           | type | level | jobfiles | jobbytes   | jobstatus |
+-------+--------------------------+---------------------+------+-------+----------+------------+-----------+
|     1 | bacula backup DefaultSet | 2025-05-17 12:00:02 | B    | F     |        0 |          0 | f         |
|     2 | bacula backup DefaultSet | 2025-05-17 12:15:52 | B    | F     |    2,248 | 65,633,599 | T         |
+-------+--------------------------+---------------------+------+-------+----------+------------+-----------+
*
```
We can ignore that first job.  It was a test job that I ran before I started writing this post, but we see our job with ID 2 as `T` (terminated successfully) and the job backed up 2,248 files and 65MB of data.

## Conclusion
In this post, we configured a Bacula client and its File Daemon.  We also ran our first backup.
