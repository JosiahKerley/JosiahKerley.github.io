---
layout: post
title:  "Configuring the Bacula Catalog and Director on NixOS" 
date:   2025-05-03 00:00:00
toc: true
series: "Immutable Enterprise Backup"
categories:
  - bacula-on-nixos
  - bacula
  - guide
---

!["The logos of the bacula and nixos project"](/assets/img/posts/2025-04-17-immutable-bacula/splash.png)

## Laying out the Configuration
Now that we have a high-level overview of the Bacula architecture, we can start configuring the Bacula catalog and director on NixOS.
First, let's break out our configuration.nix so that all the bacula related code is in its own directory.

### Create the directory structure
Current structure:
```
/etc/nixos
├── configuration.nix
└── hardware-configuration.nix
```
Create a new directory for the Bacula configuration files:
```shell
[josiah@bacula:~]$ sudo mkdir -p /etc/nixos/bacula/dir
```
New structure:
```
/etc/nixos
├── bacula
│   └── dir
├── configuration.nix
└── hardware-configuration.nix
```

### Director's Configuration File
In /etc/nixos/bacula/dir create a new file called `main.nix` with a minimal config.
```shell
[josiah@bacula:~]$ sudo nano /etc/nixos/bacula/dir/main.nix
```
```nix
{ config, pkgs, lib, ... }:
{
  imports = [
    ./services.nix
  ];
}
```
This will serve as the entrypoint for the Bacula catalog and director configuration.
Don't worry, we will be adding more to this file later.

As the import implies, we will create a new file called `services.nix` in the same directory.
```shell
[josiah@bacula:~]$ sudo nano /etc/nixos/bacula/dir/services.nix
```

To start, we will configure the server to run postgres.
```nix
{ config, pkgs, lib, ... }:
{
  services = {
    postgresql = {
      enable = true;
      package = pkgs.postgresql_13;
      settings.timezone = config.time.timeZone;
      enableTCPIP = true;
      initialScript = pkgs.writeText "postgresql-init.sql" ''
        CREATE ROLE root LOGIN CREATEDB CREATEROLE;
      '';
    };
  };
}
```

### Plumbing it all together
In `/etc/nixos/configuration.nix` there is an import statement near the top of the file.
```nix
...
  imports =
    [ # Include the results of the hardware scan.
      ./hardware-configuration.nix
    ];
...
```

We will add our new Bacula configuration to this list.
```nix
...
  imports =
    [ # Include the results of the hardware scan.
      ./hardware-configuration.nix
      ./bacula/dir/main.nix
    ];
...
```

### Rebuild the system
Now that we have our configuration in place, we can rebuild the system.
```shell

[josiah@bacula:~]$ sudo nixos-rebuild switch
building Nix...
building the system configuration...
these 25 derivations will be built:
...
building '/nix/store/sn258ay8c7fq3g3xmxva6nj2vn6q0nzz-nixos-system-bacula-24.11.710905.a0f3e10d9435.drv'...
updating GRUB 2 menu...
Warning: os-prober will be executed to detect other bootable partitions.
Its output will be used to detect bootable binaries on them and create new boot entries.
lsblk: /dev/mapper/no*[0-9]: not a block device
lsblk: /dev/mapper/block*[0-9]: not a block device
lsblk: /dev/mapper/devices*[0-9]: not a block device
lsblk: /dev/mapper/found*[0-9]: not a block device
stopping the following units: accounts-daemon.service
activating the configuration...
setting up /etc...
reloading user units for josiah...
reloading user units for gdm...
restarting sysinit-reactivation.target
reloading the following units: dbus.service
restarting the following units: polkit.service
starting the following units: accounts-daemon.service
the following new units were started: postgresql.service, run-credentials-systemd\x2dtmpfiles\x2dresetup.service.mount, sysinit-reactivation.target, systemd-tmpfiles-resetup.service
```

Looks good, let's see if we now have a running postgres instance.
```shell
[josiah@bacula:~]$ sudo systemctl status postgresql
● postgresql.service - PostgreSQL Server
     Loaded: loaded (/etc/systemd/system/postgresql.service; enabled; preset: ignored)
     Active: active (running) since Sat 2025-05-03 10:10:12 MDT; 1min 19s ago
...
```
Great! We have a running postgres instance.  With all of that boilerplate out of the way, we can now start configuring the Bacula catalog and director.

## Configuring the Catalog and Director

### The Catalog
Let's edit services.nix and add an initial director configuration.
We will need this to install the database setup scripts.
```nix
{ config, pkgs, lib, ... }:
{
  services = {
    postgresql = {
    ...
    };
    bacula-dir = {
      enable = true;
      name = "bacula-dir";
      password = "ChangeMe";
      port = 9101;
      tls.enable = false;
      extraConfig = ''
        Messages {
          Name = Daemon
          mailcommand = "${pkgs.bacula}/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula daemon message\" %r"
          operatorcommand = "/home/bacula/bin/bsmtp -h localhost
                -f \"\(Bacula\) %r\"
                -s \"Bacula: Intervention needed for %j\" %r"
          mail = sysop@lab = all, !skipped, !info
          #mail = sysop@lab = all
          console = all, !skipped, !saved
          operator = sysop@lab = all
          append = "/var/lib/bacula/bacula-messages-daemon.log" = all, !skipped
          catalog = all
        }
      '';
    };
  };
}
```

Run `nicxos-rebuild switch`.  The command will fail in the end, but it should put our database scripts in the nix store.
The three scripts we need are:
* create_bacula_database
* make_bacula_tables
* grant_bacula_privileges
To find the paths of these scripts, run the following command:
```bash
find /nix/store/ -name create_bacula_database -or -name make_bacula_tables -or -name grant_bacula_privileges
```
For me, the output was:
```shell
/nix/store/82cw16m67asbxi04rxkwxm6djnd12q98-bacula-15.0.2/etc/make_bacula_tables
/nix/store/82cw16m67asbxi04rxkwxm6djnd12q98-bacula-15.0.2/etc/create_bacula_database
/nix/store/82cw16m67asbxi04rxkwxm6djnd12q98-bacula-15.0.2/etc/grant_bacula_privileges
```

So for me, I need to run:
```shell
sudo /nix/store/82cw16m67asbxi04rxkwxm6djnd12q98-bacula-15.0.2/etc/create_bacula_database postgresql
sudo /nix/store/82cw16m67asbxi04rxkwxm6djnd12q98-bacula-15.0.2/etc/make_bacula_tables postgresql
sudo /nix/store/82cw16m67asbxi04rxkwxm6djnd12q98-bacula-15.0.2/etc/grant_bacula_privileges postgresql
sudo touch /var/lib/bacula/db-created
```
in order to setup the catalog in the postgres database.

In the future this process may not be necessary, as of the date of writing this post, the bacula-dir service in nixos does not create the database for you.

## A minimal Director configuration
With the current `service.nix`, the bacula-dir service will not start.
This is because we currently have no Job defined.  So let's put in a minimum working nix configuration that will allow the bacula-dir service to start.
```nix
{ config, pkgs, lib, ... }:
{
  services = {
    postgresql = {
      enable = true;
      package = pkgs.postgresql_13;
      settings.timezone = config.time.timeZone;
      enableTCPIP = true;
      initialScript = pkgs.writeText "postgresql-init.sql" ''
        CREATE ROLE root LOGIN CREATEDB CREATEROLE;
      '';
    };
    bacula-dir = {
      enable = true;
      name = "bacula-dir";
      password = "ChangeMe!";
      port = 9101;
      tls.enable = false;
      extraConfig = ''
        Messages {
          Name = Daemon
          mailcommand = "${pkgs.bacula}/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula daemon message\" %r"
          operatorcommand = "/home/bacula/bin/bsmtp -h localhost
                -f \"\(Bacula\) %r\"
                -s \"Bacula: Intervention needed for %j\" %r"
          mail = sysop@lab = all, !skipped, !info
          console = all, !skipped, !saved
          operator = sysop@lab = all
          append = "/var/lib/bacula/bacula-messages-daemon.log" = all, !skipped
          catalog = all
        }

        Storage {
          Name = bacula-sd_file1
          Address = bacula.lab
          Password = "ChangeMe!"
          Device = File1
          Autochanger = yes
          Media Type = File
          Maximum Concurrent Jobs = 20
        }

        Pool {
          Name = FilePool
          Pool Type = Backup
          Label Format = "Vol-"
          Recycle = yes
          Auto Prune = yes
          Volume Retention = 15 days
          Storage = bacula-sd_file1
          Maximum Volume Bytes = 50G
          Maximum Volumes = 100
        }

        FileSet {
          Name = "DefaultSet"
          Include {
            File = /
            Options {
              Signature = MD5
              OneFS = no
            }
            Options {
              Exclude = yes
              WildDir = "*/.cache"
              WildDir = "*/Downloads"
            }
          }
          Exclude {
            File = /swapfile
            File = /var/lib/bacula
            File = /nix
            File = /tmp
            File = /dev
            File = /proc
            File = /sys
            File = /run
            File = /mnt
            File = /media
            File = /lost+found
            File = /var/log/journal
            File = /var/cache
            File = /var/lib/nginx/tmp
          }
        }

        Schedule {
          Name = "StandardCycle"
          Run = Incremental mon-sun at 10:00
          Run = Full monthly at 17:00
        }

        JobDefs {
          Name = "BaseJob"
          Type = Backup
          Messages = Standard
          Priority = 30
          Pool = FilePool
          FileSet = "DefaultSet"
          Schedule = "StandardCycle"
          Write Bootstrap = "/var/lib/bacula/bootstrap-records/%c.bsr"
        }

        Job {
          Name = "bacula backup DefaultSet"
          JobDefs = "BaseJob"
          Client = bacula-fd = all
          FileSet = "DefaultSet"
        }

        Client {
          Name = bacula-fd
          Catalog = PostgreSQL
          Address = bacula.lab
          Password = "ChangeMe!"
        }
      '';
    };
  };

  systemd.services.bacula-dir = {
    serviceConfig = {
      ExecStartPre = [ "-${pkgs.coreutils}/bin/chown -R bacula:bacula /var/lib/bacula" ];
      ExecStartPost = [ "${pkgs.coreutils}/bin/chown -R bacula:bacula /var/lib/bacula" ];
    };
    path = [ pkgs.bacula pkgs.coreutils pkgs.su pkgs.postgresql_13 ];
  };

  environment.etc."bacula/bconsole.conf".text = ''
    Director {
      Name = bacula-dir
      address = localhost
      Password = "ChangeMe!"
    }
  '';

}
```
And after a `nixos-rebuild switch`, we should have a working Bacula director.
```shell
[josiah@bacula:~]$ sudo systemctl status bacula-dir
● bacula-dir.service - Bacula Director Daemon
     Loaded: loaded (/etc/systemd/system/bacula-dir.service; enabled; preset: ignored)
     Active: active (running) since Sun 2025-05-03 10:38:39 MDT; 4min 34s ago
 Invocation: 81be24a6c70e41a2817dc2679c4293c7
   Main PID: 46063 (bacula-dir)
         IO: 0B read, 0B written
      Tasks: 5 (limit: 4684)
     Memory: 2M (peak: 3.2M)
        CPU: 88ms
     CGroup: /system.slice/system-bacula.slice/bacula-dir.service
             └─46063 /nix/store/82cw16m67asbxi04rxkwxm6djnd12q98-bacula-15.0.2/sbin/bacula-dir -f -u bacula -g bacula -c /nix/store/py5c38l2zpy2jrwrfyaxh1iprbbb0kvd-bacula-dir.conf

May 03 01:38:39 bacula su[46055]: Successful su for postgres by root
May 03 01:38:39 bacula su[46055]: pam_unix(su:session): session opened for user postgres(uid=71) by (uid=0)
May 03 01:38:39 bacula bacula-dir-pre-start[46057]:  
May 03 01:38:39 bacula bacula-dir-pre-start[46057]: This script will update a Bacula PostgreSQL database
May 03 01:38:39 bacula bacula-dir-pre-start[46057]:  from any from version 12-16 or 1014-1025 to version 1026
May 03 01:38:39 bacula bacula-dir-pre-start[46057]:  which is needed to convert from any Bacula Communty to version 15.0.x
May 03 01:38:39 bacula bacula-dir-pre-start[46057]:  
May 03 01:38:39 bacula bacula-dir-pre-start[46057]: Current bacula database is up-to-date (version 1026).
May 03 01:38:39 bacula su[46055]: pam_unix(su:session): session closed for user postgres
May 03 01:38:39 bacula systemd[1]: Started Bacula Director Daemon.
```

## Poking around with bconsole
Now that we have a running Bacula director, we can use the `bconsole` command to interact with it.
`bconsole` is the defacto command line interface for Bacula.  It is also used for crude shell scripting.
```shell
[josiah@bacula:~]$ bconsole -c /etc/bacula/bconsole.conf 
Connecting to Director localhost:9101
1000 OK: 10002 bacula-dir Version: 15.0.2 (21 March 2024)
Enter a period to cancel a command.
*status director
bacula-dir Version: 15.0.2 (21 March 2024) x86_64-pc-linux-gnu unknown_distro 
Daemon started 03-May-25 02:04, conf reloaded 03-May-2025 02:04:13
 Jobs: run=0, running=0 max=20 mode=0,0
 Crypto: fips=N/A crypto=OpenSSL 3.3.2 3 Sep 2024
 Heap: heap=671,744 smbytes=338,083 max_bytes=1,339,589 bufs=324 max_bufs=336
 Res: njobs=1 nclients=1 nstores=1 npools=1 ncats=1 nfsets=1 nscheds=1

Scheduled Jobs (2/50):
Level          Type     Pri  Scheduled          Job Name           Volume
===================================================================================
Incremental    Backup    30  03-May-25 10:00    bacula backup DefaultSet *unknown*
Full           Backup    30  03-May-25 17:00    bacula backup DefaultSet *unknown*
====

Running Jobs:
Console connected using TLS at 03-May-25 02:06
No Jobs running.
====
No Terminated Jobs.
====
*
```

From that we can see that we have a running Bacula director, and that it is configured to run two jobs.
Since we haven't configured and file daemon or storage daemon, we can't do very much however.

## Conclusion
In this post, we have configured the Bacula catalog and director on NixOS using a minimal configuration.
It's not production ready, we don't have any clients, we don't even have a storage daemon yet, but the director is running and we can start adding more configuration.
In the next post, we will configure the Bacula storage daemon.
