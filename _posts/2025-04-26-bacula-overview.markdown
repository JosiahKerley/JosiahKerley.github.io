---
layout: post
title:  "Architecture of Bacula"
date:   2025-04-26 00:00:00
series: "Immutable Enterprise Backup"
categories:
  - bacula-on-nixos
  - bacula
---

!["The logos of the bacula and nixos project"](/assets/img/posts/2025-04-17-immutable-bacula/splash.png)

## Abstract
Bacula is a free and open-source backup software that provides a robust and scalable solution for data protection in various environments.
In this blog post, we will delve into the architectural overview of Bacula, exploring its key components and their interactions. 

## An Architectural Overview of Bacula

### Bacula's Roles
For the purposes of this post I will be using the term role to mean the service that the component provides to implement the architecture of Bacula.
These roles are realized as daemons that run on the server or client machines.

#### The Catalog
The catalog is the central database that stores information about the Bacula configuration, including backup jobs, schedules, file sets, clients, and job definitions.
The catalog is designed to be highly scalable and efficient, allowing it to handle large amounts of data and numerous backups.
Generally, the catalog is implemented using a relational database management system (RDBMS) such as MySQL, PostgreSQL, or SQLite.

#### The Director
The director is the primary component responsible for managing the Bacula backup process.
It acts as a coordinator between the client, storage daemons, and file daemons.
The director receives job requests from clients, schedules them, and then directs the storage daemons to retrieve and store the backed-up data.
The canonical name for the service is `bacula-dir`, but it is often referred to as the director.

#### Storage Daemon(s)
Storage daemons are responsible for receiving files from the Bacula director and storing them on designated media, such as disk or tape.
Canonically, the storage daemon is referred to as `bacula-sd`, but it is often referred to as the storage daemon.

#### File Daemons
File daemons are the client-side service that runs on the machines being backed up.
Essentially, they are the agents that communicate with the Bacula director to send backup data to the storage daemons.
This is sometimes referred to as `bacula-client` in certain package managers, but the canonical name is `bacula-fd`.
 

### Bacula Resources
In Bacula, resources are the various components that make up the backup system.
The primary resources that we will be covering in this post are Pools, File Sets, Schedules, Jobs, and Job Definitions.

#### Pools
Pools are logical containers that group multiple storage daemons together.
Pools allow administrators to manage and monitor backup data across multiple storage devices or media types.
For example, a pool can be created for disk storage, while another pool can be created for tape storage.
This way you can store full backups on tapes and incremental backups on disks.
That way you can save time during a restore operation, as the full backup is already on tape and only the incremental backups need to be restored from disk.

Example configuration of a pool:
```
Pool {
  Name = FilePool
  Pool Type = Backup
  Label Format = "Vol-"
  Recycle = yes
  Auto Prune = yes
  Volume Retention = 15 days
  Storage = some-storage
  Maximum Volume Bytes = 50G
  Maximum Volumes = 100
}
```


#### File Sets
File sets define which files or directories on a client should be included in a backup job.
A file set can contain multiple files and subdirectories, allowing for granular control over the data being backed up. 

Example configuration of a file set:
```
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
```


#### Schedules
Schedules define when backups should occur and what backup level (full, differential, incremental, etc) should be used.

Example configuration of a schedule:
```
Schedule {
  Name = "StandardCycle"
  Run = Incremental mon-sun at 10:00
  Run = Full monthly at 17:00
}
```


#### Jobs
Jobs represent a single backup operation initiated by a client. A job includes information such as the file set, schedule, and storage location. 

Example configuration of a job:
```
Job {
  Name = "backuptest backup DefaultSet"
  JobDefs = "BaseJob"
  Client = backuptest-fd = all
  FileSet = "DefaultSet"
}
```


#### Job Definitions
Job definitions are a set of defaults that can be reused for multiple jobs defined.
For example, you may want to hide the complexity of the job definition from the user and only expose the job name, client, and fileset.

Example configuration of a job definition:
```
JobDefs {
  Name = "DefaultJob"
  Type = Backup
  Messages = Standard
  Priority = 30
  Pool = FilePool
  FileSet = "DefaultSet"
  Schedule = "StandardCycle"
  Write Bootstrap = "/var/lib/bacula/bootstrap-records/%c.bsr"
}
```

## Conclusion
In this blog post, we have provided an architectural overview of Bacula, highlighting its key components and their roles in the backup process.
We have also discussed the various resources that Bacula uses to manage backup jobs, including pools, file sets, schedules, jobs, and job definitions.
Understanding these components is essential for effectively configuring and managing Bacula in a production environment.
In the next post, we will begin to configure the catalog and director.
