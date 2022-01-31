---
title: "Expanding An EBS Root Volume"
author_profile: true
header:
  overlay_image: /assets/img/home-header.jpg
---
# Expanding An EBS Root Volume
I'm prepping to install Kubernetes on an existing EC2 instance I use for FoundryVTT and a ProjectZomboid multiplayer server currently running in a Docker container. First thing I wanted to do was increase the size of the File System. Nothing worse than running out of space when you don't feel like doing anything. To clarify when I refer to the volume I mean the actual size of the EBS storage. The partition is the portion that must be manually allocated for use and then formatted with a file system. So in this case we need to increase the size of the volume, extend the partition, and extend the file system in that order.
## Create a Snapshot
List volumes:
```bash
aws ec2 describe-volumes
```
Create snapshot:
```bash
aws ec2 create-snapshot --description "Expanding EBS Volume" --volume-id <value>
```
## Expand the Volume
Expand volume specifying size in GBs:
```bash
aws ec2 modify-volume --volume-id <value> --size <value>
```
Monitor progress, can proceed after entering optimizing state:
```bash
aws ec2 describe-volumes-modifications --volume-ids <value>
```
## Extend Partion
Identify the filesystem, Amazon Linux/RHEL under 'type' column:
```bash
df -Ht
```
Locate the name of the partion to be extended with `lsblk` command:
```bash
[ec2-user ~]$ lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
nvme1n1       259:0    0  30G  0 disk /data
nvme0n1       259:1    0  16G  0 disk
└─nvme0n1p1   259:2    0   8G  0 part /
└─nvme0n1p128 259:3    0   1M  0 part
```
Note that the "partition number" is appended to the device (NAME column in example). Don't use the high number.
Extend the partition:
```bash
sudo growpart <partition> <partition number>
# example
sudo growpart /dev/xvda 1
```
Verify new partition size with `lsblk`
## Extend File System
XFS:
```bash
sudo xfs_growfs -d <mount point e.g. "/>
```
ext4:
```bash
sudo resize2fs /dev/<partition>
```
[Reference](https://aws.amazon.com/premiumsupport/knowledge-center/expand-root-ebs-linux/)
