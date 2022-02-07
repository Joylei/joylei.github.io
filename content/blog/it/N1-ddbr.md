+++
title = "N1 ddbr 备份与恢复"
date=2020-05-08

[taxonomies]
categories=["Firmware"]
tags=["N1", "ddbr", "备份", "恢复", "刷机"]
+++

使用`ddbr`来备份内存，以便随时快速恢复。

## 备份

```
root@aml:~# ddbr
 DO YOU WANT TO BACKUP OR RESTORE ?
 BACKUP=(b) RESTORE=(r) b
 AVAILABLE DEVICES: mmcblk1 sda1 sda2
 YOU ARE RUNNING bionic FROM sda2
 INTERNAL EMMC IS: mmcblk1 SIZE:        7634944
 ROOT (sda2) FREE SPACE IS:             118886072
 DO YOU WANT COMPRESSION ?
 YES=(y) NO=(n) y
 SAVING AND COMPRESSING mmcblk1 TO /ddbr/BACKUP-s9xxx-emmc.img.gz...
15269888+0 records inMiB/s] [==============================================================================>] 100% ETA 0:00:00
15269888+0 records out
7.28GiB 0:10:53 [11.4MiB/s] [==============================================================================>] 100%
7818182656 bytes (7.8 GB, 7.3 GiB) copied, 653.739 s, 12.0 MB/s
 JOB FINISHED!
root@aml:~# ls /ddbr
BACKUP-s9xxx-emmc.img.gz  install

```

## 恢复

```
root@aml:~# ls /ddbr
BACKUP-s9xxx-emmc.img  BACKUP-s9xxx-emmc.img.gz.bak  install
root@aml:~# ddbr
 DO YOU WANT TO BACKUP OR RESTORE ?
 BACKUP=(b) RESTORE=(r) r
 AVAILABLE DEVICES: mmcblk1 sda1 sda2
 YOU ARE RUNNING bionic FROM sda2
 DID YOU USED COMPRESSION WHEN YOU TOOK THE BACKUP ?
 YES=(y) NO=(n) n
 YOU ARE ABOUT TO MAKE SERIOUS CHANGES TO YOUR SYSTEM!!!
 FILE /ddbr/BACKUP-s9xxx-emmc.img IS GOING TO BE WRITEN TO mmcblk1
 MAKE SURE EVERYTHING LOOKS OK AND:
 PRESS ENTER TO CONTINUE OR CTRL+C TO CANCEL
 RESTORING /ddbr/BACKUP-s9xxx-emmc.img TO /dev/mmcblk1 | PLEASE WAIT...
6283264+0 records in2MiB/s] [===============================>                                                ] 41% ETA 0:05:50
6283264+0 records out
3217031168 bytes (3.2 GB, 3.0 GiB) copied, 244.506 s, 13.2 MB/s
3.00GiB 0:04:04 [12.5MiB/s] [===============================>                                                ] 41%
6283264+0 records in
6283264+0 records out
3217031168 bytes (3.2 GB, 3.0 GiB) copied, 248.16 s, 13.0 MB/s
 JOB FINISHED!

```
